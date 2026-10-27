# PdfTrans for Android

Translate English PDFs into Arabic on your phone. Open source (MIT), no Telegram, no server of your own.
It is a native Kotlin port of `pdftrans.py` / `bot.py`.

## Engines
| Engine | Where text goes | Notes |
|---|---|---|
| On device (ML Kit) | stays on the phone | one-time ~30 MB Arabic model, then fully offline |
| Google (unofficial) | Google | best quality, opt-in only, may break or violate Google's terms |
| MyMemory | MyMemory | official free API, ~5k chars/day (50k with an email in Settings) |
| LibreTranslate | your server | self-hosted, optional API key |

## Layouts (same as the bot)
- **Smart**: keeps the original pages, cuts them on blank scan-lines and inserts room for the Arabic under each line.
- **Line by line**: Arabic under every line at a fixed size, pages unchanged.
- **Paragraph**: rebuilds the document as text, each paragraph followed by its translation.

Scanned PDFs are read with on-device OCR (ML Kit, Latin). Pages can be limited with `1-10,15,20-`.

## Build the APK without a PC
1. Create a GitHub repository and upload this folder.
2. Open the **Actions** tab, run **Build APK** (it also runs on every push).
3. Download the `PdfTrans-apk` artifact, unzip, install `app-release.apk`.

The workflow downloads the Arabic fonts (Amiri, IBM Plex Sans Arabic, Scheherazade New, all SIL OFL).
If a font download fails the app still builds and uses Android's built-in Arabic font.

## Build with Android Studio
Open the folder, run `bash scripts/fetch_fonts.sh`, then *Build > Build APK(s)*.
(Studio creates the Gradle wrapper for you; the project needs JDK 17, AGP 8.5.2, compileSdk 34.)

## Python to Kotlin map
| Python | Kotlin |
|---|---|
| `extract` / `fitz` text blocks | `pdf/Extract.kt` (PDFBox-Android words, grouped into lines and blocks) |
| OCR (`rapidocr`) | `ocrLines()` with ML Kit text recognition |
| `translate_texts` + engines | `engine/Engines.kt` (batching, cache, retries, circuit breaker) |
| `_render_overlay`, `_Ink` | `pdf/Render.kt` `renderOverlay`, `Ink` |
| `_render_paragraphs` | `pdf/Render.kt` `renderParagraphs` |
| `translate_pdf` | `pdf/Pipeline.kt` |
| Telegram menus | `ui/Screens.kt` (Jetpack Compose) |

Arabic shaping and right-to-left layout are done by Android's text engine (`StaticLayout`), then the result is merged onto the original pages with PDFBox, so the original pages stay vector.

## Known limits
- This project was written without an Android toolchain available, so it has **not been compiled or run yet**. If the first CI build reports an error, send me the log and I'll fix it.
- ML Kit (translation and OCR) runs locally but is Google's closed-source library, so the app cannot go on F-Droid as is. A fully open-source offline engine (Bergamot or an Opus-MT model through ONNX Runtime) can be added behind the same `Engine` class.
- Translating a long file needs the app to stay open; there is no background service yet.
- Pages with /Rotate are left untouched in Smart and Line modes; use Paragraph for them.
- The release APK is signed with the debug key so it installs directly. Use your own keystore before distributing it.

## Licences
App: MIT. PDFBox-Android: Apache-2.0. Fonts: SIL OFL. ML Kit: Google ML Kit terms.
