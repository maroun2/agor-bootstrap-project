---
name: agor-mcp-add
description: Add a new MCP server to Agor's database configuration
triggers:
  - add mcp server
  - install mcp server
  - configure new mcp
  - setup mcp in agor
  - add mcp to agor
---

# Add MCP Server to Agor

This skill adds a new MCP (Model Context Protocol) server to Agor's database configuration.

## Usage

When invoked, follow these steps:

### 1. Gather Information

Ask the user for:
- **Server name** (internal ID, lowercase, no spaces)
- **Display name** (human-readable)
- **Description** (brief description of what it does)
- **Command** (full path to MCP server executable)
- **Arguments** (array of args, usually `["mcp"]` or similar)
- **Environment variables** (object with env vars needed)
- **Scope** (usually "session" or "user")

### 2. Create Database Backup

```bash
cp ~/.agor/agor.db ~/.agor/agor.db.backup-$(date +%Y%m%d-%H%M%S)
```

### 3. Generate UUIDv7

```bash
python3 << 'EOF'
import uuid
import time

def uuid7():
    timestamp_ms = int(time.time() * 1000)
    uuid_int = (timestamp_ms << 80) | (uuid.uuid4().int & ((1 << 80) - 1))
    return str(uuid.UUID(int=uuid_int, version=7))

print(uuid7())
EOF
```

### 4. Prepare JSON Data

Structure the data field as JSON:
```json
{
  "display_name": "Server Display Name",
  "description": "What this server does",
  "command": "/path/to/executable",
  "args": ["mcp"],
  "env": {
    "ENV_VAR_1": "value1",
    "ENV_VAR_2": "value2"
  }
}
```

### 5. Insert into Database

```bash
sqlite3 ~/.agor/agor.db << EOF
INSERT INTO mcp_servers (
  mcp_server_id,
  created_at,
  updated_at,
  name,
  transport,
  scope,
  enabled,
  owner_user_id,
  source,
  data
) VALUES (
  '<generated-uuid>',
  $(date +%s)000,
  NULL,
  '<server-name>',
  'stdio',
  '<scope>',
  1,
  NULL,
  'user',
  '<json-data-escaped>'
);
EOF
```

### 6. Verify Installation

```bash
sqlite3 ~/.agor/agor.db "SELECT name, data FROM mcp_servers WHERE name='<server-name>';" | jq '.'
```

### 7. Instruct User

Tell the user:
1. The MCP server has been added to Agor's configuration
2. They need to restart their Agor session to load the new server
3. After restart, use `list MCP servers` to verify it's available
4. If OAuth is needed, they'll be prompted on first use

## Example MCP Servers

### Brave Search
- Command: `npx -y @modelcontextprotocol/server-brave-search`
- Env: `BRAVE_API_KEY`

### Filesystem
- Command: `npx -y @modelcontextprotocol/server-filesystem`
- Args: `["/path/to/allowed/dir"]`

### GitHub
- Command: `npx -y @modelcontextprotocol/server-github`
- Env: `GITHUB_PERSONAL_ACCESS_TOKEN`

### PostgreSQL
- Command: `npx -y @modelcontextprotocol/server-postgres`
- Env: `POSTGRES_CONNECTION_STRING`

### Puppeteer
- Command: `npx -y @modelcontextprotocol/server-puppeteer`

### Slack
- Command: `npx -y @modelcontextprotocol/server-slack`
- Env: `SLACK_BOT_TOKEN`, `SLACK_TEAM_ID`

## Troubleshooting

### Tools Not Appearing
1. Verify the MCP server executable exists and is executable
2. Check environment variables are correct
3. Restart the Agor session completely
4. Check Agor logs for MCP connection errors

### OAuth Required
Some servers (like Slack, GitHub) require OAuth authentication. After adding:
1. Restart session
2. Agor will prompt for OAuth on first use
3. Complete OAuth flow in browser
4. Tokens stored in `user_mcp_oauth_tokens` table

### Database Schema
```sql
-- View all MCP servers
SELECT mcp_server_id, name, enabled, scope FROM mcp_servers;

-- View server details
SELECT data FROM mcp_servers WHERE name='server-name';

-- Disable server
UPDATE mcp_servers SET enabled=0 WHERE name='server-name';

-- Delete server
DELETE FROM mcp_servers WHERE name='server-name';
```

## Notes

- Always create a backup before modifying the database
- UUIDs must be valid UUIDv7 format
- JSON data must be properly escaped for SQL
- Session restart is required to load new servers
- Use `scope='session'` for project-specific, `scope='user'` for global
