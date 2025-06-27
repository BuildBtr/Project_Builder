# Fix Failed MCP Servers & Install Common Servers

## Problem
When Claude Code's MCP server status shows "Failed", it's typically due to:
1. **Filesystem server**: Missing directory arguments or wrong command structure
2. **Sequential-thinking server**: Wrong package name, missing global scope, or incorrect configuration
3. **Fetch server**: Missing Python package or incorrect configuration
4. **Memory server**: Not installed globally or missing from configuration
5. **General issues**: Missing servers from `claude mcp list` or MCP server UI

## Diagnosis Steps

1. **Check MCP server status**: 
   ```bash
   claude mcp list
   ```
   Look for "Failed" status in Claude Code's MCP server list

2. **Verify Node.js and Python packages exist**:
   ```bash
   which node && node --version
   which python && python --version
   npm list -g | grep -E "(filesystem|sequential-thinking|memory)"
   pip list | grep -E "(mcp-server-fetch|mcp)"
   which mcp-server-memory
   ```

3. **Test servers individually**:
   ```bash
   # Test filesystem server
   timeout 5s npx @modelcontextprotocol/server-filesystem /home/ivan
   
   # Test sequential-thinking server  
   timeout 5s npx @modelcontextprotocol/server-sequential-thinking /home/ivan
   
   # Test fetch server
   timeout 5s python -m mcp_server_fetch
   
   # Test memory server
   timeout 5s mcp-server-memory
   ```

## Root Causes & Solutions

### Filesystem Server Issues

**Problem**: Missing directory arguments or incorrect command structure
**Fix**: Ensure proper `npx` command with package name and directory

### Sequential-Thinking Server Issues

**Problem**: Wrong package name (`@modelcontext` vs `@modelcontextprotocol`) or missing from global scope
**Fix**: Correct package name and add to global mcpServers section

### Fetch Server Issues

**Problem**: Missing Python package `mcp-server-fetch` or incorrect configuration
**Fix**: Install Python package and configure with `python -m mcp_server_fetch`

### Memory Server Issues

**Problem**: Memory server not installed globally or missing from configuration
**Fix**: Clone official repository, build, and install globally with proper configuration

## Comprehensive Solution Script

Create and run this Python script to fix all common MCP server issues:

```bash
cat > /tmp/fix_all_mcp_servers.py << 'EOF'
#!/usr/bin/env python3
import json
import os
from datetime import datetime

def fix_mcp_servers():
    config_path = os.path.expanduser('~/.claude.json')
    
    # Create backup
    backup_path = f"{config_path}.backup-{datetime.now().strftime('%Y%m%d-%H%M%S')}"
    os.system(f"cp {config_path} {backup_path}")
    print(f"Created backup: {backup_path}")
    
    # Install missing packages
    print("📦 Installing missing MCP server packages...")
    os.system("npm install -g @modelcontextprotocol/server-filesystem @modelcontextprotocol/server-sequential-thinking")
    os.system("pip install mcp-server-fetch")
    
    # Install memory server from official repository
    print("📦 Installing Memory MCP server...")
    os.system("cd /tmp && git clone https://github.com/modelcontextprotocol/servers.git || true")
    os.system("cd /tmp/servers/src/memory && npm install && npm run build && npm install -g .")
    
    with open(config_path, 'r') as f:
        config = json.load(f)
    
    # Ensure global mcpServers section exists
    if 'mcpServers' not in config:
        config['mcpServers'] = {}
    
    # Fix/Add sequential-thinking server globally
    config['mcpServers']['sequential-thinking'] = {
        "type": "stdio",
        "command": "npx",
        "args": ["@modelcontextprotocol/server-sequential-thinking", "/home/ivan"],
        "env": {}
    }
    
    # Add/Fix fetch server globally
    config['mcpServers']['fetch'] = {
        "type": "stdio",
        "command": "python",
        "args": ["-m", "mcp_server_fetch"],
        "env": {}
    }
    
    # Add/Fix memory server globally
    config['mcpServers']['memory'] = {
        "type": "stdio",
        "command": "mcp-server-memory",
        "args": [],
        "env": {}
    }
    
    # Process all projects to fix filesystem servers
    if 'projects' in config:
        for project_path, project_config in config['projects'].items():
            if 'mcpServers' in project_config:
                for server_name, server_config in project_config['mcpServers'].items():
                    if 'filesystem' in server_name:
                        # Fix filesystem server configuration
                        server_config['command'] = 'npx'
                        server_config['args'] = ['@modelcontextprotocol/server-filesystem', '/home/ivan']
                    elif 'thinking' in server_name or 'sequential' in server_name:
                        # Fix thinking/sequential server configuration
                        server_config['command'] = 'npx'
                        server_config['args'] = ['@modelcontextprotocol/server-sequential-thinking', '/home/ivan']
    
    # Write back the fixed configuration
    with open(config_path, 'w') as f:
        json.dump(config, f, indent=2)
    
    print("✅ Installed all required packages")
    print("✅ Fixed all MCP server configurations")
    print("✅ Added sequential-thinking to global scope")
    print("✅ Added fetch server to global scope")
    print("✅ Added memory server to global scope")
    print("✅ Fixed filesystem server arguments")
    print("🔄 Restart Claude Code to see changes")

if __name__ == '__main__':
    fix_mcp_servers()
EOF

# Run the fix script
python3 /tmp/fix_all_mcp_servers.py

# Clean up
rm /tmp/fix_all_mcp_servers.py
```

