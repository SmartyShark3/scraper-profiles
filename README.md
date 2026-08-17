# webnovel-offline-profiles

Community-maintained selector profiles for [webnovel-offline](https://github.com/jrick/webnovel-offline).

Each JSON file defines CSS selectors and scraper configuration for a supported web novel hosting site. The main app loads these profiles at runtime to extract novel metadata and chapter content.

## Structure

| File | Site |
|------|------|
| `royalroad.json` | [Royal Road](https://www.royalroad.com) |
| `scribblehub.json` | [ScribbleHub](https://www.scribblehub.com) |
| `ranobes.json` | [Ranobes](https://ranobes.net) |
| `wuxiaworld.json` | [WuxiaWorld](https://www.wuxiaworld.com) |
| `index.json` | Manifest listing all active profile filenames |

## Adding a New Site

1. Copy an existing profile JSON as a template.
2. Set `"domain"` to the site's base domain (e.g. `"novelupdates.com"`).
3. Fill in the CSS selectors — test against a real novel page.
4. Add the filename to `index.json`.
5. Open a pull request.

## Profile Schema

```jsonc
{
  "domain": "example.com",              // required — base domain
  "titleSelector": "h1.title",          // required
  "authorSelector": ".author",          // required
  "coverImageSelector": ".cover img",   // optional
  "summarySelector": ".description",    // optional
  "tagSelector": "a.tag",              // optional
  "chapterLinkSelector": "ol li a",    // required
  "chapterContentSelector": ".content", // required
  "chapterDateSelector": ".pub-date",  // optional
  "ratingSelector": "meta[itemprop='ratingValue']", // optional
  "chapterUrlRegex": "https://...",    // optional — used for URL normalization
  "novelUrlTemplate": "https://...",   // optional — used for URL normalization
  "requiresWebView": false,            // default false
  "isCustom": false,                   // always false for built-in profiles
  "initials": "RR",                    // optional — short label shown in UI
  "watermarkPhrases": [],              // optional — phrases to strip from content
  "topAuthorNotesSelector": ".author-note-top",    // optional
  "bottomAuthorNotesSelector": ".author-note-bot", // optional
  "reverseChapterOrder": false         // set true if site lists newest chapter first
}
```

## License

MIT
