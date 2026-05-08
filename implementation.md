# Plan: Zed MCP Extension for Z.AI Zread Server

## Context

The zread MCP server (`https://api.z.ai/api/mcp/zread/mcp`) provides 3 tools for accessing GitHub repository documentation, code structure, and file contents. We need to build a Zed editor extension that bridges to this remote HTTP MCP server using `mcp-remote` as a stdio-to-HTTP bridge.

This is structurally identical to the existing `zed-mcp-server-zai-web-reader` extension (same Z.AI platform, same `Authorization: Bearer` auth pattern, same author). The only differences are the extension ID/name, the MCP URL, and the description.

## Files to Create

### 1. `extension.toml`
- ID: `mcp-server-zai-zread`
- Name: `Z.AI Zread MCP Server`
- Description: References GitHub repo docs/code access
- `repository`: `https://github.com/minggliangg/zed-mcp-server-zai-zread`
- `[context_servers.mcp-server-zai-zread]` section

### 2. `Cargo.toml`
- Package name: `mcp_server_zai_zread` (snake_case)
- Same deps as web-reader: `zed_extension_api = "0.7.0"`, `serde = "1.0"`, `schemars = "0.8"`
- `crate-type = ["cdylib"]`

### 3. `src/mcp_server_zai_zread.rs`
Adapted from `zed-mcp-server-zai-web-reader/src/mcp_server_zai_web_reader.rs` with these changes:
- `DEFAULT_MCP_URL` = `https://api.z.ai/api/mcp/zread/mcp`
- Struct names: `ZaiZreadMcpExtension`, `ZaiZreadContextServerSettings`
- `for_project("mcp-server-zai-zread", ...)` (matches extension.toml ID)
- Settings field: `zai_api_key: Option<String>` (same Z.AI platform key)
- Placeholder: `"YOUR_ZAI_API_KEY"` in `.replace()` calls
- `register_extension!(ZaiZreadMcpExtension)`

### 4. `configuration/default_settings.jsonc`
```json
{ "zai_api_key": "YOUR_ZAI_API_KEY" }
```

### 5. `configuration/installation_instructions.md`
```markdown
Get your API key from [z.ai/manage-apikey/apikey-list](https://z.ai/manage-apikey/apikey-list) and set it below.
```

### 6. `.gitignore`
```
target/
*.wasm
node_modules/
.DS_Store
```

### 7. `.github/workflows/release.yml`
CI/CD using `huacnlee/zed-extension-action@v2.0.0`, extension-name: `mcp-server-zai-zread`, push-to: `minggliangg/extensions`.

### 8. `LICENSE`
Apache-2.0 (matching web-reader).

## Implementation Order

1. Create all source files (extension.toml, Cargo.toml, Rust source, configuration files)
2. Initialize git repo
3. Build: `cargo check` then `cargo build --target wasm32-wasip1 --release`
4. Copy WASM to project root as `extension.wasm`

## Verification

1. `cargo check` passes
2. WASM build succeeds (`cargo build --target wasm32-wasip1 --release`)
3. Install as dev extension in Zed: `Cmd+Shift+P` -> "install dev extension" -> select project root
4. Configure in Zed settings.json under `"context_servers"` with a valid API key
5. Test the 3 tools (`search_doc`, `get_repo_structure`, `read_file`) via Zed's assistant panel
