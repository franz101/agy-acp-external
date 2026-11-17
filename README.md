# agy-acp-external

ACP stdio wrapper for Google Antigravity (`agy_acp_server.par`) that enables **Log in with Google** in Zed over **Remote SSH** (and headless macOS/Linux hosts).

## Why

When Zed connects to a remote host over SSH, `agy_acp_server.par` runs on the remote machine and:
1. Probes the remote macOS keychain (which fails in a non-GUI SSH session).
2. Starts a loopback OAuth callback server on the remote host and calls `webbrowser.open(...)` remotely, while only printing the OAuth URL to `stderr` (hidden in `Zed.log`).

`agy-acp-external` wraps `agy_acp_server.par`:
- Sets `AGY_ACP_FORCE_FILE_STORAGE=1` to avoid keychain prompts/probes.
- Rewrites the `oauth-personal` method in the ACP `initialize` response to `type: "terminal"` (`args: ["--login"]`).
- On `--login`, prints the clickable Google OAuth URL in Zed's terminal and accepts **either** an SSH port-forwarded callback (`ssh -N -L <port>:127.0.0.1:<port> <remote-host>`) **or** pasting the redirected URL / authorization code directly into the terminal.
- Saves the token to `~/.gemini/antigravity-acp/acp_token.json` (`0600`) and sets `auth.type = "oauth-personal"` in `~/.gemini/antigravity-acp/settings.json`.

## Setup

1. Install `antigravity-acp` once via Zed's registry on the remote host (so `agy_acp_server.par` is downloaded under Zed's `external_agents/registry/antigravity-acp/`).
2. Copy `agy-acp-external` to the remote host (e.g. `~/.local/bin/agy-acp-external`) and make it executable:
   ```sh
   chmod +x ~/.local/bin/agy-acp-external
   ```
3. Add `agy-external` to your local Zed `settings.json`:
   ```json
   {
     "agent_servers": {
       "agy-external": {
         "type": "custom",
         "command": "/Users/<remote-user>/.local/bin/agy-acp-external",
         "args": []
       }
     }
   }
   ```
4. Select **agy-external** in Zed's Agent panel and click **Log in with Google**.

## Environment Overrides

- `AGY_ACP_BIN`: Explicit path to `agy_acp_server.par`.
- `AGY_ACP_CLIENT_ID` / `AGY_ACP_CLIENT_SECRET`: Override OAuth client credentials (otherwise extracted from `agy_acp_server.par`).
- `GEMINI_HOME`: Override `~/.gemini` directory.
