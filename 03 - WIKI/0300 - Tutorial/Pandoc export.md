---
type:
  - "[[note]]"
date: 2025-09-29
timestamp: 2025-09-29 09:52:21

tags:
  - xxx
  - yyy
  - zzz
---
# Learning from Pandoc YAML Errors to EPUB Metadata

## Key Points

- **YAML Front Matter Format Matters:**  
  Correct YAML syntax is crucial. Always use `---` to start and end metadata blocks, with proper indentation and ASCII spaces. Malformed YAML causes Pandoc to fail parsing.

- **Horizontal Rules (`---`) Need Surrounding Blank Lines:**  
  To prevent Pandoc from mistaking horizontal rules for YAML delimiters, make sure there is a blank line before and after every `---` used as a divider inside the document.

- **EPUB Output Requires a Title:**  
  EPUB conversion needs a non-empty `<title>` element. This can be set in the YAML front matter via `title: "Document Title"` or passed via command-line using `--metadata title="Document Title"`.

- **Pandoc Extensions and Flags Matter:**  
  If using extended Markdown features (like Obsidian wikilinks), use appropriate Pandoc flags (e.g., `+wikilinks_title_after_pipe`) to ensure correct parsing.

- **Command Syntax Must Be Correct:**  
  Use the right Pandoc flags (`-f` for format, `-o` for output) and ensure resource paths are correctly set to avoid silent errors or failures.

- **Update Pandoc Regularly:**  
  Bugs with parsing YAML or Markdown extensions are fixed regularly. Use the latest Pandoc release for best compatibility.

## Troubleshooting Strategy

1. Check and fix YAML front matter syntax.
2. Add blank lines before and after each `---` that is a horizontal rule.
3. Add a `title` to metadata in YAML or command.
4. Test with minimal Markdown files.
5. Upgrade Pandoc if errors persist.

## Summary Table

| Issue                             | Cause                              | Fix / Best Practice                      |
|----------------------------------|----------------------------------|-----------------------------------------|
| YAML parse exception             | Malformed YAML or bad `---` usage | Fix YAML syntax, add blank lines around `---` |
| Horizontal rule mistaken for YAML | No blank lines around `---`       | Add blank line before and after every `---`  |
| Missing EPUB title warning       | No title in metadata              | Add `title` in YAML or via `--metadata` flag  |
| Wikilink parsing issues          | Missing appropriate extension flag | Use `+wikilinks_title_after_pipe` or similar   |
| Command line errors              | Typos or wrong flags              | Use correct Pandoc syntax and flags          |
| Old Pandoc version bugs          | Using outdated version            | Update to latest Pandoc release                |

---

By following these guidelines, you can reliably convert your Obsidian Markdown notes to professional EPUBs using Pandoc without format or metadata errors.

try using *** not --- for line breaks
