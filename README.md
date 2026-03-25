# check-indexed-pages

A Claude Code skill that checks how many pages from any domain are indexed by Google.

Uses browser automation to navigate `site:` search results to the last page and counts the exact total — more accurate than Google's "About X results" estimate.

## Installation

```bash
git clone https://github.com/zachariah-mithani/check-indexed-pages.git ~/.claude/skills/check-indexed-pages
```

## Usage

```
/check-indexed-pages example.com
```

Optionally log the result to a markdown file:

```
/check-indexed-pages example.com --log ~/projects/mysite/progress.md
```

The `--log` flag appends a timestamped line to the first `## Indexing Status` section found in the file, or to the end if no such section exists.

## Requirements

- [Claude Code](https://claude.ai/code) with the `claude-in-chrome` MCP extension installed
- Chrome browser open

## How it works

1. Opens a new browser tab
2. Navigates to `https://www.google.com/search?q=site:<domain>`
3. Finds the highest page number in the result navigation
4. Jumps to that page and checks for a "Next" button
5. Keeps paginating until the last page
6. Counts results on the final page
7. Calculates total: `(last_page - 1) × 10 + results_on_last_page`

## License

MIT
