---
name: ego-browser-linkedin-post
description: Write and publish a LinkedIn post, with an image, by driving LinkedIn's own composer in the Ego browser. Use whenever the user asks to post to LinkedIn "using Ego browser" or "in the browser", or when Postiz is not wanted. Finds the source article, drafts the text with correct spacing, captures an image from the article at full size, attaches it, posts, and verifies the live post.
---

# Post to LinkedIn with Ego browser

This skill drives LinkedIn directly. It does not use Postiz. For scheduling through Postiz see `../postiz-post-with-image/SKILL.md`. The user is already signed in to LinkedIn in Ego, so there is nothing to log into.

Before any browser step, run `ego-browser skill` and follow its guide (task spaces, `finish()`, user-control rules). That guide belongs to the installed `ego-browser` skill. Do not edit it.

## Hard rules

1. **Every post gets an image.** Capture it from the source article. Never post text-only without telling the user why.
2. **Facts only from the source.** Prefer the company's own announcement over press coverage. Do not invent opinions, experiences, or numbers for the user. Plans stay plans.
3. **The user's request decides whether to publish.** If they said to post, post. If they only asked for a draft, stop with the composer filled, show a screenshot, and wait.
4. **Default author is the personal profile.** The composer's dropdown shows the signed-in person's name; any company pages they manage need that dropdown changed. Switching to a page is untested; ask the user before posting as a page.
5. **One task space for the whole job.** Close it with `finish({ keep: [] })` in a separate last script, after the post is verified.
6. **Say what you could not check.** If the exact post link is unavailable, say so and give the activity-page link.

## Workflow

1. **Find the source.** Search for the official page first (for example `claude.com/resources/articles/...`), then scrape it with Firecrawl (`formats: ["markdown", "images"]`, `maxAge: 0`). Note the image URLs and any video embed.
2. **Draft the text** (see "Write the text").
3. **Get the image** (see "Capture the image").
4. **Compose, attach, type, check, post** (see "The composer").
5. **Verify** on the activity page.
6. **Report**: channel, image source and pixel size, link or activity-page link, and anything unchecked.

## Write the text

- Opening line states the news. Short paragraphs, one idea each.
- Bullets use the `•` character, one per line.
- Source link on its own line, then hashtags (three at most) on the last line.
- No emoji unless the user asks. Sentence case. No hype words.
- Keep to what the page says; attribute it ("Anthropic spells out…", "OpenAI says…").
- Keep it under about 250 words.

## Capture the image

Open each candidate image URL from the article in the Ego page, screenshot it, and look at it with Read. Choose the one that shows the feature in use, not a title card.

Then capture the chosen one at full size with CDP. `page.screenshot()` returns only the viewport (about 900 px wide, with dark letterbox), which is too small:

```js
await page.goto("<image url>", { timeout: 45000 });
await page.waitForLoadState();
const r = await page.evaluate(() => { const b = document.querySelector("img").getBoundingClientRect(); return { x: b.x, y: b.y, w: b.width, h: b.height }; });
const shot = await page.cdp("Page.captureScreenshot", { format: "png", clip: { x: r.x, y: r.y, width: r.w, height: r.h, scale: 2 } });
const fs = await import("node:fs/promises");
await fs.writeFile("<absolute path to this repo>/Social Posts/images/<name>.png", Buffer.from(shot.data, "base64"));
```

Scripts need absolute paths. The result can be very large (6000 px, about 3 MB); shrink it with `sips -Z 2000 <file>` and check the size with `sips -g pixelWidth -g pixelHeight`. Then open it with Read to confirm nothing is cut off.

If the article has no plain image files (the picture is a video), use the Vimeo frame-capture method in the `postiz-post-with-image` skill, under "Capturing a frame from an embedded video". Its pixel limits apply.

State the image's source in the final message. It is usually the source company's artwork, and the user decides whether to use it. Mention if it shows named people.

## The composer

Order matters. Do these in this order:

1. Open it: `await page.goto("https://www.linkedin.com/feed/")`, then `await page.click("text=Start a post")`. The editor is a `contenteditable` div in a dialog. Use `page.snapshot()` and find the `textfield` ref.
2. **Attach the image first, before any text.** A URL typed into the box starts a link preview and the Media button disappears. Arm the chooser before clicking:

```js
const chooserPromise = page.waitForFileChooser({ timeout: 20000 });
await page.click('loc=css:button[aria-label="Media"]');
const chooser = await chooserPromise;
await chooser.setFiles("/absolute/path/to/image.png");
await page.waitForTimeout(3000);
await page.click('loc=role:button[name="Next"]');
```

   Clicking Media without arming the chooser makes Ego cancel the OS dialog and the Editor modal sticks on "Loading". If that happens, click Back and try again.
3. **Type the text line by line.** `keyboard.paste()` and `insertText()` of a multi-line string drop the blank lines. Instead:

```js
await page.evaluate(() => document.querySelector('div[contenteditable="true"][role="textbox"]').focus());
for (let i = 0; i < lines.length; i++) {
  if (lines[i] !== "") await page.keyboard.insertText(lines[i]);
  if (i < lines.length - 1) await page.keyboard.press("Enter");
}
```

   An empty string in `lines` makes a blank line. Consecutive lines stay tight.
4. **Check before posting.** Read the editor structure (`[...el.children].map(c => c.innerText.trim())`). Scroll with `await page.mouse.move(700, 350); await page.mouse.wheel(0, 900)` to see the image under the text. Take a screenshot and open it. Confirm visibility reads "Post to Anyone".
5. **Post:** `await page.click('loc=role:button[name="Post"]')`. A spinner shows on the button for a few seconds.

## Verify

Go to `https://www.linkedin.com/in/<handle>/recent-activity/all/` and confirm the new post is first, shows "now", and has its image. Click "more" to expand it and look at the paragraph spacing in a screenshot.

LinkedIn rewrites the source link into an `lnkd.in` short link and links bare domains such as `claude.ai`. This is LinkedIn's behavior and cannot be turned off here; tell the user.

The newest post usually has no analytics link yet, so the exact post URL may not be readable. Report the activity-page link in that case.

## Errors seen so far

- `Invalid locator: loc=role:textbox`: a role locator needs a name, such as `loc=role:button[name="Post"]`. Use a snapshot ref (`@88`) for the editor.
- `intercepts pointer events` after a failed upload: the Editor modal is still open. Click Back.
- `page.screenshot()` size: capped by the viewport; use the CDP capture above.
- A click on the "now" timestamp opened the profile, not the post. Do not rely on it for the link.

## Not covered

Posting as a company page, video or document posts, scheduling, and editing or deleting a live post are untested. Ask the user before trying them. Deleting a live post removes its reactions and comments.
