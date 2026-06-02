---
name: tt-download
description: "Resolve the signed video URL behind an Oceanengine 素材中心 video_player page and optionally save it as MP4."
homepage: https://docs.openclaw.ai/
license: MIT
metadata:
  {
    "openclaw":
      {
        "emoji": "🎬",
        "requires": { "bins": ["python3"] },
        "primaryEnv": "",
        "install":
          [
            {
              "id": "brew-python",
              "kind": "brew",
              "formula": "python@3.11",
              "bins": ["python3"],
              "label": "Install Python 3 (brew)",
            },
          ],
      },
  }
---

# tt-download

Resolve the real (signed) video URL hiding behind an Oceanengine 素材中心 `video_player` page — the SPA at `https://ad.oceanengine.com/material_center/outer/video_player?token=...` that injects `<video src=...>` via JS. Optionally stream the result to disk.

## When to use

Trigger when the user shares a URL matching `https://ad.oceanengine.com/material_center/outer/video_player?token=...` (or asks to save / grab / 真实地址 / 下载 such a video). Do **not** trigger for general video downloading — yt-dlp / video-frames cover that.

## Why a skill is needed

`curl` / `requests` / `yt-dlp` cannot recover the URL: the page is a React SPA, the resolved `cc.oceanengine.com` URL sets a `vck_*` cookie the final CDN requires (403 `X-Moat-Code 4119` without it), and the signed `video-cn.oceanengine.com` URL has a ~5-minute TTL. The tool renders the SPA in headless Chrome, follows the 302 manually to preserve the cookie, then streams the body — end-to-end in one process.

## Quick start

```bash
# Print the resolved URL only (stdout, one line)
{baseDir}/scripts/tt-download 'https://ad.oceanengine.com/material_center/outer/video_player?token=...'

# Download to disk
{baseDir}/scripts/tt-download '...token=...' -o video.mp4

# URL from a file (long tokens are tedious on the shell)
{baseDir}/scripts/tt-download @url.txt -o video.mp4

# Also echo the intermediate cc.oceanengine.com URL (stderr)
{baseDir}/scripts/tt-download @url.txt -o video.mp4 --show-intermediate
```

## Output contract

- **stdout**: the resolved `https://video-cn.oceanengine.com/...` URL (one line). Empty on failure.
- **stderr**: progress + errors. `intermediate: <cc-url>` when `--show-intermediate`. `✅ 已下载 16.4 MB → video.mp4` after a successful `-o`.
- **exit 0** = success; **1** = extract / download failure (token expired, browser missing, network, HTTP 4xx/5xx); **2** = bad CLI usage.

If the user only wants the URL, run without `-o` and hand them stdout. If they want a file, use `-o`.

## Files in this skill

```
tt-download/
├── SKILL.md                     # this file (agent-facing)
├── scripts/
│   ├── tt_download.py           # the tool (stdlib only)
│   └── tt-download              # thin bash wrapper → exec python3
├── references/
│   ├── usage.json               # machine-readable CLI schema + examples
│   ├── chrome-paths.json        # browser discovery paths per OS (for audits / extensions)
│   └── troubleshooting.md       # failure-mode matrix (load on demand)
├── agents/
│   └── openai.yaml              # UI metadata for OpenClaw Skills UI
└── LICENSE
```

## References (load on demand)

- For full argument schema, exit codes, and invocation examples in machine form: read `{baseDir}/references/usage.json`.
- For browser discovery paths across macOS / Linux / Windows, or to add a custom browser location: read `{baseDir}/references/chrome-paths.json`.
- For failure modes and recovery: read `{baseDir}/references/troubleshooting.md`.

## Notes

- Headless Chrome / Edge is required (auto-detected at default install paths). `python3` is the only hard `requires.bins` gate.
- No `pip install` — pure Python 3.7+ stdlib.
- The signed URL contains a `policy=` JWT-style token with ~5-minute TTL. **Extraction and download must happen in the same process** — never cache the URL for later.

## Publishing to ClawHub

```bash
clawhub publish ./tt-download --slug tt-download --name "tt-download" --version 1.0.0 --changelog "Initial release"
```

Users install with `openclaw skills install tt-download` (or `clawhub install tt-download`).
