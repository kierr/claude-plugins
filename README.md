# Kierr's Claude Code Plugins Marketplace

A collection of plugins for Claude Code to enhance development workflows.

## Available Plugins

### GitHub PR Comments Plugin
**Name**: `gh-pr-comments`  
**Description**: Managing GitHub pull request review comments through extraction, validation, fixes, and resolution workflows using gh CLI.

This plugin helps manage GitHub pull request review comments through a complete workflow:
- Extract unresolved comments from the current PR
- Validate each comment against project standards
- Apply fixes for valid comments
- Resolve addressed comment threads

## Installation

1. Add this marketplace using the GitHub URL:
   ```
   /plugin marketplace add https://github.com/kierr/claude-plugins
   ```

2. Browse available plugins:
   ```
   /plugin
   ```

3. Install desired plugins:
   ```
   /plugin install gh-pr-comments@kierr
   ```

## Plugin Development

To add a new plugin to this marketplace:

1. Create a new plugin repository following the Claude Code plugin structure
2. Add an entry to the `marketplace.json` file in this repository
3. Submit a pull request with your changes

## Marketplace Structure

The marketplace.json file contains metadata about available plugins:
- Plugin name
- Source location (GitHub URL)
- Description

## Contributing

Feel free to submit pull requests to add new plugins to this marketplace or improve existing ones.
