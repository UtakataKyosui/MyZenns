# AGENTS.md: Guidelines for Agentic Coding in Zenn Repository

This repository contains Zenn articles and books in Markdown format, with supporting DevContainer and GitHub workflows. No traditional code (e.g., Rust/TS) is present; focus on Markdown content creation, linting, and Zenn CLI workflows.

## Build/Preview Commands
- Preview content: `zenn preview` - Serves local site at http://localhost:3000 for articles/books.
- New article: `zenn new:article` - Creates new Markdown article in /articles/.
- New book chapter: `zenn new:book` - Adds to /books/ structure.
- List content: `zenn list:articles` or `zenn list:books` - Views available files.
- No build step; content is static Markdown. Use DevContainer for isolated editing: `devcontainer up`.

## Lint/Test Commands
- Markdown linting: Install markdownlint-cli globally (`npm i -g markdownlint-cli`), then `markdownlint "**/*.md"` from root.
- Single file lint: `markdownlint articles/claude-code-devcontainer-safety.md`.
- No unit tests; validate via `zenn preview` and manual review. For workflows, test GitHub Actions locally with `act`.
- Single 'test' (preview one file): Open in VS Code/Obsidian or use `zenn preview` (no single-file mode; previews all).
- Pre-commit: Run `markdownlint` before commits; check .github/workflows for automated reviews.

## Code Style Guidelines (Markdown-Focused)
- **Formatting**: Use consistent indentation (2 spaces). Headings: # H1, ## H2, etc. Lists: - or 1. for ordered.
- **Imports/Links**: Internal: [Link text](path/to/file.md). External: Full URLs. No code imports; embed code blocks with ```language.
- **Types**: N/A for Markdown. Use YAML frontmatter in books (e.g., config.yaml) for metadata: title, published: true.
- **Naming Conventions**: Files: kebab-case (e.g., claude-code-devcontainer-safety.md). Sections: Descriptive, title-case headings.
- **Error Handling**: In content, explain errors clearly with code snippets. For workflows, handle in YAML (e.g., if: failure()).
- **General**: Keep lines <80 chars. Use semantic line breaks. Emojis sparingly. Follow Zenn guidelines: https://zenn.dev/zenn/articles/zenn-cli-guide.
- **Obsidian Plugins**: If editing in Obsidian, respect .obsidian/ configs; avoid modifying plugin files unless updating.
- **Git Practices**: Commit Markdown changes only; use descriptive messages. No sensitive data in articles.

For any code snippets in Markdown (e.g., Rust/TS examples), follow language-specific styles: Rust - cargo fmt; TS - Prettier/ESLint if tools present (none here).

(18 lines)