# Portfolio — Md. Rasul Islam Bapary

Personal portfolio of an AI engineer working on computer vision, speech, LLM and agentic systems.

**Live site:** https://rasul-ai.github.io/portfolio/

## Visit alerts on/off (per browser)

Every visit sends an alert to the owner's Telegram. To keep your own browsing out of the alerts, open
this link once in each browser you use:

| Link | Effect |
| --- | --- |
| https://rasul-ai.github.io/portfolio/?notrack | Stops alerts from this browser |
| https://rasul-ai.github.io/portfolio/?track | Turns alerts back on for this browser |

- The setting is saved in the browser's local storage, so it lasts across restarts. Each browser and
  each browser profile needs it once.
- Private/incognito windows forget it when closed, which makes them handy for testing that alerts work.
- Clearing the site's data resets it; open the `?notrack` link again.
- To check the current state, open the browser console (F12): with alerts off it prints
  "Visit alerts are off in this browser."
- The contact form always delivers, whatever this setting is.

## What's implemented

A static, single-page site built from the CV (`Rasul_Bapary_CV.pdf` is the source of truth), with no
framework and no build step.

- **Content:** intro and fact sheet, experience by role (18 professional projects, grouped as in the
  CV), 20 independent projects, the MMF-Gait paper (*Symmetry* 2025) and Medium article, skills,
  education, references (names, titles and affiliations only) and contact.
- **Project demos:** cards show a poster and play a short silent preview on hover or focus (when
  in view on touch screens). Clicking opens a viewer dialog with the full video, image, slides or
  embed, a system outline and links. Prev/next and arrow keys move between projects, and each
  project has a shareable `#demo/<slug>` link.
- **Independent projects:** an index with area filters, search and a live result count.
- **Research:** expandable abstract plus "Copy citation" and "Copy BibTeX".
- **Contact:** copy-email button and a message form with sending, sent and error states. Messages
  reach Telegram through the Worker.
- **Visit alerts:** see below.
- **Design:** Mona Sans and JetBrains Mono, warm paper/ink palette with one green accent, light and
  dark themes (follows the OS, toggle persists, no flash on load), responsive layout, print styles.
- **Quality:** semantic HTML that reads without JavaScript, keyboard access throughout, AA contrast
  in both themes, lazy-loaded media with fixed dimensions, previews off under reduced motion or
  Save-Data, Open Graph tags, JSON-LD `Person` and an SVG favicon.

Not yet verified: video playback on Safari/iOS, and the mobile menu without JavaScript.

## Project structure

| Path | What it is |
| --- | --- |
| `index.html` | The whole site: sections, project cards, viewer dialog |
| `style.css` | Plain CSS, no build step; light/dark tokens at the top |
| `script.js` | Theme toggle, project viewer, hover previews, contact form, visit alerts |
| `assets/` | Fonts, images, posters, demo videos and their short hover previews |
| `Rasul_Bapary_CV.pdf` | The CV linked from the site |
| `worker/` | Cloudflare Worker that relays visit alerts and contact messages to Telegram |

## Running locally

No build step. Serve the folder and open it:

```sh
python3 -m http.server 8000
# http://localhost:8000
```

Visit alerts only run on `rasul-ai.github.io`, so local browsing never sends them.

## Deploying

The site is served by GitHub Pages from the `main` branch. Push to `main` and it updates within a
minute or two.

## How visit alerts work

```
browser (script.js) ──POST──▶ Cloudflare Worker (worker/) ──▶ Telegram Bot API ──▶ owner's chat
```

1. A new visit sends one alert with location, network, device, browser, source and whether the visitor is returning.
2. A CV download sends a reply to that alert.
3. Leaving or hiding the tab edits the alert with a short summary: active time, scroll depth,
   sections viewed, demos opened, links clicked.
4. Contact-form messages arrive as a reply to the sender's visit alert.

Automated link scanners (for example email security services) often load the page and close it
abruptly, so their alerts never get a summary.

The bot token and chat ID live only in Cloudflare as Worker secrets. They are never in the page or
in this repository.

### Worker commands

Run from `worker/`:

```sh
npx wrangler@4 login                              # once per machine
npx wrangler@4 deploy                             # after changing worker/src/index.js
npx wrangler@4 secret put TELEGRAM_BOT_TOKEN      # set or replace the bot token
npx wrangler@4 secret put TELEGRAM_CHAT_ID        # set the chat that receives alerts
npx wrangler@4 secret list                        # show which secrets are set (names only)
npx wrangler@4 tail                               # live logs, useful when alerts stop
```

Changing a secret takes effect immediately; no deploy or git push is needed.

### Rotating the Telegram bot token

1. In Telegram, send `/revoke` to @BotFather and pick the bot. Copy only the new token.
2. Run `npx wrangler@4 secret put TELEGRAM_BOT_TOKEN` in `worker/` and paste it.
3. Open the site in a private window and confirm an alert arrives.

If alerts stop, `npx wrangler@4 tail` shows Telegram's reason. `Not Found` means the stored token is
malformed (check the paste); `Unauthorized` means it was revoked.

## Adding a project

Each project is an `<article class="project">` (or a compact/index row) in `index.html`. The viewer
reads its `data-*` attributes:

| Attribute | Meaning |
| --- | --- |
| `data-dialog` | Unique slug, also used in the `#demo/<slug>` link |
| `data-kind` | `video`, `image` or `embed` |
| `data-src` | The video, image or embed URL shown in the viewer |
| `data-poster` | Poster image for videos |
| `data-aspect` | Media aspect ratio, e.g. `16 / 10` |
| `data-context` | Small label above the title (company · area) |
| `data-caption` | Caption under an image |
| `data-outline` | System outline steps, separated by `\|` |
| `data-note` | Note under the outline |
| `data-drive`, `data-slides`, `data-repo`, `data-live` | Optional external links |

A card with `<video class="preview" data-src="…">` plays that short clip on hover.

Don't publish demos that show personal data (ID cards, names, ID numbers, addresses). Use fake
sample data or a diagram instead.