## Manual Alternative

If you prefer manual fixes:

1. **Backup configuration**:
   ```bash
   cp ~/.claude.json ~/.claude.json.backup-$(date +%Y%m%d-%H%M%S)
   ```

2. **Fix package names and add global scope**:
   ```bash
   # Fix wrong package names
   sed -i 's/@modelcontext\/server-sequential-thinking/@modelcontextprotocol\/server-sequential-thinking/g' ~/.claude.json
   
   # Ensure filesystem servers use npx with proper args
   sed -i 's|/home/ivan/.nvm/versions/node/v[^/]*/bin/mcp-server-filesystem|npx|g' ~/.claude.json
   ```

3. **Verify configuration**:
   ```bash
   # Check global servers
   python3 -c "
   import json
   with open('~/.claude.json'.replace('~', '$HOME'), 'r') as f:
       config = json.load(f)
       if 'mcpServers' in config:
           for name, server in config['mcpServers'].items():
               print(f'Global {name}: {server[\"command\"]} {server[\"args\"]}')
       else:
           print('No global mcpServers found')
   "
   ```

4. **Test all servers**:
   ```bash
   timeout 5s npx @modelcontextprotocol/server-filesystem /home/ivan
   timeout 5s npx @modelcontextprotocol/server-sequential-thinking /home/ivan
   timeout 5s python -m mcp_server_fetch
   timeout 5s mcp-server-memory
   ```

## Expected Results

After applying fixes:
- `claude mcp list` shows all servers:
  ```
  filesystem: npx @modelcontextprotocol/server-filesystem /home/ivan
  sequential-thinking: npx @modelcontextprotocol/server-sequential-thinking /home/ivan
  fetch: python -m mcp_server_fetch
  memory: mcp-server-memory
  ```
- Claude Code MCP UI shows all servers as "Connected"
- All servers respond with startup messages when tested

## Server Scope Understanding

- **Global scope**: Servers available to all Claude Code sessions
- **Project scope**: Servers only available to specific project directories
- **Visibility**: Only global servers appear in `claude mcp list` when in home directory

## Troubleshooting

### Server Still Missing
- Ensure server is in global `mcpServers` section, not just project-specific
- Check package installation: `npm list -g @modelcontextprotocol/server-*`

### Server Still Failed
- Verify directory paths exist and are readable
- Check Node.js version compatibility
- Look for typos in JSON configuration
- Ensure no duplicate server names across scopes

### Command Line vs UI Mismatch
- `claude mcp list` shows global servers when run from home directory
- Claude Code UI may show project-specific servers depending on current directory
- Restart Claude Code after configuration changes

## Package Installation

If servers are missing entirely:
```bash
# Install Node.js-based MCP servers
npm install -g @modelcontextprotocol/server-filesystem
npm install -g @modelcontextprotocol/server-sequential-thinking

# Install Python-based MCP servers
pip install mcp-server-fetch

# Install Memory MCP server from source
cd /tmp && git clone https://github.com/modelcontextprotocol/servers.git
cd /tmp/servers/src/memory && npm install && npm run build && npm install -g .
```

