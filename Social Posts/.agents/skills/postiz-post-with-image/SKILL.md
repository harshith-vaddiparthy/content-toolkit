---
name: postiz-post-with-image
description: Draft, illustrate, and publish a social post through the Postiz CLI. Use whenever the user asks to post, schedule, or share something via Postiz (also called "Posters"), especially a LinkedIn post about an article or announcement. Writes the text with correct paragraph spacing, finds or captures an image from the source, uploads it, schedules the post, and confirms it is live.
---

# Post with an image through Postiz

This skill sits on top of the installed `postiz` skill (`../postiz/SKILL.md`) and its CLI. Read that skill's authentication and file-upload rules first. This one adds the parts it lacks: spacing that renders properly, an image for every post, and a verification step.

## Hard rules

1. **Every post gets an image.** If the user did not supply one, find one using the order in "Find the image". Never post text-only without telling the user why.
2. **Spacing uses empty paragraphs.** Postiz joins plain `<p>` blocks with a single newline on LinkedIn, which produces a wall of text. See "Format the text".
3. **Do not shorten links.** Pass `--shortLink false`. The default rewrites the source URL to a `dub.sh` link, and the reader no longer sees where it points.
4. **Publishing is public.** If the user said to post or publish, go ahead. If they only asked for a draft, use `-t draft`. When the channel is ambiguous (two LinkedIn channels, two X accounts), ask which one; never guess.
5. **Never say "posted" until `posts:list` shows `PUBLISHED`.** `QUEUE` means waiting, and a few seconds to a minute of lag after the scheduled time is normal.
6. **Facts only from the source.** Do not invent experiences, numbers, or opinions for the user. Plans stay plans ("expects to", "not available yet"). For editing rules see the `humanizer-zh-en` skill (its principles apply to English text too).

## Workflow

1. Run `postiz auth:status`, then `postiz integrations:list` and choose the channel.
2. Read the source (Firecrawl scrape or the browser). Write the post.
3. Find and capture the image.
4. `postiz upload <file>` and keep the returned `path` URL.
5. `postiz posts:create` with the image, `--shortLink false`, and a date about two minutes ahead.
6. Wait for `PUBLISHED`, open the live post, check spacing and image, and report the link.

## Format the text

Postiz content is HTML. For LinkedIn, X, and most channels:

- One `<p>` per paragraph, with an empty `<p></p>` between paragraphs. This renders as a blank line (confirmed on the published post of 2026-10-23).
- Lines that belong together, such as a numbered list or three short questions, are consecutive `<p>` elements with no empty one between them. They stay tight.
- No Markdown, no `**bold**`, no emoji unless the user asks.
- The source link gets its own paragraph as a plain URL.
- Hashtags go in the last paragraph, three at most.
- Keep it under the channel's limit (`postiz integrations:settings <id>` shows `maxLength`; LinkedIn is 3000).

Template:

```html
<p>Opening line that states the news.</p><p></p><p>Second paragraph.</p><p></p><p>1. First step.</p><p>2. Second step.</p><p></p><p>Source: https://example.com/article</p><p></p><p>#TagOne #TagTwo</p>
```

Build the string in a shell heredoc (`cat <<'EOF'`) so apostrophes survive.

## Find the image

Try these in order and stop at the first that makes the post's main point visible:

1. **The article's own media.** Check for a hero video or animation: in the browser, run `[...document.querySelectorAll('iframe')].map(f => f.src)` and `document.querySelectorAll('video')`. Capture the frame that shows the feature the post describes (for a payments post, the approval screen), not a title card.
2. **A static image from the article** (diagram, product screenshot).
3. **The page's `og:image`** (from the Firecrawl metadata). Acceptable, but say it is a generic social card.
4. **A screenshot of the relevant section of the page.**
5. If nothing exists, tell the user and offer to generate one. Do not use a stock photo.

State the image's source in your final message. It is usually the source company's artwork, and the user decides whether to use it. Demo screens can contain fake names or addresses; mention that if present.

### Capturing a frame from an embedded video

