# Markdown Formatting Cheat Sheet

---

## Table of Contents

- [Headings](#headings)
- [Text Formatting](#text-formatting)
- [Paragraphs & Line Breaks](#paragraphs--line-breaks)
- [Lists](#lists)
- [Links](#links)
- [Images](#images)
- [Blockquotes](#blockquotes)
- [Code](#code)
- [Tables](#tables)
- [Horizontal Rules](#horizontal-rules)
- [Task Lists](#task-lists)
- [Footnotes](#footnotes)
- [Escaping Characters](#escaping-characters)
- [HTML in Markdown](#html-in-markdown)
- [Extended Syntax (GitHub Flavored)](#extended-syntax-github-flavored)

---

## Headings

Headings are created with the `#` symbol. The number of `#` symbols determines the level — think of them like H1 through H6 in HTML.

```markdown
# Heading 1       → Largest, top-level title
## Heading 2      → Section title
### Heading 3     → Subsection
#### Heading 4    → Sub-subsection
##### Heading 5   → Rarely used
###### Heading 6  → Smallest heading
```

**Tip:** Always leave a space between the `#` and the text. `#Title` won't render as a heading in strict parsers.

You can also underline H1 and H2 with `===` or `---` (Setext style), though the `#` style is more portable:

```markdown
Heading 1
=========

Heading 2
---------
```

---

## Text Formatting

These inline styles can be combined and nested within paragraphs, list items, table cells, and so on.

```markdown
**bold text**              → Bold (strong emphasis)
*italic text*              → Italic (emphasis)
_italic text_              → Also italic (same result)
__bold text__              → Also bold
***bold and italic***      → Both at once
~~strikethrough~~          → Strikethrough (GFM)
`inline code`              → Monospace code span
<mark>highlighted</mark>   → Highlighted (HTML, not universal)
```

**Rendered examples:**

| Syntax | Result |
|--------|--------|
| `**bold**` | **bold** |
| `*italic*` | *italic* |
| `***both***` | ***both*** |
| `` `code` `` | `code` |
| `~~struck~~` | ~~struck~~ |

---

## Paragraphs & Line Breaks

This is one of the trickier parts of Markdown because the behavior is not always intuitive.

A **paragraph** is simply one or more lines of text separated by a **blank line**. Without the blank line, the two lines merge into one paragraph:

```markdown
This is the first paragraph.

This is the second paragraph — note the blank line above.

This line and
this line will merge into one paragraph (no blank line between them).
```

To force a **line break** within a paragraph (without starting a new paragraph), end the line with **two spaces** before pressing Enter, or use a `<br>` HTML tag:

```markdown
Line one with two trailing spaces  
Line two appears on its own line.

Or use HTML: Line one<br>Line two.
```

---

## Lists

### Unordered Lists

Use `-`, `*`, or `+` as bullet markers. Mixing them in the same list is allowed but inconsistent style is discouraged.

```markdown
- Item one
- Item two
  - Nested item (indent with 2 spaces)
  - Another nested item
    - Deeper nesting (4 spaces)
- Item three
```

### Ordered Lists

Numbers followed by a period. Interestingly, the actual numbers don't matter — Markdown will auto-number them correctly. It's good practice to number them correctly anyway for readability in the source.

```markdown
1. First item
2. Second item
   1. Nested ordered item
   2. Another nested item
3. Third item
```

### Mixed Lists

```markdown
1. First ordered item
2. Second ordered item
   - Unordered sub-item
   - Another sub-item
3. Third ordered item
```

**Tip:** To add a paragraph inside a list item, indent the paragraph content by the same amount as the list content (usually 4 spaces or a tab):

```markdown
- This is a list item.

    This paragraph belongs to the same list item because it is indented.

- This is a new list item.
```

---

## Links

```markdown
[Link text](https://example.com)                        → Basic link
[Link with title](https://example.com "Hover tooltip")  → With hover title
<https://example.com>                                   → Auto-link (bare URL)
<email@example.com>                                     → Auto-link email

[Reference link][ref-id]                                → Reference style
[ref-id]: https://example.com "Optional title"          → Define reference anywhere
```

**Tip:** Reference-style links are great for readability when the same URL is used many times or when you want to keep the prose clean.

---

## Images

Images follow the same syntax as links, but with a `!` prefix. The alt text inside `[]` is important for accessibility and is shown when the image fails to load.

```markdown
![Alt text](image.png)
![Alt text](https://example.com/image.png "Optional title")

<!-- Reference style: -->
![Alt text][img-ref]
[img-ref]: image.png "Optional title"
```

To control image size (not standard Markdown — requires HTML):

```html
<img src="image.png" alt="Alt text" width="300">
```

---

## Blockquotes

Use `>` to create a blockquote. Blockquotes can be nested and can contain other Markdown elements inside them.

```markdown
> This is a blockquote.

> This blockquote has
> multiple lines.

> First level
>> Second level (nested)
>>> Third level

> **Markdown works inside blockquotes.**
> You can have lists, code, etc.
```

Blockquotes render like this:

> First level
>> Second level (nested)

---

## Code

Code comes in two flavors: **inline** (for short snippets within a sentence) and **fenced blocks** (for multi-line code samples).

### Inline Code

Wrap with single backticks:

```markdown
Use the `print()` function to output text.
```

If your code contains a backtick, wrap it in double backticks:

```markdown
`` Use `backticks` inside inline code ``
```

### Fenced Code Blocks

Use triple backticks (` ``` `) or triple tildes (`~~~`) to open and close a block. Specifying a language after the opening fence enables **syntax highlighting**:

````markdown
```python
def greet(name):
    # This comment is highlighted as Python
    return f"Hello, {name}!"
```

```bash
sudo apt update && sudo apt upgrade -y
```

```json
{
  "name": "Alice",
  "role": "admin"
}
```
````

**Common language identifiers:** `python`, `javascript`, `js`, `bash`, `sh`, `html`, `css`, `json`, `yaml`, `sql`, `markdown`, `md`, `c`, `cpp`, `java`, `go`, `rust`, `typescript`, `ts`, `diff`.

You can also indent code by 4 spaces (older Markdown style), but fenced blocks are preferred:

```markdown
    This is a code block
    indented by 4 spaces
```

---

## Tables

Tables are a GitHub Flavored Markdown (GFM) extension — not part of the original spec, but supported almost everywhere.

```markdown
| Column 1   | Column 2   | Column 3   |
|------------|------------|------------|
| Cell A1    | Cell B1    | Cell C1    |
| Cell A2    | Cell B2    | Cell C2    |
```

### Column Alignment

The colons in the separator row control alignment:

```markdown
| Left       | Center     | Right      |
|:-----------|:----------:|----------:|
| Left text  | Centered   | Right text |
| More left  | More mid   | More right |
```

**Tip:** You don't need to align the pipes perfectly in the source — it's just good style. The table still renders correctly even if spacing is uneven.

---

## Horizontal Rules

Any of these produce a horizontal dividing line. Use them to separate major sections:

```markdown
---
***
___
```

**Tip:** Leave a blank line before and after a horizontal rule to avoid accidental interpretation as an H2 heading (Setext style).

---

## Task Lists

A GitHub Flavored Markdown feature that renders interactive checkboxes (clickable on GitHub and many other platforms):

```markdown
- [x] Task already completed
- [ ] Task still to do
- [ ] Another pending task
  - [x] Nested completed sub-task
  - [ ] Nested pending sub-task
```

---

## Footnotes

Supported in many renderers (GitHub, Obsidian, Pandoc) but not all:

```markdown
Here is a sentence with a footnote.[^1]
And another one.[^note]

[^1]: This is the first footnote.
[^note]: Footnotes can use labels instead of numbers.
```

The footnote definitions can be placed anywhere in the document — they always render at the bottom.

---

## Escaping Characters

To display a character that Markdown would normally interpret as syntax, prefix it with a backslash `\`:

```markdown
\*This is not italic\*
\# This is not a heading
\[This is not a link\]
\`This is not code\`
\\ This is a literal backslash
```

Characters you can escape: `\ * _ {} [] () # + - . ! |`

---

## HTML in Markdown

Raw HTML is supported in most Markdown parsers. This is useful when you need features Markdown doesn't provide:

```html
<details>
  <summary>Click to expand</summary>
  Hidden content revealed on click.
</details>

<kbd>Ctrl</kbd> + <kbd>C</kbd>   → Keyboard key styling

<sub>subscript</sub>             → H₂O style
<sup>superscript</sup>           → E=mc² style

<br>                             → Forced line break
```

---

## Extended Syntax (GitHub Flavored)

These features are part of **GFM (GitHub Flavored Markdown)** and are widely supported but not in the original spec.

### Syntax Highlighting in Diffs

````markdown
```diff
- This line was removed
+ This line was added
  This line is unchanged
```
````

### Emoji

```markdown
:tada:   → 🎉
:rocket: → 🚀
:white_check_mark: → ✅
```

Full list: [emoji-cheat-sheet.com](https://www.webfx.com/tools/emoji-cheat-sheet/)

### Alerts / Callouts (GitHub only)

````markdown
> [!NOTE]
> Useful information.

> [!TIP]
> A helpful suggestion.

> [!WARNING]
> Something the reader should be cautious about.

> [!IMPORTANT]
> Critical information.
````

### Definition Lists (Pandoc / some renderers)

```markdown
Term
:   Definition of the term.

Another Term
:   Its definition.
```

---

## Quick Reference Card

| Element | Syntax |
|---|---|
| H1 | `# Title` |
| H2 | `## Section` |
| Bold | `**text**` |
| Italic | `*text*` |
| Bold+Italic | `***text***` |
| Strikethrough | `~~text~~` |
| Inline code | `` `code` `` |
| Link | `[text](url)` |
| Image | `![alt](url)` |
| Blockquote | `> text` |
| Unordered list | `- item` |
| Ordered list | `1. item` |
| Task list | `- [x] done` |
| Horizontal rule | `---` |
| Code block | ` ```lang ` |
| Table | `\| col \| col \|` |
| Footnote | `[^1]` |
| Escape | `\*literal\*` |

---
