---
name: agent-browser
description: Automates browser interactions for web testing, form filling, screenshots, and data extraction. Use when the user needs to navigate websites, interact with web pages, fill forms, take screenshots, test web applications, or extract information from web pages.
allowed-tools:
  - Bash(npx agent-browser:*)
---

# Agent Browser

A browser automation skill for AI agents. Automates browser interactions for web testing, form filling, screenshots, and data extraction.

## When to Use This Skill

Use this skill when the user wants to:

- Navigate to websites and interact with web pages
- Fill out forms or click buttons on websites
- Take screenshots of web pages
- Extract information from web pages
- Test web applications
- Automate browser workflows

## How It Works

This skill uses Playwright under the hood via the `npx agent-browser` command. It provides a simple interface for controlling a browser programmatically.

## Basic Usage

### Opening a Browser

```bash
npx agent-browser open [url]
```

### Taking a Screenshot

```bash
npx agent-browser screenshot [url] --output [filename]
```

### Extracting Page Content

```bash
npx agent-browser extract [url] --selector [css-selector]
```

### Clicking Elements

```bash
npx agent-browser click [url] --selector [css-selector]
```

### Filling Forms

```bash
npx agent-browser fill [url] --selector [css-selector] --value [text]
```

## Workflow Examples

### Test a Web Application

```bash
# 1. Open the app
npx agent-browser open http://localhost:3000

# 2. Take a screenshot to see the current state
npx agent-browser screenshot http://localhost:3000 --output before.png

# 3. Fill a form
npx agent-browser fill http://localhost:3000 --selector "#email" --value "test@example.com"

# 4. Click submit
npx agent-browser click http://localhost:3000 --selector "#submit-button"

# 5. Take screenshot after
npx agent-browser screenshot http://localhost:3000 --output after.png
```

### Extract Data from a Page

```bash
# Extract all links from a page
npx agent-browser extract https://example.com --selector "a"

# Extract text from specific elements
npx agent-browser extract https://example.com --selector ".product-title"
```

## Prerequisites

Node.js must be installed. The first run will automatically install Playwright browsers.

## Tips

- Always take a screenshot first to understand the current state of the page
- Use CSS selectors to target specific elements
- For dynamic pages, you may need to wait for elements to load
- Check screenshots to verify actions had the expected effect
