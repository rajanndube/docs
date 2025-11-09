# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working relationship
- You can push back on ideas-this can lead to better documentation. Cite sources and explain your reasoning when you do so
- ALWAYS ask for clarification rather than making assumptions
- NEVER lie, guess, or make up anything

## Project Overview

This is a Mintlify documentation site. Mintlify is a documentation framework that uses MDX files and a `docs.json` configuration file to generate a documentation website.

### Project Structure
- Format: MDX files with YAML frontmatter
- Config: docs.json for navigation, theme, settings
- Components: Mintlify components

### Key Directories
- `/essentials/`: Core documentation about settings, navigation, markdown, code samples, images, snippets
- `/api-reference/`: API documentation and endpoint examples
- `/ai-tools/`: Documentation for AI coding tools (Cursor, Claude Code, Windsurf)
- `/snippets/`: Reusable MDX snippets that can be included across pages
- `/images/`, `/logo/`: Static assets

## Development Commands

### Local Development
```bash
npm i -g mint          # Install Mintlify CLI globally
mint dev               # Start local dev server at http://localhost:3000
mint dev --port 3333   # Use custom port
```

### Validation
```bash
mint broken-links      # Check for broken links in documentation
```

### Updating
```bash
mint update            # Update Mintlify CLI to latest version
```

### Prerequisites
- Node.js version 19 or higher

## Architecture

### Configuration (docs.json)
- Central configuration file defining navigation structure, theming, tabs, and global settings
- Navigation is organized into tabs (Guides, API reference)
- Each tab contains groups of pages
- Theme colors, logos, navbar, and footer are configured here
- Refer to the [docs.json schema](https://mintlify.com/docs.json) when building the docs.json file and site navigation

### Adding New Pages
When adding new pages:
1. Create the `.mdx` file in the appropriate directory
2. Add the page path (without extension) to the corresponding navigation group in `docs.json`

Example: A page at `/api-reference/endpoint/get.mdx` is referenced as `"api-reference/endpoint/get"` in docs.json.

## Content strategy
- Document just enough for user success - not too much, not too little
- Prioritize accuracy and usability
- Make content evergreen when possible
- Search for existing content before adding anything new. Avoid duplication unless it is done for a strategic reason
- Check existing patterns for consistency
- Start by making the smallest reasonable changes


## Frontmatter requirements for pages
- title: Clear, descriptive page title
- description: Concise summary for SEO/navigation

## Writing standards
- Second-person voice ("you")
- Prerequisites at start of procedural content
- Test all code examples before publishing
- Match style and formatting of existing pages
- Include both basic and advanced use cases
- Language tags on all code blocks
- Alt text on all images
- Relative paths for internal links

## Git workflow
- NEVER use --no-verify when committing
- Ask how to handle uncommitted changes before starting
- Create a new branch when no clear branch exists for changes
- Commit frequently throughout development
- NEVER skip or disable pre-commit hooks

## Do not
- Skip frontmatter on any MDX file
- Use absolute URLs for internal links
- Include untested code examples
- Make assumptions - always ask for clarification

## Troubleshooting
- If preview doesn't match production: run `mint update`
- If encountering sharp module errors: upgrade to Node v19+, reinstall CLI
- For unknown errors: delete `~/.mintlify` folder and restart