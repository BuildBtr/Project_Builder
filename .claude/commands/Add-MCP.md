# Complete MCP Server Integration Guide

This guide provides a step-by-step process to install, configure, and integrate any MCP server with both Claude Code and Cursor IDE, ensuring complete functionality and green indicators across all tools.

## Overview

This process covers:
1. **Discovery** - Finding and understanding the MCP server
2. **Installation** - Building and installing the server globally
3. **Configuration** - Adding to Claude Code and Cursor IDE configs
4. **Verification** - Testing and ensuring all tools show green indicators
5. **Documentation** - Recording the process for future reference

## Prerequisites

Before starting, ensure you have:
- Node.js and npm installed
- Python and pip installed (for Python-based servers)
- Git for cloning repositories
- Access to modify `~/.claude.json` and `~/.cursor/mcp.json`

## Step 1: Discovery and Analysis

### 1.1 Identify the MCP Server
```bash
# Check if it's an official MCP server
# Official servers: https://github.com/modelcontextprotocol/servers
ls /tmp/servers/src/  # See available official servers

# For third-party servers, check their documentation
git clone <repository-url>
```

### 1.2 Understand Server Type and Requirements
```bash
# Check package.json for Node.js servers
cat package.json

# Check pyproject.toml or setup.py for Python servers
cat pyproject.toml

# Look for README or documentation
cat README.md
```

### 1.3 Analyze Available Tools
```bash
# For Node.js servers, examine main file
grep -n -A 5 -B 2 "name:" index.ts
grep -n "tools:" index.ts

# For Python servers, examine main module
grep -n "tools" *.py
```

## Step 2: Installation Process

### 2.1 For Node.js-based MCP Servers

**Official MCP Servers (from modelcontextprotocol/servers):**
```bash
# Clone the repository if not already available
cd /tmp && git clone https://github.com/modelcontextprotocol/servers.git || true

# Navigate to specific server directory
cd /tmp/servers/src/<server-name>

# Install dependencies
npm install

# Build the server
npm run build

# Install globally
npm install -g .

# Verify installation
which mcp-server-<name>
```

**Third-party Node.js MCP Servers:**
```bash
# Clone the repository
git clone <repository-url>
cd <repository-directory>

# Install dependencies
npm install

# Build if required
npm run build  # Only if package.json has build script

# Install globally
npm install -g .

# Verify installation
which <server-command>
```

### 2.2 For Python-based MCP Servers

```bash
# Clone repository
git clone <repository-url>
cd <repository-directory>

# Install using pip
pip install .

# Or install in development mode
pip install -e .

# Verify installation
python -m <module_name>
```

### 2.3 For Pre-built MCP Servers

```bash
# Install via npm
npm install -g <package-name>

# Install via pip
pip install <package-name>

# Install via other package managers as specified
```

## Step 3: Configuration Integration

### 3.1 Add to Claude Code Configuration

Create or update `~/.claude.json`:

```bash
# Create backup
cp ~/.claude.json ~/.claude.json.backup-$(date +%Y%m%d-%H%M%S)

# Use automated script approach
cat > /tmp/add_mcp_to_claude.py << 'EOF'
#!/usr/bin/env python3
import json
import os
from datetime import datetime

def add_mcp_server():
    config_path = os.path.expanduser('~/.claude.json')
    
    # Get server details from user
    server_name = input("Enter server name: ")
    command = input("Enter command (e.g., 'mcp-server-name' or 'python'): ")
    args_input = input("Enter args (comma-separated, or press Enter for none): ")
    args = [arg.strip() for arg in args_input.split(',')] if args_input.strip() else []
    
    # Create backup
    if os.path.exists(config_path):
        backup_path = f"{config_path}.backup-{datetime.now().strftime('%Y%m%d-%H%M%S')}"
        os.system(f"cp {config_path} {backup_path}")
        print(f"Created backup: {backup_path}")
        
        with open(config_path, 'r') as f:
            config = json.load(f)
    else:
        config = {}
    
    # Ensure global mcpServers section exists
    if 'mcpServers' not in config:
        config['mcpServers'] = {}
    
    # Add server to global scope
    config['mcpServers'][server_name] = {
        "type": "stdio",
        "command": command,
        "args": args,
        "env": {}
    }
    
    # Write configuration
    with open(config_path, 'w') as f:
        json.dump(config, f, indent=2)
    
    print(f" Added {server_name} to Claude Code configuration")
    print("= Restart Claude Code to see changes")
    print("=Ë Verify with: claude mcp list")

if __name__ == '__main__':
    add_mcp_server()
EOF

python3 /tmp/add_mcp_to_claude.py && rm /tmp/add_mcp_to_claude.py
```

