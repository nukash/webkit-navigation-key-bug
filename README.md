# WebKit Navigation API – `NavigationHistoryEntry.key` stability bug

Minimal reproduction for a WebKit/Safari bug where `NavigationHistoryEntry.key` is not preserved when traversing back to a previously visited history entry in an SPA.

## Live demo

👉 **https://YOUR_USERNAME.github.io/webkit-navigation-key-bug/**

Open on **iOS Safari 26.x** to reproduce.

## The bug

In a single-page application, the Navigation API's `key` property should be a **stable identifier for a history entry's slot**. When the user navigates back to a previously visited page, the same `key` should be returned.

### Steps to reproduce

1. **Page A** → record `navigation.currentEntry.key` (= `keyA`)
2. **Page B** → push navigation
3. **← Back** → Page A → key should equal `keyA` ✓
4. **Page C** → push navigation
5. **← Back** → Page A → **key should equal `keyA`** ✗ (new UUID assigned)

### Expected

`key` remains the same across all visits to the same history slot.

### Actual (iOS Safari 26.3.1)

At step 5, Page A receives a **new key**, different from the original. Once this occurs, every subsequent traversal back to Page A continues to produce new keys.

This breaks `navigation.traverseTo(savedKey)`, which is the primary use case for `key`.

## How it works

- **Zero framework dependencies** – uses only `navigation.navigate()` + `event.intercept()` for SPA behavior
- **Auto-detection** – compares every Page A key to the first recorded one and flags mismatches
- **Key tracker table** – shows all Page A visits with pass/fail status
- **Full event log** – logs `navigation.entries()` at every step for debugging

## Deploying to GitHub Pages

1. Fork or clone this repo
2. Go to **Settings → Pages**
3. Set source to **Deploy from a branch** → `main` / `/ (root)`
4. Access at `https://YOUR_USERNAME.github.io/webkit-navigation-key-bug/`

The `404.html` is a copy of `index.html`. This is the standard trick for SPA routing on GitHub Pages – any path like `/page-a` returns the app instead of a 404 error.

## Related WebKit bugs

| Bug | Title | Date |
|-----|-------|------|
| [278253](https://bugs.webkit.org/show_bug.cgi?id=278253) | Fix unstable UUIDs in failing WPT tests | 2024-10 |
| [282627](https://bugs.webkit.org/show_bug.cgi?id=282627) | Entry state can be lost during traversals | 2024-12 |
| [278294](https://bugs.webkit.org/show_bug.cgi?id=278294) | Fix navigation-history-entry/current-basic.html | 2024-08 |
| [284663](https://bugs.webkit.org/show_bug.cgi?id=284663) | NavigationHistoryEntry JS wrapper not kept alive until dispose | 2024-12 |
| [306050](https://bugs.webkit.org/show_bug.cgi?id=306050) / CVE-2026-20643 | Cross-origin issue in Navigation API | 2026-03 |

## Specification references

- [HTML Spec – NavigationHistoryEntry key](https://html.spec.whatwg.org/multipage/nav-history-apis.html#concept-navigationhistoryentry-key)
- [Navigation API explainer (WICG)](https://github.com/WICG/navigation-api)
- [MDN – NavigationHistoryEntry](https://developer.mozilla.org/en-US/docs/Web/API/NavigationHistoryEntry)
