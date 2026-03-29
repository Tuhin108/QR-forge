# 🎯 QR Forge — Free Custom QR Code Generator

> **Dynamic redirects · Artistic image-style · Permanent embed · Logo support · PNG & SVG download**

Built with Python, Streamlit, segno, and Pillow by **Tuhin Kumar Singha Roy**
🔗 [LinkedIn](https://www.linkedin.com/in/tuhininaiml) · 💻 [GitHub](https://github.com/Tuhin108)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Project Structure](#-project-structure)
- [Modes of Operation](#-modes-of-operation)
  - [Dynamic QR Code](#1️⃣-dynamic-qr-code-with-redirect)
  - [Artistic QR Code](#2️⃣-artistic-qr-code)
  - [Permanent QR Code](#3️⃣-permanent-qr-code)
- [Analytics & Data Management](#-analytics--data-management)
- [GitHub Auto-Push](#-github-auto-push)
- [SEO Optimization](#-seo-optimization)
- [Tech Stack](#-tech-stack)
- [Installation & Setup](#-installation--setup)
- [Configuration](#-configuration)
- [Environment Variables & Secrets](#-environment-variables--secrets)
- [Usage Guide](#-usage-guide)
- [API / Architecture Notes](#-architecture-notes)
- [Comparison: Dynamic vs Permanent QR](#-dynamic-vs-permanent-qr-comparison)

---

## 🌐 Overview

**QR Forge** is a full-featured, free online QR code generator deployed as a Streamlit web application. It goes well beyond basic QR generation by offering three distinct modes: trackable dynamic QR codes with expiry-based redirects, artistic QR codes that visually resemble an uploaded image, and permanent QR codes that embed the destination URL directly without any server dependency.

The app is designed to be production-ready with SEO meta injection, structured JSON-LD data, scan analytics with GitHub auto-push, and a clean, polished UI.

---

## ✨ Features

- **Three QR modes** — Dynamic (trackable redirect), Artistic (image-mapped), and Permanent (serverless)
- **Custom colors** — Foreground and background color pickers for full branding control
- **Logo overlay** — Embed a logo image at the center of any QR code
- **Background image blending** — Blend a background image into the QR pattern (Dynamic mode)
- **Error correction control** — Choose from L (7%), M (15%), Q (25%), or H (30%)
- **Expiry timer** — Set a link lifetime from 1 minute up to 7 days
- **Scan analytics** — Each token tracks scan count, creation time, and expiry
- **GitHub auto-push** — Analytics JSON is automatically committed to a GitHub repo after every scan
- **PNG & SVG download** — Download QR codes in both raster and vector formats
- **SEO optimized** — Meta tags, Open Graph, Twitter Cards, and JSON-LD structured data injected at runtime
- **Redirect handling** — The app intercepts `?r=...&t=...` query parameters and performs client-side redirects
- **Expired link page** — Gracefully handles expired or invalid tokens with a styled error page

---

## 📁 Project Structure

```
qr-forge/
│
├── streamlit_app.py        # Main application — all logic and UI
├── analytics.json          # Persisted QR token store (auto-updated)
├── requirements.txt        # Python dependencies
└── README.md               # This file
```

> **Note:** `analytics.json` (named `s.json` internally at runtime) is both stored locally and pushed to GitHub automatically whenever scan data changes.

---

## 🔧 Modes of Operation

### 1️⃣ Dynamic QR Code (with Redirect)

This mode generates a QR code that points to a **short-link on the QR Forge server** rather than directly to the target URL. The short-link:

- Is identified by a UUID token (e.g., `?r=https://...&t=<uuid>`)
- Redirects the scanner to the real destination URL
- Tracks each scan by incrementing a counter
- Expires after a user-defined duration (1 minute to 7 days)

**How it works internally:**
1. A UUID token is generated and stored with the target URL, creation timestamp, expiry timestamp, and scan count in `analytics.json`.
2. The redirect URL (`APP_URL?r=<encoded_url>&t=<token>`) is encoded into the QR.
3. When scanned, the app intercepts the `?t=` query parameter, looks up the token, checks expiry, increments the scan count, and performs a JavaScript redirect via `window.top.location.href`.
4. If the token is expired or not found, the user sees a styled "Invalid or Expired" error page.

**Options available:**
- Target URL
- Foreground / background color
- Error correction level (L / M / Q / H)
- Expiry duration (minutes)
- Scale / size
- Optional background image (blended at 65% alpha)
- Optional logo overlay (max 25% of QR size, centered with a white background pad)

---

### 2️⃣ Artistic QR Code

This mode maps the **pixel colors of an uploaded image** onto the QR modules, resulting in a QR code that visually resembles the source image while remaining fully scannable.

**How it works internally:**
1. The source image is resized to the QR matrix dimensions (`n_cols × n_rows`).
2. For each dark module in the QR matrix, the corresponding pixel color from the resized image is applied (at 75% brightness to ensure contrast).
3. Finder patterns (the three corner squares) are always rendered in pure black/white to guarantee scannability.
4. Light modules are rendered according to the chosen background style.
5. An UnsharpMask filter is applied at the end to sharpen edges (except in Transparent mode).

**Options available:**
- Target URL (also uses the token-based redirect system)
- Image upload (PNG, JPG, JPEG, WEBP)
- Error correction (M / Q / H — minimum M recommended for complex images)
- Background style:
  - **White** — light modules are pure white (most scannable)
  - **Image tint** — light modules get a faint tint from the image colors (15% blend)
  - **Transparent** — light modules are left transparent (for PNG overlays)
- Color saturation (0.5–2.5)
- Module size in pixels (8–24px)
- Expiry duration (also uses the redirect system)

> ⚠️ **Tip embedded in UI:** Always test scan before sharing. If it fails to scan, switch to White background.

---

### 3️⃣ Permanent QR Code

This mode encodes the destination URL **directly into the QR pattern** with no redirect, no token, and no server dependency. The QR code will work indefinitely — even if the QR Forge app goes offline.

**How it works internally:**
It calls the same `generate_qr_code()` function used by Dynamic mode but passes the raw URL as data instead of a redirect URL. No entry is written to `analytics.json`.

**Trade-offs vs Dynamic:**

| Feature | Dynamic QR | Permanent QR |
|---|---|---|
| Works forever | ❌ (expires) | ✅ |
| Scan analytics | ✅ | ❌ |
| Change destination | ✅ (re-generate) | ❌ |
| Needs server to scan | ✅ | ❌ |
| Pattern complexity | Shorter URL = simpler | Depends on URL length |

**Options available:**
- Destination URL
- Foreground / background color
- Error correction level (L / M / Q / H)
- Scale / size
- Optional logo overlay

---

## 📊 Analytics & Data Management

Analytics are stored in a flat JSON file (`analytics.json` / `s.json`). Each entry uses a UUID as the key:

```json
{
  "<uuid-token>": {
    "url": "https://target-url.com",
    "created_at": "2026-03-29T04:58:46.372097+00:00",
    "expires_at": "2026-03-29T05:58:46.372097+00:00",
    "scan_count": 1
  }
}
```

**Data operations:**
- `load_data()` — reads and parses the JSON file; returns empty dict on failure
- `save_data(data)` — writes to local file and triggers GitHub push
- `store_redirect(token, url, expiry_minutes)` — creates a new token entry
- `get_redirect(token)` — retrieves a token entry and validates expiry (handles legacy naive timestamps)
- `increment_scan_count(token)` — atomically increments the scan counter for a token

---

## 🐙 GitHub Auto-Push

Every time `save_data()` is called (i.e., on every new QR generation and every scan), the app automatically commits the updated `analytics.json` to a GitHub repository using the GitHub Contents API.

**Mechanism:**
1. Reads the current file SHA from the repo via `GET /repos/{repo}/contents/{path}`
2. Base64-encodes the updated JSON
3. Pushes via `PUT` with the commit message `"📊 Auto-updating QR scan analytics"`

**Requires:** A `GITHUB_TOKEN` secret in Streamlit's secrets manager (`st.secrets`). If not present, the push is silently skipped and only local save occurs.

---

## 🔍 SEO Optimization

The app injects SEO metadata into the parent document `<head>` at runtime using a zero-height `components.html` JavaScript block (the only reliable method in Streamlit's iframe-based rendering):

- **Primary meta tags** — `description`, `keywords`, `robots`, `author`
- **Open Graph tags** — `og:type`, `og:title`, `og:description`, `og:url`, `og:image`, `og:site_name`
- **Twitter Card tags** — `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- **Canonical link** — `<link rel="canonical">`
- **JSON-LD structured data** — `WebApplication` schema with feature list, author, and pricing info

---

## 🛠️ Tech Stack

| Library | Purpose |
|---|---|
| **Streamlit** | Web UI framework and app server |
| **segno** | QR code generation (PNG, SVG) |
| **Pillow (PIL)** | Image processing — logo overlay, blending, artistic rendering, sharpening |
| **NumPy** | Pixel array manipulation for artistic QR color mapping |
| **Pandas** | Listed in requirements (available for analytics extensions) |
| **Requests** | GitHub API calls for analytics auto-push |

---

## 🚀 Installation & Setup

### Prerequisites
- Python 3.9 or higher
- pip

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/Tuhin108/qr-forge.git
cd qr-forge

# 2. (Optional) Create a virtual environment
python -m venv venv
source venv/bin/activate       # Linux/macOS
venv\Scripts\activate          # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the app
streamlit run streamlit_app.py
```

The app will be available at `http://localhost:8501`.

---

## ⚙️ Configuration

| Parameter | Location | Default | Description |
|---|---|---|---|
| `BASE_URL` | Environment variable | `""` | Override the app's public URL (used in redirect links) |
| `APP_URL` | Derived from `BASE_URL` | `https://qr-forge.streamlit.app` | Base URL embedded in dynamic QR codes |
| `DATA_FILE` | Hardcoded | `s.json` | Local analytics storage filename |
| `MAX_EXPIRY_DAYS` | Hardcoded | `7` | Maximum allowed expiry duration in days |

---

## 🔐 Environment Variables & Secrets

For Streamlit Cloud deployments, add the following in **Settings → Secrets**:

```toml
# .streamlit/secrets.toml

GITHUB_TOKEN = "ghp_your_personal_access_token"
```

The token requires **`repo` scope** (write access to contents) on the target repository (`Tuhin108/qr-forge`).

For a custom domain or self-hosted deployment, also set:

```bash
BASE_URL=https://your-custom-domain.com
```

---

## 📖 Usage Guide

### Generating a Dynamic QR Code
1. Open the **✏️ Dynamic QR** tab
2. Enter a target URL (must start with `http://` or `https://`)
3. Optionally pick foreground/background colors, upload a logo or background image
4. Set the expiry duration (how long before the short-link stops working)
5. Click **🎨 Generate Dynamic QR**
6. Download the QR as PNG or SVG

### Generating an Artistic QR Code
1. Open the **🖼️ Artistic QR** tab
2. Enter a target URL
3. Upload an image (PNG, JPG, JPEG, or WEBP)
4. Adjust saturation, module size, background style, and error correction
5. Click **🎨 Generate Artistic QR**
6. Test scan before sharing — use White background if scanning fails

### Generating a Permanent QR Code
1. Open the **🔒 Permanent QR** tab
2. Enter the destination URL
3. Optionally configure colors, error correction, scale, and logo
4. Click **🔒 Generate Permanent QR**
5. Download and use — it will never expire

---

## 🏗️ Architecture Notes

- **Redirect flow:** The app uses Streamlit's `st.query_params` to detect incoming `?t=` tokens on page load. The redirect itself is performed via `window.top.location.href` injected through `components.html`, which ensures the parent window (not the Streamlit iframe) navigates away.
- **Timezone safety:** Token expiry comparisons always use timezone-aware UTC datetimes. Legacy naive timestamps stored without `tzinfo` are patched on read to avoid `TypeError` crashes.
- **Logo centering:** Logos are resized to a maximum of 25% of the QR dimension, given a white background pad, and composited at the center using alpha blending.
- **Finder pattern preservation:** In Artistic mode, the three finder squares (top-left, top-right, bottom-left corners) are always rendered in pure black and white regardless of the source image, as these are critical for scanner alignment.

---

## 👤 Author

**Tuhin Kumar Singha Roy**
- 🔗 [LinkedIn — tuhininaiml](https://www.linkedin.com/in/tuhininaiml)
- 💻 [GitHub — Tuhin108](https://github.com/Tuhin108)


---

*QR Forge · Free QR Code Generator · Custom · Artistic · Permanent QR codes*
