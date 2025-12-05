# DevTools analysis: Emily Lubonty

**Student:** emilylubonty (Emily Lubonty)
**Project:** "Nighttime Routine"
**Generated:** December 5, 2025 at 10:35 AM ET

---

## About this analysis

This report was generated using the **Chrome DevTools MCP Server**, a Model Context Protocol (MCP) integration that allows AI assistants to interact directly with Chrome's Developer Tools. Rather than manually inspecting pages, the MCP server provides programmatic access to the same powerful debugging features developers use in the browser.

### Tools used in this analysis

#### 1. Performance tracing (`performance_start_trace`)

This tool captures a performance trace similar to what you would see in Chrome's Performance panel or a Lighthouse audit. By setting `reload: true` and `autoStop: true`, the tool:

- Reloads the page to capture a fresh page load
- Records all browser activity during the load
- Automatically stops when the page finishes loading
- Returns Core Web Vitals metrics (LCP, CLS) and identifies performance insights

The trace provides insight into render-blocking resources, layout shifts, and the breakdown of time spent loading the Largest Contentful Paint element.

#### 2. Network request monitoring (`list_network_requests`)

This tool retrieves all network requests made by the page, equivalent to the Network panel in DevTools. For each request, it shows:

- The resource URL and file type
- HTTP status codes (200 for success, errors like `ERR_FILE_NOT_FOUND`)
- Whether requests succeeded or failed

This is particularly useful for verifying that fonts are self-hosted (loaded from local `fonts/` folder) rather than fetched from external CDNs like Google Fonts.

#### 3. Console message capture (`list_console_messages`)

This tool captures everything logged to the browser's Console panel, including:

- **Errors:** JavaScript exceptions, failed resource loads, CORS violations
- **Warnings:** Performance hints, deprecation notices, unused preloads
- **Info/Log:** Debug output from scripts

Console messages reveal issues that may not be visually apparent but indicate problems with the code, such as missing files or misconfigured resource paths.

### Why use DevTools MCP?

Traditional code review catches issues in source files, but DevTools analysis reveals **runtime behavior**—what actually happens when a browser loads the page. This includes:

- Resources that fail to load due to incorrect paths or CORS policies
- Performance bottlenecks not visible in static code
- Console errors that indicate broken functionality
- Verification that assets load from expected locations (self-hosted vs. CDN)

---

## Performance summary

| Metric | Value | Rating |
|--------|-------|--------|
| LCP (Largest Contentful Paint) | 93 ms | Excellent |
| CLS (Cumulative Layout Shift) | 0.00 | Excellent |

---

## Network requests analysis

### Successful requests

| Resource | Status | Type |
|----------|--------|------|
| `index.html` | 200 | Document |
| `style.css` | 200 | Stylesheet |
| `Card game-amico-hsl.svg` | 200 | Image |
| `fonts/DeliusSwashCaps-Regular.woff2` | 200 | Font |
| `fonts/FuzzyBubbles-Bold.woff2` | 200 | Font |
| `fonts/FuzzyBubbles-Regular.woff2` | 200 | Font |

### Failed requests

| Resource | Error | Issue |
|----------|-------|-------|
| `css/styles.css` | ERR_FILE_NOT_FOUND | Referenced in HTML but file doesn't exist |
| `fonts/delius-swash-caps.woff2` | ERR_FAILED/CORS | Lowercase filename doesn't match actual file |
| `fonts/fuzzy-bubbles-regular.woff2` | ERR_FAILED/CORS | Lowercase filename doesn't match actual file |
| `fonts/fuzzy-bubbles-bold.woff2` | ERR_FAILED/CORS | Lowercase filename doesn't match actual file |

### Font hosting verification

**Result: Fonts ARE self-hosted**

- No external CDN requests (Google Fonts, Adobe Fonts, etc.)
- All font files located in local `fonts/` directory
- Font files present: `DeliusSwashCaps-Regular.woff2`, `FuzzyBubbles-Bold.woff2`, `FuzzyBubbles-Regular.woff2`

---

## Console errors and warnings

### Errors (8 total)

| Type | Description |
|------|-------------|
| CORS Error | Font `delius-swash-caps.woff2` blocked by CORS policy |
| CORS Error | Font `fuzzy-bubbles-regular.woff2` blocked by CORS policy |
| CORS Error | Font `fuzzy-bubbles-bold.woff2` blocked by CORS policy |
| Load Error | Failed to load resource: `net::ERR_FAILED` (3 occurrences) |
| File Not Found | `css/styles.css` - `net::ERR_FILE_NOT_FOUND` |
| Issue | Verify stylesheet URLs (1 count) |

### Warnings (3 total)

| Type | Description |
|------|-------------|
| Preload Warning | `fuzzy-bubbles-bold.woff2` preloaded but not used |
| Preload Warning | `delius-swash-caps.woff2` preloaded but not used |
| Preload Warning | `fuzzy-bubbles-regular.woff2` preloaded but not used |

---

## Root cause analysis

### Issue 1: Duplicate @font-face declarations with mismatched filenames

The CSS contains two sets of @font-face rules:

1. **Lowercase filenames** (incorrect - files don't exist):
   - `fonts/delius-swash-caps.woff2`
   - `fonts/fuzzy-bubbles-regular.woff2`
   - `fonts/fuzzy-bubbles-bold.woff2`

2. **Proper case filenames** (correct - files exist):
   - `fonts/DeliusSwashCaps-Regular.woff2`
   - `fonts/FuzzyBubbles-Regular.woff2`
   - `fonts/FuzzyBubbles-Bold.woff2`

**Impact:** The correct fonts load successfully, so the page displays properly. However, the duplicate declarations cause unnecessary errors and preload warnings.

### Issue 2: Missing stylesheet reference

The HTML references `css/styles.css` which doesn't exist in the project. This may be leftover from a template or previous iteration.

---

## Recommendations

### High priority

1. **Remove duplicate @font-face declarations** - Keep only the working declarations with correct filenames
2. **Remove or fix `css/styles.css` reference** - Either remove the `<link>` tag from HTML or create the file if needed

### Low priority

3. **Clean up font preload tags** - Remove preload links for the incorrect lowercase filenames

---

## Summary

| Category | Status |
|----------|--------|
| Self-hosted fonts | ✅ Verified |
| Font files loading | ✅ Working (with fallback) |
| External CDN usage | ✅ None detected |
| Console errors | ⚠️ 8 errors (non-breaking) |
| Console warnings | ⚠️ 3 warnings |
| Missing files | ⚠️ `css/styles.css` not found |

**Overall:** The project functions correctly despite the errors. The fonts are properly self-hosted and loading. The issues are cleanup items that don't affect the visual presentation.