This worked for a Vimeo hero video (2560x1440). The built-in browser pane has limits that shaped every step:

- Screenshots are capped at 800x600. The `zoom` action does not add resolution.
- A page in the pane cannot reach `localhost`, so you cannot post a canvas export to a local server. Do not try; it wastes time.
- Setting `video.currentTime` can clamp to an earlier time. Play the video instead and take screenshots every few seconds until the right frame shows, then pause.

Steps:

1. Navigate the pane to the player URL from the iframe (add `autoplay=1&muted=1&controls=0`).
2. `resize_window` to `1280x720`, play, and sample screenshots (`computer screenshot` with `scale: 0.5` in a `browser_batch`, `wait` 3 seconds between) until the frame you want appears.
3. Pause: `const v = document.querySelector('video'); v.pause()`.
4. Zoom the video with CSS so the interesting region fills the 800x600 viewport, using `resize_window` 800x600 first. Compute `k = target_width_px / region_width_in_video_px`, then:

```js
const v = document.querySelector('video');
v.style.cssText = 'position:fixed;left:Lpx;top:Tpx;width:Wpx;height:Hpx;max-width:none;max-height:none;z-index:9999;background:#fff';
```

   where `W = 2560 * k`, `H = 1440 * k`, `L` puts the region's left edge where you want it, and `T = 600 - H` aligns the bottom. For the Wallet video, `L=-514, T=-429, W=1829, H=1029` framed the whole approval sheet.
5. Take a screenshot (`computer screenshot`, scale 1). The tool prints the saved file path.
6. Crop to a square or 4:5 with macOS `sips`, for example `sips -s format png -c 600 600 --cropOffset 0 100 <screenshot.jpg> --out <name>.png`. Save into `Social Posts/images/`.
7. Open the result with Read and check it: nothing cut off, text legible.
8. `resize_window` with `preset: "desktop"` to clear the emulation.

Resolution is limited to about 600-800 px. That is fine for a feed image but soft if enlarged. Tell the user the pixel size.

## Upload and create

```bash
postiz upload "Social Posts/images/<file>.png"        # note the "path" URL in the output
CONTENT=$(cat <<'EOF'
<p>...</p><p></p><p>...</p>
EOF
)
WHEN=$(date -u -v+2M +%Y-%m-%dT%H:%M:%SZ)
postiz posts:create -c "$CONTENT" -m "<path URL>" -s "$WHEN" -t schedule \
  -i "<integration id>" --shortLink false
```

Notes:

- LinkedIn accepts one image per ordinary post. Two or more images need the carousel setting (see `postiz integrations:settings <id>`).
- `-s` is required, even for a draft.
- The result gives a `postId`. Keep it for verification.

## Verify

`posts:list` returns many old posts, so filter by id. Do not chain `sleep` commands; run a loop in the background and wait for the notification:

```bash
for i in $(seq 1 60); do
  S=$(postiz posts:list 2>&1 | tail -n +2 | jq -r '.posts[] | select(.id=="<postId>") | .state')
  [ "$S" != "QUEUE" ] && break; sleep 10
done
postiz posts:list 2>&1 | tail -n +2 | jq '.posts[] | select(.id=="<postId>") | {state,publishDate,releaseURL,error}'
```

When `state` is `PUBLISHED`, scrape the `releaseURL` (Firecrawl, `maxAge: 0`) and confirm that paragraphs are separated by blank lines and the source link is the full URL. If `state` is `ERROR`, report the `error` field and stop.

## Final message

Give the live link, the channel, where the image came from and its pixel size, and anything you could not check. Say plainly if the post is still queued.

## Posting through Ego browser instead

Posting by driving LinkedIn's own composer in the Ego browser is a separate method with its own skill: `../ego-browser-linkedin-post/SKILL.md`. Use that one when the user says "using Ego browser". The Postiz-only rules above (`<p></p>` spacing, `--shortLink false`) do not apply there.

## Fixing a post that is already live

LinkedIn posts cannot be edited through Postiz. Fixing spacing or a link means deleting the post and publishing a corrected one. That removes any reactions and comments, so ask first and say what will be lost.