## Individual Server Installation & Configuration

### Fetch MCP Server (Web Content Fetching)

The Fetch server provides web content fetching capabilities, converting HTML to markdown.

**Installation:**
```bash
pip install mcp-server-fetch
```

**Configuration for Claude Code:**
```json
{
  "mcpServers": {
    "fetch": {
      "type": "stdio",
      "command": "python",
      "args": ["-m", "mcp_server_fetch"]
    }
  }
}
```

**Capabilities:**
- `fetch` tool: Fetches URLs and converts HTML to markdown
  - `url` (required): URL to fetch
  - `max_length` (optional): Character limit (default: 5000)
  - `start_index` (optional): Start from specific character position
  - `raw` (optional): Get raw content without markdown conversion

**Security Note:** ⚠️ The fetch server can access local/internal IP addresses. Use with caution.

**Testing:**
```bash
# Test the server starts correctly
timeout 5s python -m mcp_server_fetch

# Test with MCP inspector
npx @modelcontextprotocol/inspector python -m mcp_server_fetch
```

### Alternative Installation Methods

**Using uvx (recommended for Python servers):**
```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```

**Using Docker:**
```json
{
  "mcpServers": {
    "fetch": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "mcp/fetch"]
    }
  }
}
```

### Memory MCP Server (Knowledge Graph & Persistent Memory)

The Memory server provides persistent storage capabilities for creating knowledge graphs and maintaining session memory across conversations.

**Installation:**
```bash
# Clone the official MCP servers repository
cd /tmp && git clone https://github.com/modelcontextprotocol/servers.git

# Navigate to memory server directory
cd /tmp/servers/src/memory

# Install dependencies and build
npm install && npm run build

# Install globally
npm install -g .
```

**Configuration for Claude Code:**
```json
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
```

**Capabilities:**
- `create_entities` tool: Create entities in the knowledge graph
- `create_relations` tool: Create relationships between entities
- `add_observations` tool: Add observations about entities
- `delete_entities` tool: Remove entities from the graph
- `delete_observations` tool: Remove specific observations
- `delete_relations` tool: Remove relationships
- `read_graph` tool: Query and read from the knowledge graph
- `search_nodes` tool: Search for specific nodes in the graph
- `open_nodes` tool: Access detailed node information

**Security Note:** ⚠️ The memory server stores data persistently. Consider data privacy when using with sensitive information.

**Testing:**
```bash
# Test the server starts correctly
timeout 5s mcp-server-memory

# Test with MCP inspector (optional)
npx @modelcontextprotocol/inspector mcp-server-memory
```

**Automated Installation Script:**
Create and run this script to automatically install and configure the Memory MCP server:

```bash
cat > /tmp/install_memory_mcp.py << 'EOF'
#!/usr/bin/env python3
import json
import os
from datetime import datetime

def install_memory_mcp():
    config_path = os.path.expanduser('~/.claude.json')
    
    # Create backup
    backup_path = f"{config_path}.backup-{datetime.now().strftime('%Y%m%d-%H%M%S')}"
    if os.path.exists(config_path):
        os.system(f"cp {config_path} {backup_path}")
        print(f"Created backup: {backup_path}")
    
    # Install memory server
    print("📦 Installing Memory MCP server...")
    os.system("cd /tmp && git clone https://github.com/modelcontextprotocol/servers.git || true")
    os.system("cd /tmp/servers/src/memory && npm install && npm run build && npm install -g .")
    
    # Load or create config
    if os.path.exists(config_path):
        with open(config_path, 'r') as f:
            config = json.load(f)
    else:
        config = {}
    
    # Ensure global mcpServers section exists
    if 'mcpServers' not in config:
        config['mcpServers'] = {}
    
    # Add memory server to global scope
    config['mcpServers']['memory'] = {
        "type": "stdio",
        "command": "mcp-server-memory",
        "args": [],
        "env": {}
    }
    
    # Write configuration
    with open(config_path, 'w') as f:
        json.dump(config, f, indent=2)
    
    print("✅ Memory MCP server installed and configured")
    print("🔄 Restart Claude Code to see changes")
    print("📋 Verify with: claude mcp list")

if __name__ == '__main__':
    install_memory_mcp()
EOF

python3 /tmp/install_memory_mcp.py && rm /tmp/install_memory_mcp.py
```