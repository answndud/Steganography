# Repository Guidelines

## Project Structure & Module Organization
- `index.html` contains the full app (HTML, CSS, and vanilla JS in one file). Keep UI, styles, and logic together unless a refactor is explicitly requested.
- `README.md` is the user-facing overview, while `TECHNICAL.md` holds deeper implementation notes.
- There are no separate assets or test directories; images and files are loaded by users at runtime.

## Build, Test, and Development Commands
- No build step or bundler is used. Open `index.html` directly in a browser for local development.
- For local static hosting (recommended for file API behavior):
  - `python3 -m http.server 8000` and visit `http://localhost:8000`.
- There are no automated tests configured.

## Coding Style & Naming Conventions
- Indentation: 4 spaces in HTML/CSS/JS, matching the existing file.
- JavaScript: use `camelCase` for functions/variables (`encodeDataInImage`), `UPPER_SNAKE_CASE` for constants (`DELIMITER`, `HEADER_LENGTH`).
- Keep user-facing strings in Korean to match the current UI.
- External dependencies are loaded via CDN (CryptoJS, Google Fonts). Avoid adding new dependencies without a clear need.

## Testing Guidelines
- Manual testing only: encode a short message, download the PNG, then decode it with the same key.
- Validate error paths: wrong key, oversized message, and unsupported file types.
- No test naming conventions apply until a test framework is added.

## Commit & Pull Request Guidelines
- Git history has only an initial commit, so no established message convention exists yet.
- Prefer concise commit messages (e.g., "Add decode error handling").
- PRs should include: a short summary, before/after screenshots for UI changes, and steps to verify (commands or manual flows).

## Security & Configuration Notes
- Encryption and steganography run client-side only; do not add server calls that transmit images or secrets.
- If you change encryption, update both `README.md` and `TECHNICAL.md` to keep documentation aligned.
