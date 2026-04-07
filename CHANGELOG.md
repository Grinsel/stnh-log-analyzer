# Changelog

## v1.1.0 (2026-04-07)

### Added
- **Log Library**: Uploaded logs are now persistently stored in the browser (IndexedDB) and can be re-used across sessions
  - Each upload is saved with timestamp and an optional custom name
  - Saved logs are listed in a library panel alongside the upload area
  - Users can select one or multiple saved logs for analysis without re-uploading
  - Logs can be individually or bulk-deleted from the library
- **Online hosting**: App is now available via GitHub Pages — no need to download the HTML file
- **New Analysis** button in tab navigation to return to log selection and start fresh
- `index.html` entry point for GitHub Pages deployment

### Changed
- Upload flow redesigned: files are staged first, then analyzed via explicit "Analyze" button
- Upload area now side-by-side with log library on wider screens
- Version bumped to 1.1.0

## v1.0.0 (2026-04-07)

- Initial release
- Dashboard, Revenue, Stats, Rankings, Wars, Spreadsheet, Faction Detail, Meta Analysis tabs
- Multi-file support with cross-file comparison
- Human player detection and highlighting
- Custom canvas chart for revenue/stats comparison
