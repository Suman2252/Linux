# 🤝 Contributing to The Ultimate Linux Guide

Thank you for your interest in contributing! This guide is a community effort and we welcome contributions of all kinds.

## How to Contribute

### 📝 Content Contributions

1. **Fix errors** — Found a typo, incorrect command, or outdated information? Open a PR!
2. **Add examples** — More real-world examples make the guide better for everyone.
3. **Expand topics** — Feel a section could cover more? Add to it.
4. **New chapters** — Have a topic not covered? Propose it via an issue first.

### 🐛 Reporting Issues

- Use GitHub Issues to report errors or suggest improvements
- Include the chapter number and section in your issue title
- Provide the correct information when reporting errors

### 🔀 Pull Request Process

1. **Fork** this repository
2. **Create a branch** for your changes: `git checkout -b fix/chapter-03-typo`
3. **Make your changes** following the style guide below
4. **Test** your Markdown renders correctly
5. **Submit** a pull request with a clear description

## 📏 Style Guide

### File Structure
- Each chapter lives in its own numbered directory (e.g., `01-introduction/`)
- Each directory contains a `README.md` as the main content file

### Markdown Conventions
- Use `#` for chapter title, `##` for major sections, `###` for subsections
- Use fenced code blocks with language hints: ` ```bash `
- Use `>` blockquotes for tips and warnings
- Include navigation links at bottom: `← Previous | Home | Next →`

### Command Examples
```bash
# ✅ Good — includes explanation comment
ls -la /etc    # List all files in /etc with details

# ❌ Bad — no context
ls -la /etc
```

### Content Tone
- Write in a **friendly, approachable** tone
- Use **real-world analogies** to explain complex concepts
- Assume the reader is intelligent but may be new to the topic
- Bold important terms when first introduced

## 📋 Checklist Before Submitting

- [ ] Markdown renders correctly
- [ ] Code examples are tested and working
- [ ] Navigation links are correct
- [ ] No broken links
- [ ] Consistent formatting with existing chapters

## 💬 Questions?

Open an issue with the `question` label and we'll get back to you!

---

Thank you for helping make Linux accessible to everyone! 🐧