**Manual Configuration Example:**
```json
{
  "mcpServers": {
    "<server-name>": {
      "type": "stdio",
      "command": "<command>",
      "args": ["<arg1>", "<arg2>"],
      "env": {}
    }
  }
}
```

### 3.2 Add to Cursor IDE Configuration

Update `~/.cursor/mcp.json`:

```bash
# Create backup
cp ~/.cursor/mcp.json ~/.cursor/mcp.json.backup-$(date +%Y%m%d-%H%M%S)

# Use automated script approach
cat > /tmp/add_mcp_to_cursor.py << 'EOF'
#!/usr/bin/env python3
import json
import os
from datetime import datetime

def add_mcp_to_cursor():
    config_path = os.path.expanduser('~/.cursor/mcp.json')
    
    # Get server details from user
    server_name = input("Enter server name: ")
    command = input("Enter command (e.g., 'mcp-server-name' or 'python'): ")
    args_input = input("Enter args (comma-separated, or press Enter for none): ")
    args = [arg.strip() for arg in args_input.split(',')] if args_input.strip() else []
    
    # Create backup
    if os.path.exists(config_path):
        backup_path = f"{config_path}.backup-{datetime.now().strftime('%Y%m%d-%H%M%S')}"
        os.system(f"cp {config_path} {backup_path}")
        print(f"Created backup: {backup_path}")
        
        with open(config_path, 'r') as f:
            config = json.load(f)
    else:
        config = {"mcpServers": {}}
    
    # Ensure mcpServers section exists
    if 'mcpServers' not in config:
        config['mcpServers'] = {}
    
    # Add server
    config['mcpServers'][server_name] = {
        "command": command,
        "args": args
    }
    
    # Write configuration
    with open(config_path, 'w') as f:
        json.dump(config, f, indent=2)
    
    print(f" Added {server_name} to Cursor IDE configuration")
    print("= Restart Cursor IDE to see changes")

if __name__ == '__main__':
    add_mcp_to_cursor()
EOF

python3 /tmp/add_mcp_to_cursor.py && rm /tmp/add_mcp_to_cursor.py
```

**Manual Configuration Example:**
```json
{
  "mcpServers": {
    "<server-name>": {
      "command": "<command>",
      "args": ["<arg1>", "<arg2>"]
    }
  }
}
```

## Step 4: Testing and Verification

### 4.1 Test Server Functionality
```bash
# Test server starts correctly
timeout 10s <server-command>

# Test with MCP inspector (optional)
npx @modelcontextprotocol/inspector <server-command>
```

### 4.2 Verify Claude Code Integration
```bash
# Check server appears in list
claude mcp list

# Expected output should include your server:
# <server-name>: <command> <args>
```

### 4.3 Verify Cursor IDE Integration

1. **Restart Cursor IDE**
2. **Open Settings** (Ctrl/Cmd + ,)
3. **Navigate to MCP Tools**
4. **Verify your server appears** with:
   - Server name
   - Green indicator (connected)
   - Tool count (e.g., "5 tools enabled")

### 4.4 Test Tool Availability

In Claude Code or Cursor IDE:
```bash
# List available tools from your server
# This should be done within the IDE interface
```

## Step 5: Documentation and Integration

### 5.1 Document Server Details

Create a record of the server configuration:

```bash
cat >> ~/mcp-servers-log.md << EOF

## $(date +%Y-%m-%d): Added <Server Name>

**Source:** <repository-url>
**Installation Method:** <method-used>
**Command:** \`<server-command>\`
**Tools Available:** <number> tools
- tool1: description
- tool2: description
[... list all tools]

**Claude Code Config:**
\`\`\`json
{
  "<server-name>": {
    "type": "stdio",
    "command": "<command>",
    "args": ["<args>"],
    "env": {}
  }
}
\`\`\`

**Cursor IDE Config:**
\`\`\`json
{
  "<server-name>": {
    "command": "<command>",
    "args": ["<args>"]
  }
}
\`\`\`

**Status:**  Fully integrated and verified

---
EOF
```

