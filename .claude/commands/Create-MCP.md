# Complete Guide: Creating New Local MCP Servers

This guide provides a comprehensive, replicable process for creating new MCP servers from scratch using Claude Code, leveraging the MCP Server Chain tools and established patterns.

## Overview

This process enables you to:
1. **Design** - Define server purpose and tools
2. **Generate** - Use Claude Code to create server implementation
3. **Install** - Set up the server locally
4. **Configure** - Integrate with Claude Code and Cursor IDE
5. **Test** - Verify functionality and troubleshoot
6. **Document** - Record the implementation for future reference

## Prerequisites

Before starting, ensure you have:
- MCP Server Chain installed and configured (`terminal-claude` and `auto-fix` servers)
- Python 3.8+ with MCP SDK (`pip install mcp`)
- Node.js and npm (for Node.js-based servers)
- Access to modify `~/.claude.json` and `~/.cursor/mcp.json`
- Claude Code with working MCP integration

## Step 1: Environment Preparation

### 1.1 Verify MCP Server Chain Status
```bash
# In Claude Code, use auto-fix server
Use tool: diagnose_mcp_issues
```

**Expected Output:**
-  Node.js and Python available
-  Claude Code accessible
-  MCP servers visible
-  Configuration files valid

### 1.2 Set Up Development Environment
```bash
# In Claude Code, use terminal-claude server
Use tool: launch_terminal
Parameters:
  command: "cd /home/ivan/mcp-server-chain && mkdir mcp_[your_server_name] && cd mcp_[your_server_name]"
```

### 1.3 Launch Development Instance (Optional)
```bash
# For complex servers, launch dedicated Claude Code instance
Use tool: launch_claude
Parameters:
  new_instance: true
  workspace: "/home/ivan/mcp-server-chain/mcp_[your_server_name]"
```

## Step 2: Server Design and Specification

### 2.1 Define Server Purpose
Create a clear specification following this template:

```markdown
# MCP Server Specification: [Server Name]

## Purpose
[Brief description of what the server does]

## Tools Required
1. **tool_name_1** - Description and purpose
   - Input: parameter descriptions
   - Output: what it returns
   - Use case: when to use this tool

2. **tool_name_2** - Description and purpose
   [... continue for all tools]

## Security Considerations
- File access restrictions
- Command execution limitations
- Data validation requirements

## Integration Requirements
- Dependencies needed
- External tools or APIs
- Configuration requirements
```

### 2.2 Reference Existing Patterns
Study the established server patterns:

```bash
# Look at existing server structures
ls -la /home/ivan/mcp-server-chain/
# mcp_terminal_claude/ - Terminal and process management
# mcp_auto_fix/ - Configuration and system management
```

**Common Patterns:**
- **Simple Tools**: Like `echo`, `add` from everything server
- **System Integration**: Like `launch_claude`, `launch_terminal`
- **Configuration Management**: Like `validate_configurations`, `repair_server_config`
- **File Operations**: Reading, writing, validation
- **Process Management**: Running commands, monitoring

## Step 3: Server Implementation Request

### 3.1 Basic Server Creation Prompt
Use this template in Claude Code:

```markdown
Create a new MCP server called 'mcp_[name]' based on the established patterns in /home/ivan/mcp-server-chain/.

## Server Specification:
**Purpose:** [Your server purpose]

**Tools to implement:**
1. **[tool_name]** - [description]
   - Input: [parameters with types]
   - Output: [return description]
   - Function: [what it does]

[... list all tools]

## Implementation Requirements:

### Project Structure:
```
mcp_[name]/
   __init__.py
   __main__.py
   setup.py
   README.md
   [additional files if needed]
```

### Technical Requirements:
- Follow the exact patterns from mcp_terminal_claude and mcp_auto_fix
- Use Python MCP SDK with async/await
- Include comprehensive error handling
- Add input validation with proper schemas
- Implement security measures (file access restrictions, command validation)
- Follow the established naming conventions

### Code Structure Template:
```python
#!/usr/bin/env python3
import asyncio
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import (
    CallToolRequest, CallToolResult, ListToolsRequest, 
    ListToolsResult, TextContent, Tool
)

