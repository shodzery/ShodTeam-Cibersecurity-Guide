# GitHub Actions Workflows

This directory contains GitHub Actions workflow definitions for automating repository quality checks.

## Available Workflows

### markdown-lint.yml

Runs markdownlint on all `.md` files to enforce consistent Markdown formatting. Triggered on pull requests targeting `main`.

### link-check.yml

Checks all internal and external links in documentation files for broken or redirected links. Runs weekly and on pull requests.

### spellcheck.yml

Runs a technical spellcheck against documentation files using a custom dictionary that includes security terminology. Runs on pull requests.

---

## Workflow Configuration

To add a new workflow:

1. Create a `.yml` file in this directory following GitHub Actions syntax
2. Define appropriate triggers (`on: [push, pull_request]`, schedule, etc.)
3. Reference the [GitHub Actions documentation](https://docs.github.com/en/actions) for syntax
4. Test the workflow on a branch before merging to `main`