### 5.2 Update Fix-Failed-MCP.md (if official server)

If adding an official MCP server, update the existing fix guide:

```bash
# Add to the comprehensive solution script
# Add to testing commands
# Add to expected results
# Add individual server section if needed
```

## Common Server Types and Examples

### Memory Server Example
```json
// Claude Code
"memory": {
  "type": "stdio",
  "command": "mcp-server-memory",
  "args": [],
  "env": {}
}

// Cursor IDE
"memory": {
  "command": "mcp-server-memory",
  "args": []
}
```

### Filesystem Server Example
```json
// Claude Code
"filesystem": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/ivan"],
  "env": {}
}

// Cursor IDE
"filesystem": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/ivan"]
}
```

### Python-based Server Example
```json
// Claude Code
"fetch": {
  "type": "stdio",
  "command": "python",
  "args": ["-m", "mcp_server_fetch"],
  "env": {}
}

// Cursor IDE
"fetch": {
  "command": "python",
  "args": ["-m", "mcp_server_fetch"]
}
```

## Troubleshooting

### Server Not Appearing
1. **Check installation:** `which <server-command>`
2. **Test manually:** `timeout 5s <server-command>`
3. **Verify JSON syntax:** Use JSON validator
4. **Check file permissions:** `ls -la ~/.claude.json ~/.cursor/mcp.json`
5. **Restart IDEs** after configuration changes

### Red Indicators in IDE
1. **Check server logs** in IDE developer tools
2. **Verify command path** is correct
3. **Test dependencies** (Node.js, Python modules)
4. **Check argument format** matches server expectations

### Tools Not Loading
1. **Wait for initialization** (some servers take time)
2. **Check server implements tools correctly**
3. **Verify server responds to list_tools request**
4. **Review server documentation** for special requirements

## Complete Integration Checklist

- [ ] Server installed and accessible via command line
- [ ] Added to `~/.claude.json` (global mcpServers section)
- [ ] Added to `~/.cursor/mcp.json` (mcpServers section)
- [ ] Appears in `claude mcp list` output
- [ ] Shows green indicator in Claude Code MCP UI
- [ ] Shows green indicator in Cursor IDE MCP Tools
- [ ] All tools are enumerated and accessible
- [ ] Tested basic functionality of key tools
- [ ] Configuration backed up
- [ ] Integration documented in log file

## Automated Complete Integration Script

For streamlined integration of any MCP server:

