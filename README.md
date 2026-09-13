# webnovel-offline-profiles

Community-maintained selector profiles for [webnovel-offline](https://github.com/jrick/webnovel-offline).

Each JSON file defines CSS selectors, scraping rules, and network behavior for a supported web novel hosting site. The Android app bundles these profiles at build time and periodically synchronizes updates from remote at runtime, allowing selector fixes and new sites without requiring a full app release.

---

## Supported Sites

| File | Site | Domain | Badge | Notes |
|------|------|--------|:-----:|-------|
| [`archiveofourown.json`](archiveofourown.json) | [Archive of Our Own (AO3)](https://archiveofourown.org) | `archiveofourown.org` | `AO` | Preseeded adult cookie, landing page handling, noise filtering |
| [`boxnovel.json`](boxnovel.json) | [BoxNovel](https://boxnovel.com) | `boxnovel.com` | `BN` | Standard Wp-Manga structure |
| [`createnovels.json`](createnovels.json) | [CreateNovels](https://createnovels.com) | `createnovels.com` | `CN` | Standard WordPress layout |
| [`freewebnovel.json`](freewebnovel.json) | [FreeWebNovel](https://freewebnovel.com) | `freewebnovel.com` | `FWN` | Standard layout |
| [`lightnovelpub.json`](lightnovelpub.json) | [Light Novel Pub](https://lightnovelpub.com) | `lightnovelpub.com` | `LNP` | Standard layout |
| [`literotica.json`](literotica.json) | [Literotica](https://literotica.com) | `literotica.com` | `LT` | Preseeded adult consent cookies, multi-page story pagination |
| [`mtlnovel.json`](mtlnovel.json) | [MTLNovel](https://mtlnovel.com) | `mtlnovel.com` | `MTL` | Standard layout |
| [`novelfull.json`](novelfull.json) | [NovelFull](https://novelfull.com) | `novelfull.com` | `NF` | Standard layout |
| [`ranobes.json`](ranobes.json) | [Ranobes](https://ranobes.net) | `ranobes.net` | `RN` | Standard layout |
| [`readlightnovel.json`](readlightnovel.json) | [Read Light Novel](https://readlightnovel.me) | `readlightnovel.me` | `RLN` | Standard layout |
| [`royalroad.json`](royalroad.json) | [Royal Road](https://www.royalroad.com) | `royalroad.com` | `RR` | Anti-scraper watermark scrubbing, author note extraction |
| [`scribblehub.json`](scribblehub.json) | [ScribbleHub](https://www.scribblehub.com) | `scribblehub.com` | `SH` | Reversed chapter order, author notes |
| [`storiesonline.json`](storiesonline.json) | [StoriesOnline](https://storiesonline.net) | `storiesonline.net` | `SO` | Dynamic AJAX detail enrichment, 4.5s rate limit, login URL template |
| [`wuxiaworld.json`](wuxiaworld.json) | [WuxiaWorld](https://wuxiaworld.eu) | `wuxiaworld.eu` | `WW` | Standard layout |
| [`index.json`](index.json) | Manifest | — | — | Listing of all active profile files fetched during sync |

---

## Profile Schema

```jsonc
{
  // --- Core Identification & Metadata (Required) ---
  "domain": "example.com",                     // string, required — Base domain without protocol or www (e.g. "royalroad.com")
  "titleSelector": "h1.title",                 // string, required — CSS selector for novel title
  "authorSelector": ".author",                 // string, required — CSS selector for novel author (leading "By " stripped automatically)
  "chapterLinkSelector": "ol li a",           // string, required — CSS selector for chapter links in the Table of Contents
  "chapterContentSelector": ".chapter-text",   // string, required — CSS selector for chapter body text container

  // --- Optional Novel Metadata Selectors ---
  "coverImageSelector": ".cover img",          // string | null — CSS selector for cover image (reads src, abs:src, or meta content)
  "summarySelector": ".synopsis",              // string | null — CSS selector for novel description / synopsis
  "tagSelector": ".genre-tags a",              // string | null — CSS selector for tags and genres
  "ratingSelector": "meta[property='rating']", // string | null — CSS selector for novel score / rating (supports text, meta, data-*, ld+json)

  // --- Chapter Details & Ordering ---
  "chapterDateSelector": ".date",              // string | null — CSS selector for chapter release date (reads datetime attr or text)
  "topAuthorNotesSelector": ".note-top",       // string | null — CSS selector for author's notes appearing before chapter content
  "bottomAuthorNotesSelector": ".note-bottom", // string | null — CSS selector for author's notes appearing after chapter content
  "reverseChapterOrder": false,                // boolean — Set true if TOC displays newest chapters first to reverse them chronologically

  // --- URL Normalization ---
  "chapterUrlRegex": "https?://example\\.com/novel/(\\d+)/chapter/(\\d+)", // string | null — Regex with capture groups matching chapter URLs
  "novelUrlTemplate": "https://example.com/novel/{1}",                     // string | null — Template mapping regex capture groups to the novel landing URL

  // --- Content Cleaning & Hygiene ---
  "noiseSelectors": [                          // string[] — CSS selectors to strip from chapter body (ads, vote widgets, nav buttons, share links)
    ".advertisement",
    "#vote-box"
  ],
  "watermarkPhrases": [                        // string[] — Substrings to detect and remove copy-protection / anti-plagiarism watermark paragraphs
    "stolen from royal road",
    "copied without authorization"
  ],

  // --- Network, Rate Limiting & Auth ---
  "rateLimitMs": 3000,                         // number | null — Minimum interval (in ms) between HTTP requests to this domain (default: 1500ms)
  "loginUrlTemplate": "https://example.com/login?redirect={path}", // string | null — Auth URL template for paywalled content ({url} & {path} substituted)
  "preseedCookies": [                          // string[] — Cookie headers pre-loaded into CookieManager (e.g. adult age-gate bypass)
    "view_adult=true"
  ],
  "requiresWebView": false,                    // boolean — Force initial loading through WebView instead of OkHttp (e.g. aggressive Cloudflare)

  // --- Intake & Sync Behavior ---
  "supportsNewChapters": true,                 // boolean — Set false if site is a static/completed archive where polling for updates should be skipped
  "treatFirstChapterAsLandingPage": false,     // boolean — If true, importing a chapter 1 URL does not auto-advance lastRead to chapter 1
  "initials": "EX",                            // string | null — 2-3 letter badge abbreviation displayed on covers in the UI
  "version": 1                                 // number — Profile schema version (increment when updating selectors so remote sync updates the DB)

  // --- Dynamic Metadata Enrichment (AJAX) ---
  "dynamicDetails": {                          // object | null — Config for fetching novel metadata loaded via secondary AJAX/POST calls
    "triggerSelector": "#s-details",           // string | null — Only trigger AJAX call if this element exists on the landing page
    "endpointTemplate": "https://example.com/api/details", // string, required — Endpoint URL (supports {var} interpolation)
    "method": "POST",                          // string — HTTP method ("POST" or "GET", default: "POST")
    "jsVariables": ["story_id", "csrf_token"], // string[] — Variable names extracted from inline JS or page URL
    "formDataTemplate": {                      // object — Form fields for POST request, interpolating {var} values
      "id": "{story_id}",
      "token": "{csrf_token}"
    },
    "injectTargetSelector": "#s-details"       // string, required — Target DOM selector where returned HTML snippet is appended
  }
}
```

---

## Field Reference

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `domain` | `string` | Base domain name of the website (e.g. `"royalroad.com"`). Do not include protocols (`https://`) or `www.`. |
| `titleSelector` | `string` | CSS selector for the novel's title on its main page. If targeting a `<meta>` tag, its `content` attribute is extracted automatically. |
| `authorSelector` | `string` | CSS selector for the novel's author. Leading `"By "` / `"by "` prefixes are stripped automatically. |
| `chapterLinkSelector` | `string` | CSS selector targeting chapter links in the Table of Contents. Can target `<a>` tags or `<option value="...">` dropdown items. Prev/Next navigation links are ignored automatically. |
| `chapterContentSelector` | `string` | CSS selector for the chapter body container. |

### Metadata Selectors (Optional)

| Field | Type | Default | Description |
|-------|------|:-------:|-------------|
| `coverImageSelector` | `string` | `null` | CSS selector for novel cover art. Extracts `abs:src`, `src`, or `<meta content="...">`. |
| `summarySelector` | `string` | `null` | CSS selector for novel description / synopsis text. |
| `tagSelector` | `string` | `null` | CSS selector matching genre and tag links. Tags are automatically sanitized and deduplicated. |
| `ratingSelector` | `string` | `null` | CSS selector for the rating. Parses numeric values from text, `data-*` attributes, or `schema.org` JSON-LD metadata. |

### Chapter Ordering & Author Notes

| Field | Type | Default | Description |
|-------|------|:-------:|-------------|
| `chapterDateSelector` | `string` | `null` | CSS selector for chapter release date. Reads `datetime` attribute or inner text. |
| `topAuthorNotesSelector` | `string` | `null` | CSS selector for author note containers placed before the chapter body. |
| `bottomAuthorNotesSelector` | `string` | `null` | CSS selector for author note containers placed after the chapter body. |
| `reverseChapterOrder` | `boolean` | `false` | When `true`, reverses the parsed chapter list so the oldest chapter is indexed first. |

### URL Normalization

| Field | Type | Default | Description |
|-------|------|:-------:|-------------|
| `chapterUrlRegex` | `string` | `null` | Regex pattern matching individual chapter URLs. Numbered capture groups (`(\\d+)`, `([^/]+)`) are extracted. |
| `novelUrlTemplate` | `string` | `null` | Template URL where `{1}`, `{2}`, etc., are substituted by the regex match groups to reconstruct the novel's landing page URL when adding by chapter URL. |

### Content Sanitization & Cleaning

| Field | Type | Default | Description |
|-------|------|:-------:|-------------|
| `noiseSelectors` | `string[]` | `[]` | List of CSS selectors to remove from the chapter HTML before saving (e.g. advertisements, rating widgets, jump links, report buttons). |
| `watermarkPhrases` | `string[]` | `[]` | List of lowercase phrases to detect and remove anti-scraper copy-paste watermark blocks inserted within chapter text. |

### Network, Rate Limiting & Authentication

| Field | Type | Default | Description |
|-------|------|:-------:|-------------|
| `rateLimitMs` | `number` | `null` | Minimum delay in milliseconds between requests to this domain. If `null`, uses the global default of `1500` ms. Sites with strict firewalls (e.g. StoriesOnline) should set `3000` to `4500`. |
| `loginUrlTemplate` | `string` | `null` | URL template to redirect the user to when login/paywall detection triggers. Supports `{url}` (full target URL) and `{path}` (URL path) placeholders. |
| `preseedCookies` | `string[]` | `[]` | Cookies pre-loaded into the Android `CookieManager` before making requests (e.g. `["view_adult=true"]` for AO3 or consent cookies for Literotica). |
| `requiresWebView` | `boolean` | `false` | When `true`, delegates web requests through Android's WebView rather than OkHttp. |

### App Behavior & Sync Control

| Field | Type | Default | Description |
|-------|------|:-------:|-------------|
| `supportsNewChapters` | `boolean` | `true` | Set to `false` for static story repositories or complete archives to prevent background workers from running unnecessary update polling checks. |
| `treatFirstChapterAsLandingPage` | `boolean` | `false` | When `true`, importing a story via URL that points to chapter 1 does not automatically mark chapter 1 as the current reading bookmark. |
| `initials` | `string` | `null` | 2–3 letter uppercase badge shown on novel covers in the library (e.g. `"RR"`, `"AO"`, `"SH"`). |
| `version` | `number` | `1` | Schema version number. **Always increment this integer** when modifying selectors so that client devices update their cached DB profiles during background synchronization. |

### Dynamic Details Specification (`dynamicDetails`)

For websites where synopsis, tags, or metadata are loaded via asynchronous secondary AJAX requests (such as StoriesOnline):

| Key | Type | Description |
|-----|------|-------------|
| `triggerSelector` | `string?` | Optional selector; dynamic enrichment is skipped if this element is absent. |
| `endpointTemplate` | `string` | The API/endpoint URL to request. Supports `{variable}` substitution. |
| `method` | `string` | HTTP method: `"POST"` (default) or `"GET"`. |
| `jsVariables` | `string[]` | JavaScript variables extracted from inline `<script>` tags, HTML attributes, or the page URL. Special recognized names: `story_id`, `id`. |
| `formDataTemplate` | `map<string, string>` | Key-value pairs for POST body parameters, supporting `{variable}` interpolation. |
| `injectTargetSelector` | `string` | CSS selector of the DOM element into which the resulting HTML snippet is appended before standard selector extraction runs. |

---

## Contributing a Profile

1. **Create or Copy a Profile JSON**:
   Name the file `<sitename>.json` in the `profiles/` directory.
2. **Configure Domain & Selectors**:
   - Test your selectors in browser developer tools (`document.querySelector(...)` or `$$('...')`) on real novel landing and chapter pages.
   - Use comma-separated selectors if the site uses different layouts or themes across pages (e.g. `.fic_title, .fic-title`).
3. **Configure Cleanup & Limits**:
   - Add any floating ads or navigation junk to `noiseSelectors`.
   - Add anti-piracy watermark text to `watermarkPhrases`.
   - Set `rateLimitMs` if the site restricts crawling speed.
4. **Register in Manifest**:
   Add your new filename (e.g. `"mysite.json"`) to [`profiles/index.json`](index.json).
5. **Increment Version**:
   If updating an existing profile, increment its `"version"` integer so devices receive the update.
6. **Submit a Pull Request**:
   Commit your changes and open a PR against the main repository.

---

## License

MIT
