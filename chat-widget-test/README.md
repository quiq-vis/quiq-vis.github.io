# Quiq Widget CSS Test Harness

A self-hosted copy of the Quiq chat-demo page
(`https://seanc.goquiq.com/app/chat-demo/chat-ui/default`) with one addition:
it passes a `customStylesheet` into the `Quiq()` init so the CSS in
`quiq-overrides.css` is injected **inside** the chat widget. Use it to test
styling of rich-message cards tagged with `cssClassName`.

## Run locally

```bash
cd quiq-widget-host
python3 -m http.server 8000
```

Open <http://localhost:8000>. The footer badge must show
`✓ quiq-overrides.css loaded`. (Chrome treats `localhost` as a secure
context, so the widget accepts the stylesheet URL without HTTPS.)

## Test a styled card

1. Open the chat widget on the page and start a conversation.
2. From the bot / agent side, send a rich message tagged with the CSS class:

   ```json
   {
     "text": "Hi",
     "assets": [],
     "messageAttributes": {
       "excludeFromLinkScraping": false,
       "transcriptVisibilityOptions": { "transcriptHintsOnly": false },
       "cssClassName": "custom-blue-card"
     }
   }
   ```

3. The tagged card renders with a blue background, white text, and a styled
   button (rules in `quiq-overrides.css`).
4. Edit `quiq-overrides.css`, save, and refresh the page — the init script
   appends `?v=<timestamp>` to the stylesheet URL, so you always get the
   fresh file.

## If the chat panel opens but spins forever

The page renders, the launcher bubble appears, but clicking it shows an
endless loading spinner. The widget swallows the real error; the browser
console shows:

```
Error: Unable to initialize webchat (error in Launcher initialization):
The domain "..." is not allowed to load webchat for this contact point.
Make sure to add this domain to your 'Whitelisted Domains' in the admin app.
```

Fix in the Quiq admin app: Contact Points → your contact point → Web Chat →
**Whitelisted Domains** — add the domain hosting this page (`localhost` for
local dev, `quiq-vis.github.io` for the published copy). Note: Quiq mangles
the origin when reporting it — `localhost:8000` shows up as `localhost00`
(the string `:80` is stripped) — so match on the domain, not the error text.

## If your CSS doesn't apply

- **Badge shows ✗** — the stylesheet URL is unreachable (wrong path, server
  not running, or blocked). Fix that first; nothing else matters until ✓.
- **Badge shows ✓ but the card is unstyled** — open DevTools, inspect the
  card, and check the actual class names Quiq rendered; update the selectors
  in `quiq-overrides.css` to match. Keep `!important` on every declaration.
- **Stylesheet never appears inside the widget frame** — `customStylesheet`
  may be ignored when initializing via `pageConfigurationId`. In
  `index.html`, comment out the `pageConfigurationId` line and uncomment
  `contactPoint: "default"` instead.

## Publish to GitHub Pages

The public URL is `https://quiq-vis.github.io/chat-widget-test/`.

```bash
mkdir -p ../quiq-vis.github.io/chat-widget-test
cp index.html quiq-overrides.css README.md ../quiq-vis.github.io/chat-widget-test/
cd ../quiq-vis.github.io
git add chat-widget-test
git commit -m "Add Quiq widget CSS test harness"
git push
```

Wait ~1 minute for Pages to deploy, then re-run the card test at the public
URL.

> **The `quiq-vis/quiq-vis.github.io` repo is PUBLIC.** Never put tenant
> secrets, tokens, or customer data in these files.