```bash
cat > /tmp/complete_mcp_integration.py << 'EOF'
#!/usr/bin/env python3
import json
import os
import subprocess
from datetime import datetime

def complete_mcp_integration():
    print("=€ Complete MCP Server Integration Tool")
    print("=" * 50)
    
    # Get server information
    server_name = input("Enter server name: ")
    command = input("Enter command: ")
    args_input = input("Enter args (comma-separated, or Enter for none): ")
    args = [arg.strip() for arg in args_input.split(',')] if args_input.strip() else []
    
    # Test server
    print(f"\n>ê Testing server: {command} {' '.join(args)}")
    try:
        result = subprocess.run([command] + args, timeout=5, capture_output=True)
        print(" Server test successful")
    except Exception as e:
        print(f"L Server test failed: {e}")
        return
    
    # Configure Claude Code
    claude_config_path = os.path.expanduser('~/.claude.json')
    claude_backup = f"{claude_config_path}.backup-{datetime.now().strftime('%Y%m%d-%H%M%S')}"
    
    if os.path.exists(claude_config_path):
        os.system(f"cp {claude_config_path} {claude_backup}")
        with open(claude_config_path, 'r') as f:
            claude_config = json.load(f)
    else:
        claude_config = {}
    
    if 'mcpServers' not in claude_config:
        claude_config['mcpServers'] = {}
    
    claude_config['mcpServers'][server_name] = {
        "type": "stdio",
        "command": command,
        "args": args,
        "env": {}
    }
    
    with open(claude_config_path, 'w') as f:
        json.dump(claude_config, f, indent=2)
    
    print(f" Added to Claude Code configuration")
    
    # Configure Cursor IDE
    cursor_config_path = os.path.expanduser('~/.cursor/mcp.json')
    cursor_backup = f"{cursor_config_path}.backup-{datetime.now().strftime('%Y%m%d-%H%M%S')}"
    
    if os.path.exists(cursor_config_path):
        os.system(f"cp {cursor_config_path} {cursor_backup}")
        with open(cursor_config_path, 'r') as f:
            cursor_config = json.load(f)
    else:
        cursor_config = {"mcpServers": {}}
    
    if 'mcpServers' not in cursor_config:
        cursor_config['mcpServers'] = {}
    
    cursor_config['mcpServers'][server_name] = {
        "command": command,
        "args": args
    }
    
    with open(cursor_config_path, 'w') as f:
        json.dump(cursor_config, f, indent=2)
    
    print(f" Added to Cursor IDE configuration")
    
    # Verification
    print(f"\n=Ë Integration Summary:")
    print(f"Server: {server_name}")
    print(f"Command: {command} {' '.join(args)}")
    print(f"Claude backup: {claude_backup}")
    print(f"Cursor backup: {cursor_backup}")
    print(f"\n= Next steps:")
    print(f"1. Restart Claude Code and Cursor IDE")
    print(f"2. Verify with: claude mcp list")
    print(f"3. Check MCP Tools panels for green indicators")

if __name__ == '__main__':
    complete_mcp_integration()
EOF

python3 /tmp/complete_mcp_integration.py && rm /tmp/complete_mcp_integration.py
```

This script automates the entire integration process, ensuring consistent and complete setup across both Claude Code and Cursor IDE.

## Real-World Example: Memory MCP Server Integration

Here's a complete walkthrough using the Memory MCP server as an example:

### Installation
```bash
# Clone official servers repository
cd /tmp && git clone https://github.com/modelcontextprotocol/servers.git

# Navigate to memory server
cd /tmp/servers/src/memory

# Install and build
npm install && npm run build && npm install -g .

# Verify installation
which mcp-server-memory
# Output: /home/ivan/.nvm/versions/node/v23.11.0/bin/mcp-server-memory
```

### Configuration
```bash
# Claude Code (~/.claude.json)
{
  "mcpServers": {
    "memory": {
      "type": "stdio",
      "command": "mcp-server-memory",
      "args": [],
      "env": {}
    }
  }
}

# Cursor IDE (~/.cursor/mcp.json)
{
  "mcpServers": {
    "memory": {
      "command": "mcp-server-memory",
      "args": []
    }
  }
}
```

### Verification
```bash
# Test server
timeout 5s mcp-server-memory
# Output: Knowledge Graph MCP Server running on stdio

# Check Claude Code integration
claude mcp list
# Output includes: memory: mcp-server-memory
```

### Tools Available
The Memory server provides 9 tools:
- `create_entities` - Create entities in knowledge graph
- `create_relations` - Create relationships between entities  
- `add_observations` - Add observations to entities
- `delete_entities` - Remove entities from graph
- `delete_observations` - Remove specific observations
- `delete_relations` - Remove relationships
- `read_graph` - Read entire knowledge graph
- `search_nodes` - Search for nodes in graph
- `open_nodes` - Access detailed node information

### Result
-  Claude Code: Green indicator, 9 tools enabled
-  Cursor IDE: Green indicator, 9 tools enabled
-  Full functionality across both platforms

## Summary

This guide provides a comprehensive, replicable process for integrating any MCP server with both Claude Code and Cursor IDE. Follow the steps sequentially to ensure complete integration with green indicators and full tool availability across both platforms.

The key to success is:
1. **Proper installation** - Ensure the server is globally accessible
2. **Correct configuration** - Match the format requirements for each IDE
3. **Thorough testing** - Verify functionality before and after integration
4. **Complete documentation** - Record the process for future reference

With this guide, you can confidently integrate any MCP server and achieve the desired green indicators showing full tool availability.