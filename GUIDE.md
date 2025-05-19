# stal/IX Documentation Style Guide

> This guide defines a temporary formatting standard for **stal/IX** documentation on GitHub.

<!-- {% raw %} -->

---

### TL;DR Markup conventions

Used in all documents:

**Headings** - are created by using one to six `#` symbols (depending on the required size) before the heading text. `# The document title` (h1) is written on the first line.

**Paragraphs and line breaks** - Leave a blank line between lines of text for paragraphs, use `<br>` at the end of the previous line for line breaks.

**Text formatting** - italics - `*asterisks*`; bold - double `**asterisks**`; combined highlighting - `**asterisks and _underscores_**`.

**Quotes** - created with `>` before quoting text. Used here to mark up a Prerequisites block. If there is no Prerequisites, quote the document description.<br> 
Exception: if there are `> indeed quotes`, mark up Prerequisites as a bulleted list, and the document description as plain text in the first paragraph.

**Code and syntax highlighting** - enclosed in single backticks for inline `code` - \`code\`; enclosed in triple backticks - \`\`\` for  code blocks.<br>
Use a language prefix next to the top line of triple backticks for code highlighting, no spaces - \`\`\`shell.<br>

Also used:

**Lists**: regular - each item on a new line, use `<br>` (e.g. to list links); bulleted - use asterisks `* some text`.

**Links**: You can create an inline link by enclosing the link text in brackets.<br>
`[stal/IX](https://github.com/stal-ix/ix)` [stal/IX](https://github.com/stal-ix/ix)

**Horizontal dividing line** - formed by three (or more) hyphens `---`. Use it to highlight special text blocks, for example: *Exercise*. In the text of a document, a block is highlighted by two dividing lines, at the top and bottom, if the block is at the end of the document - by one dividing line at the top.

---

## stal/IX documentation

The documentation is under development and is regularly updated.

All **stal/IX** documentation is formatted using [Markdown](https://en.wikipedia.org/wiki/Markdown). [HTML](https://en.wikipedia.org/wiki/HTML) is acceptable if there is no Markdown equivalent or special formatting is required.

**stal/IX** is written as is, with a lowercase letter, even in a title or at the beginning of a sentence, **IX** is written in capital letters if **IX** package manager is meant. **stal/IX** is written in bold.

## General documentation markup

A document always begins with a title. The title is placed at the top, on the first line. There are no buttons, shields, banners, quotes or other design or formatting elements above it.

Place under the title:

* Shields, buttons and other elements if necessary;
* Prerequisites;
* Document description. 

The sections of the document are designated by [headings](GUIDE.md#headings).<br>
Special text blocks - with [horizontal rule](GUIDE.md#horizontal-rule).

## Headings

Headings are created by using one to six `#` symbols (depending on required size) before the heading text.

```Markdown
# h1 - the largest heading
## h2 
### h3
#### h4
##### h5
###### h6 - the smallest heading
```

# h1 - the largest heading
## h2 
### h3
#### h4
##### h5
###### h6 - the smallest heading

The `# h1` heading marks the document title on the first line. There can only be one h1 heading per document.<br>
The second-level `# h2` heading marks the document sections, if any. Then, use the other heading levels as needed.

## Paragraphs and line breaks

Paragraph - leave a blank line between lines of text. Line breaks - use `<br>` at the end of the previous line.

*Important:* Markdown Here's line break rules (two spaces at the end of a line) are not used to avoid removing trailing spaces when the editor is set to do so.

## Text styling

Bold and italic are used:<br> 
* italic - `*asterisks*`; 
* bold type - double `**asterisks**`; 
* combined highlighting - `**asterisks and _underscores_**`.

Required text highlightings:<br>
* *Pro tip* - `*Pro tip*` italic
* *Warning* - `*Warning*` italic
* *Important* - `*Important*` italic

## Quoting text

Before quoting text, use the `>` symbol. 

```Markdown
> Quoting text
```
> Quoting text

Here we mark the Prerequisites block as a quote. If the document has no prerequisites, quote the document description.<br>
Exception: if the top of the document text (the first two paragraphs) indeed has quotes, then mark the Prerequisites as a bulleted list, the document description as plain text in the first paragraph.

## Code quoting and syntax highlighting

To embed `code` or `command output` in a sentence, use single backticks - \`code\` or \`command output\`.
To format code or scripts in a separate block, use triple backticks.<br>
Use a language prefix next to the top line of triple backticks, with no spaces, to highlight code - \`\`\`shell.

\`\`\`Markdown

code block

\`\`\`

```Markdown
code block
```

## Lists

**Unordered lists:**

* regular - each item on a new line, use line breaks `<br>` (for example, to list links);
* bulleted - markup with asterisks separated by spaces.<br>
<!-- Does not render properly -->
```Markdown
* point a; 
* point b.
```

**Ordered (numbered) lists:**

Due to the fact that the documentation is still under development and is regularly updated, we do not use numbered lists yet.

## Links

### Absolute links

You can create an inline link by enclosing the link text in square brackets `[ ]` and the URL in parentheses `( )`, without spaces.

```Markdown
[IX package manager](https://github.com/stal-ix/ix)
```

[IX package manager](https://github.com/stal-ix/ix)

### Relative links

The link text is enclosed in square brackets, the link itself is enclosed in parentheses, without spaces.

```Markdown
[STALIX.md](STALIX.md)
```

[STALIX.md](STALIX.md)

A relative link is a link that is relative to the current file. Define relative links to navigate to other files in the same repository, as absolute links may not work in clones of a repository.

## Horizontal rule

To create a horizontal dividing line, use three (or more) hyphens `---`. The horizontal line is used to separate special blocks of text (e.g. Exercise).

```Markdown
---
Exercise<br>
special text block

---
```

---
Exercise<br>
special text block

---

In the text of the document, the block is separated by two dividing lines, at the top and bottom, if the block is at the end of the document (for example, comments) - by one dividing line at the top. There must be a blank line between the last line of the block text and the bottom dividing line.

## Ignoring Markdown formatting

To ignore Markdown formatting, use `\` before a Markdown character.

<!-- {% endraw %} -->