server = Server("mcp-[name]")

@server.list_tools()
async def list_tools() -> ListToolsResult:
    # Define tools here

@server.call_tool()
async def call_tool(name: str, arguments: Dict[str, Any]) -> CallToolResult:
    # Handle tool calls here

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(read_stream, write_stream, server.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

### Security Implementation:
- Restrict file access to /home/ivan and subdirectories
- Validate all input parameters
- Use timeouts for external operations
- Handle errors gracefully without exposing system details

### Installation Setup (setup.py):
- Follow the exact pattern from existing servers
- Include proper dependencies
- Set up console scripts entry point

### Documentation (README.md):
- Tool descriptions and usage examples
- Installation instructions
- Configuration examples for both Claude Code and Cursor IDE
- Security considerations
- Troubleshooting guide

Please create all files with complete implementations, following the established patterns exactly.
```

### 3.2 Advanced Server Creation Prompt
For more complex servers:

```markdown
Create an advanced MCP server 'mcp_[name]' with the following specifications:

## Advanced Requirements:
- Multiple tool categories (like auto-fix server)
- Configuration file management
- External dependency integration
- Comprehensive error reporting
- Progress tracking for long operations
- Resource management and cleanup

## Implementation Pattern:
Use the mcp_auto_fix server as the primary reference for:
- Complex tool organization
- Configuration file handling
- System integration
- Error diagnosis and reporting
- Backup and recovery mechanisms

## Specific Tools Needed:
[Your detailed tool specifications]

## Integration Requirements:
- CLI command integration
- File system operations
- Process management
- Configuration validation
- System health checking

Please implement following the advanced patterns from mcp_auto_fix, including all error handling, backup mechanisms, and comprehensive validation.
```

## Step 4: Installation and Setup

### 4.1 Install the New Server
```bash
# After Claude Code creates the server files
Use tool: launch_terminal
Parameters:
  command: "cd /home/ivan/mcp-server-chain/mcp_[name] && pip install -e ."
```

### 4.2 Test Server Functionality
```bash
# Test the server starts correctly
Use tool: launch_terminal
Parameters:
  command: "cd /home/ivan/mcp-server-chain && PYTHONPATH=/home/ivan/mcp-server-chain timeout 10s python -m mcp_[name]"
```

**Expected Result:** Server starts without errors

### 4.3 Verify Installation
```bash
# Check Python can import the module
Use tool: launch_terminal  
Parameters:
  command: "python -c 'import mcp_[name]; print(\" Import successful\")'"
```

## Step 5: Configuration Integration

### 5.1 Add to Claude Code Configuration
```bash
# Use auto-fix server to add the new server
Use tool: repair_server_config
Parameters:
  server_name: "[name]"
  config_type: "claude"

# Or configure manually by adding to ~/.claude.json:
{
  "mcpServers": {
    "[name]": {
      "type": "stdio",
      "command": "python",
      "args": ["-m", "mcp_[name]"],
      "env": {
        "PYTHONPATH": "/home/ivan/mcp-server-chain"
      }
    }
  }
}
```

### 5.2 Add to Cursor IDE Configuration
```bash
# Use auto-fix server for Cursor configuration
Use tool: repair_server_config
Parameters:
  server_name: "[name]"
  config_type: "cursor"

# Or manually add to ~/.cursor/mcp.json:
{
  "mcpServers": {
    "[name]": {
      "command": "python",
      "args": ["-m", "mcp_[name]"],
      "timeout": 300
    }
  }
}
```

### 5.3 Verify Configuration
```bash
# Check server appears in MCP list
Use tool: launch_terminal
Parameters:
  command: "claude mcp list"
```

**Expected Output:** Your new server appears in the list

## Step 6: Testing and Validation

### 6.1 Comprehensive Server Testing
```bash
# Test all servers including the new one
Use tool: test_all_servers
Parameters:
  timeout: 10
```

### 6.2 Validate Configurations
```bash
# Ensure all configurations are valid
Use tool: validate_configurations
```

### 6.3 Test Individual Tools
Restart Claude Code and Cursor IDE, then test each tool:
1. Verify green indicators appear
2. Test each tool with valid inputs
3. Test error handling with invalid inputs
4. Verify tool outputs are as expected

## Step 7: Documentation and Integration

### 7.1 Document the New Server
Add to your MCP servers log:

```bash
# Use auto-fix server to create documentation entry
# Or manually update ~/mcp-servers-log.md
```

### 7.2 Update MCP Server Chain
If the server should be part of the standard chain:
1. Update install_mcp_chain.sh
2. Update README.md in the chain project
3. Add to automated configuration scripts

## Common Server Patterns and Examples

### Pattern 1: File Operations Server
```markdown
Purpose: File management and operations
Tools: create_file, read_file_info, backup_file, find_files
Security: Restrict to specific directories
Reference: Use filesystem server patterns
```

### Pattern 2: Development Tools Server
```markdown
Purpose: Development workflow automation
Tools: run_tests, build_project, format_code, check_dependencies
Security: Validate commands, sandbox execution
Reference: Use terminal-claude patterns
```

### Pattern 3: Configuration Management Server
```markdown
Purpose: Manage application configurations
Tools: validate_config, backup_config, restore_config, sync_configs
Security: Configuration file validation
Reference: Use auto-fix server patterns
```

### Pattern 4: API Integration Server
```markdown
Purpose: Integrate with external APIs
Tools: api_call, authenticate, cache_response, format_data
Security: API key management, rate limiting
Reference: Use fetch server patterns
```

### Pattern 5: System Monitoring Server
```markdown
Purpose: Monitor system health and performance
Tools: check_disk_space, monitor_processes, get_system_info, cleanup_logs
Security: Read-only operations, safe commands
Reference: Use diagnostic patterns from auto-fix
```

## Pre-built Server Templates

### Template 1: Simple Utility Server
```bash
# Request this exact template:
"Create a simple MCP server called 'mcp_utils' with these tools:
- generate_uuid: Generate unique identifiers
- hash_text: Hash text with different algorithms
- encode_decode: Base64 and URL encoding/decoding
- validate_email: Email format validation
- format_json: Pretty-print JSON strings

Use the simple patterns from the everything server."
```

### Template 2: File Operations Server
```bash
"Create a file operations MCP server called 'mcp_fileops' with:
- safe_read: Read files with size limits and encoding detection
- safe_write: Write files with backup and validation
- file_info: Get detailed file metadata
- find_files: Search files by pattern with exclusions
- compress_folder: Create zip archives safely

Use security patterns from filesystem server and error handling from auto-fix."
```

### Template 3: Development Server
```bash
"Create a development workflow MCP server called 'mcp_devtools' with:
- run_command: Execute development commands safely
- check_ports: Check if ports are available
- format_code: Format code files using standard tools
- git_status: Get git repository status
- dependency_check: Check for missing dependencies

Use process management patterns from terminal-claude and validation from auto-fix."
```

## Troubleshooting

### Server Not Appearing
1. **Check installation**: `python -c "import mcp_[name]"`
2. **Verify PYTHONPATH**: Ensure correct path in configuration
3. **Test manually**: Run server with explicit paths
4. **Check logs**: Look for import or startup errors

### Tools Not Loading
1. **Restart IDEs**: After configuration changes
2. **Check tool schemas**: Validate input schema format
3. **Test server individually**: Ensure server responds correctly
4. **Check permissions**: Verify file access permissions

### Import Errors
1. **Check dependencies**: `pip list | grep mcp`
2. **Verify file structure**: Ensure __init__.py exists
3. **Check Python path**: Verify module location
4. **Reinstall server**: `pip install -e . --force-reinstall`

### Configuration Issues
1. **Validate JSON**: Check configuration file syntax
2. **Use auto-fix**: Run `validate_configurations`
3. **Check backups**: Restore from backup if needed
4. **Restart services**: Restart Claude Code/Cursor IDE

## Advanced Features

### Progress Tracking
```python
# For long-running operations, implement progress tracking
from mcp.types import ProgressToken

async def long_operation(progress_token: ProgressToken = None):
    if progress_token:
        # Send progress updates
        await server.request_context.session.send_progress_notification(
            progress_token, progress=50, total=100
        )
```

### Resource Management
```python
# For servers that manage resources
async def cleanup_resources():
    # Implement proper cleanup
    pass

# Register cleanup on server shutdown
server.on_shutdown(cleanup_resources)
```

### Configuration Files
```python
# For servers that need configuration
import json
from pathlib import Path

config_path = Path.home() / ".mcp_[name]_config.json"
default_config = {"setting1": "value1", "setting2": "value2"}

def load_config():
    if config_path.exists():
        with open(config_path) as f:
            return json.load(f)
    return default_config
```

## Integration with Existing Tools

### Using Auto-Fix Server
- Use `diagnose_mcp_issues` before creating new servers
- Use `install_core_dependencies` to ensure prerequisites
- Use `repair_server_config` to add new servers to configurations
- Use `test_all_servers` to verify everything works together

### Using Terminal-Claude Server
- Use `launch_terminal` for development environment setup
- Use `launch_claude` for dedicated development instances
- Use `check_claude_status` to verify Claude Code availability
- Use `get_recursion_info` to manage development instances

## Automated Server Creation Script

Create a reusable script for rapid server creation:

```bash
#!/bin/bash
# create_mcp_server.sh

SERVER_NAME=$1
if [ -z "$SERVER_NAME" ]; then
    echo "Usage: $0 <server_name>"
    exit 1
fi

echo "=� Creating MCP Server: $SERVER_NAME"

# Create directory structure
mkdir -p "/home/ivan/mcp-server-chain/mcp_$SERVER_NAME"
cd "/home/ivan/mcp-server-chain/mcp_$SERVER_NAME"

# Use Claude Code to generate server
echo "=� Use Claude Code to generate server with specification..."
echo "   Server directory: $(pwd)"
echo "   Server name: mcp_$SERVER_NAME"

# After server generation:
echo "=� Installing server..."
pip install -e .

echo "� Configuring server..."
# Use auto-fix server to configure

echo ">� Testing server..."
# Use auto-fix server to test

echo " Server creation complete!"
echo "= Restart Claude Code and Cursor IDE to see the new server"
```

## Complete Workflow Checklist

### Pre-Creation
- [ ] MCP Server Chain working
- [ ] Development environment ready
- [ ] Server specification defined
- [ ] Security requirements identified

### Creation
- [ ] Server implementation requested from Claude Code
- [ ] All files created with proper structure
- [ ] Error handling and validation included
- [ ] Security measures implemented

### Installation
- [ ] Server installed with `pip install -e .`
- [ ] Import test successful
- [ ] Server starts without errors
- [ ] Dependencies satisfied

### Configuration
- [ ] Added to Claude Code configuration
- [ ] Added to Cursor IDE configuration
- [ ] PYTHONPATH configured correctly
- [ ] Appears in `claude mcp list`

### Testing
- [ ] All tools tested individually
- [ ] Error handling verified
- [ ] Security restrictions confirmed
- [ ] Green indicators in both IDEs

### Documentation
- [ ] README.md complete with examples
- [ ] Added to MCP servers log
- [ ] Configuration examples provided
- [ ] Troubleshooting guide included

## Summary

This guide provides a complete, replicable process for creating new MCP servers using Claude Code and the MCP Server Chain tools. The key to success is:

1. **Proper Environment Setup** - Use the auto-fix and terminal-claude servers
2. **Clear Specifications** - Define exactly what the server should do
3. **Pattern Following** - Use established patterns from existing servers
4. **Comprehensive Testing** - Verify all functionality before deployment
5. **Complete Documentation** - Record everything for future reference

With this process, you can rapidly create new MCP servers that integrate seamlessly with your existing MCP ecosystem and provide powerful new capabilities to Claude Code and Cursor IDE.

---

**Remember**: Each new server adds to your MCP capabilities, creating a powerful, self-expanding development environment. Use the established patterns, leverage the existing tools, and always follow the security and validation practices outlined in this guide.