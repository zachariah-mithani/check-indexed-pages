---
name: check-indexed-pages
version: 1.1.0
description: |
  Check how many pages from any domain are currently indexed by Google.
  Uses browser automation to run a site: search, navigates to the last page
  of results, and counts the exact total. More accurate than Google's
  "About X results" estimate.
  Usage: /check-indexed-pages <domain> [--log <path/to/file.md>]
allowed-tools:
  - mcp__claude-in-chrome__tabs_context_mcp
  - mcp__claude-in-chrome__tabs_create_mcp
  - mcp__claude-in-chrome__navigate
  - mcp__claude-in-chrome__javascript_tool
  - Read
  - Edit
---

# Check Indexed Pages

Check how many pages Google has indexed for any domain by finding the last page of `site:` search results.

## Usage

```
/check-indexed-pages example.com
/check-indexed-pages example.com --log ~/projects/mysite/progress.md
```

- `<domain>` — required. The domain to check (e.g. `aplosai.com`, `example.com`)
- `--log <path>` — optional. Absolute path to a markdown file. If provided, appends a timestamped line to the first "Indexing Status" section found, or to the end of the file if no such section exists.

## Steps

1. **Load browser tools** — use ToolSearch to load each `mcp__claude-in-chrome__` tool before calling it

2. **Get tab context** — call `tabs_context_mcp`

3. **Create a new tab** — always create a fresh tab via `tabs_create_mcp`

4. **Navigate to the site: search**
   ```
   https://www.google.com/search?q=site:<domain>
   ```

5. **Find the last page of results**

   Get the highest page number from the navigation:
   ```js
   Math.max(...[...document.querySelectorAll('.AaVjTc a, #nav a')]
     .map(a => parseInt(a.innerText.trim()))
     .filter(n => !isNaN(n)))
   ```

   Navigate to that page:
   ```
   https://www.google.com/search?q=site:<domain>&start=<(highest_page - 1) * 10>
   ```

   Check for a Next button:
   ```js
   !!document.querySelector('#pnnext')
   ```

   If Next exists, keep incrementing by 10 and rechecking until no Next button.

   On the final page, count results:
   ```js
   document.querySelectorAll('.tF2Cxc').length
   ```

6. **Calculate total**
   ```
   total = (last_page_number - 1) × 10 + results_on_last_page
   ```

7. **If `--log <path>` was passed**
   - Read the file at `<path>`
   - Find the `## Indexing Status` section (or nearest equivalent heading)
   - Append a new line in this format:
     ```
     - YYYY-MM-DD: X indexed (confirmed via site:<domain> search)
     ```
   - Use today's date from context

8. **Report the result** — state the exact count clearly

## Notes

- Google's "About X results" stat is unreliable — always count from the last page
- The `site:` operator may include some non-canonical or duplicate URLs, so the real count may be slightly lower than what GSC reports
- If Google shows a CAPTCHA or blocks the search, report that and suggest checking GSC directly
- If the domain has fewer than 10 indexed pages, the result is on page 1 — count `.tF2Cxc` elements directly
