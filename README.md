# Filesystem-Extra MCP Server

> **Note**: This is a fork of the official [`@modelcontextprotocol/server-filesystem@0.6.2`](https://github.com/modelcontextprotocol/servers) with added `.gitignore` support.

[MCP Filesystem Document](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem#readme)

## Overview

Providing automatic filtering of files and directories based on gitignore rules. This feature ensures that sensitive or unnecessary files (such as `node_modules`, build artifacts, or private configuration files) are automatically excluded from MCP client access.

## Features

- **Automatic `.gitignore` Detection**: The server automatically detects and applies `.gitignore` files in any directory
- **Hierarchical Rule Application**: Supports nested `.gitignore` files with proper precedence rules
- **Standard GitIgnore Syntax**: Full support for standard gitignore patterns including wildcards, negation, and directory-specific rules
- **Performance Optimization**: Gitignore rules are cached to ensure minimal performance impact
- **Transparent Integration**: Works seamlessly with all existing file operations

## How It Works

1. When accessing any file or directory, the server checks for `.gitignore` files in the current directory and all parent directories up to the allowed root
2. Rules from multiple `.gitignore` files are applied in order, with closer `.gitignore` files taking precedence
3. Any file or directory matching ignore patterns is completely hidden from the MCP client

## Examples

### Basic Directory Structure
```
project/
├── .gitignore
├── src/
│   ├── main.js
│   └── config.js
├── node_modules/       # Ignored
├── dist/              # Ignored
└── .env               # Ignored
```

### Sample `.gitignore` Content
```gitignore
# Dependencies
node_modules/
*.lock

# Build outputs
dist/
build/
*.min.js

# Environment files
.env
.env.*

# IDE files
.vscode/
.idea/
*.swp

# OS files
.DS_Store
Thumbs.db
```

### Nested `.gitignore` Files
```
project/
├── .gitignore          # Root gitignore
├── src/
│   ├── .gitignore     # Src-specific rules
│   └── temp/          # May be ignored by src/.gitignore
└── docs/
    ├── .gitignore     # Docs-specific rules
    └── drafts/        # May be ignored by docs/.gitignore
```

## Behavior with MCP Operations

### List Directory
When listing a directory, ignored files and directories are automatically filtered out:
```bash
# If node_modules is in .gitignore
$ list_directory /project
[DIR] src
[DIR] docs
[FILE] package.json
[FILE] README.md
# node_modules is not shown
```

### Read File
Attempting to read an ignored file will result in an access denied error:
```bash
$ read_file /project/.env
Error: Access denied - path is ignored by .gitignore: /project/.env
```

### Search Files
File searches automatically skip ignored files and directories:
```bash
$ search_files /project "config"
/project/src/config.js
/project/docs/configuration.md
# Ignored files like .env.config are not included
```

### Directory Tree
The directory tree excludes ignored items:
```json
[
  {
    "name": "src",
    "type": "directory",
    "children": [
      {
        "name": "main.js",
        "type": "file"
      }
    ]
  },
  {
    "name": "README.md",
    "type": "file"
  }
]
```

## Important Notes

1. **Security First**: This feature is designed to prevent accidental exposure of sensitive files. Once a file is ignored by `.gitignore`, it cannot be accessed through any MCP operation.

2. **No Override Option**: For security reasons, there is no way to override gitignore rules through the MCP interface.

3. **Performance**: The gitignore rules are cached per directory to ensure minimal performance impact on file operations.

4. **Symlinks**: If a symlink points to an ignored file or directory, access will be denied.

## Common Use Cases

### Development Projects
Automatically hide:
- Dependencies (`node_modules/`, `vendor/`, `packages/`)
- Build artifacts (`dist/`, `build/`, `out/`)
- IDE configurations (`.vscode/`, `.idea/`)
- Temporary files (`*.tmp`, `*.cache`)

### Security-Sensitive Applications
Automatically protect:
- Environment files (`.env`, `.env.local`)
- Private keys (`*.pem`, `*.key`)
- Configuration files with secrets
- Database files

## Troubleshooting

### Files Not Being Ignored
1. Check if the `.gitignore` file is in the correct directory
2. Verify the pattern syntax matches gitignore standards
3. Remember that rules in closer `.gitignore` files override parent rules

### Cannot Access Needed Files
1. Check if the file is matched by a gitignore pattern
2. Update the `.gitignore` file to exclude the file from ignore rules using negation (`!filename`)
3. Ensure the file path is within the allowed directories

### Performance Issues
If you experience performance issues with large directory trees:
1. The gitignore cache should handle most cases efficiently
2. Consider structuring your `.gitignore` rules to be more specific
3. Avoid overly complex patterns with multiple wildcards

## Migration Guide

If you're upgrading from a previous version without gitignore support:

1. **No Action Required**: The feature is automatically enabled and backward compatible
2. **Review Existing `.gitignore` Files**: Ensure your existing `.gitignore` files don't accidentally hide files you need to access
3. **Test Access**: Verify that your MCP client can still access all required files
4. **Update Documentation**: Inform users that ignored files will no longer be accessible

## API Changes

All existing APIs remain the same, but with the following behavioral changes:
- File operations on ignored paths will return "Access denied" errors
- Directory listings will exclude ignored items
- Search operations will skip ignored files and directories
- All tool descriptions have been updated to mention gitignore support
