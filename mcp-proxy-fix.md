# Fixing MCP Server Connection Issues Caused by HTTP Proxy

When using Claude Code with MCP (Model Context Protocol) servers like Figma Desktop, you may encounter connection failures due to HTTP proxy misconfiguration.

## Problem

Running `claude mcp list` shows:

```
figma-desktop: http://127.0.0.1:3845/mcp (HTTP) - ✗ Failed to connect
```

## Root Cause

If your system has `http_proxy` or `https_proxy` environment variables set, localhost connections may be incorrectly routed through the proxy server, resulting in a `502 Bad Gateway` error.

You can verify this by checking your proxy settings:

```bash
echo "http_proxy: $http_proxy"
echo "https_proxy: $https_proxy"
echo "no_proxy: $no_proxy"
```

If `http_proxy` is set but `no_proxy` is empty or doesn't include localhost, that's the problem.

## Diagnosis

1. **Check if the MCP server is running:**

   ```bash
   lsof -i :3845
   ```

   If Figma is listening, you'll see output like:
   ```
   Figma  5673 user  73u  IPv4 ...  TCP localhost:v-one-spp (LISTEN)
   ```

2. **Test connection through proxy vs direct:**

   ```bash
   # Through proxy (fails with 502)
   curl -v http://127.0.0.1:3845/mcp

   # Bypass proxy (works)
   curl --noproxy '*' -v http://127.0.0.1:3845/mcp
   ```

## Solution

Add localhost addresses to the `no_proxy` environment variable.

### Temporary Fix (Current Session)

```bash
export no_proxy="localhost,127.0.0.1,::1"
```

### Permanent Fix

Add the following line to your shell configuration file (`~/.zshrc` or `~/.bashrc`):

```bash
# Bypass proxy for localhost connections
export no_proxy="localhost,127.0.0.1,::1"
```

Then reload:

```bash
source ~/.zshrc
```

## Verification

After applying the fix, test again:

```bash
claude mcp list
```

Expected output:

```
figma-desktop: http://127.0.0.1:3845/mcp (HTTP) - ✓ Connected
```

## Summary

| Issue | Cause | Fix |
|-------|-------|-----|
| MCP connection fails with 502 | Proxy intercepts localhost traffic | Set `no_proxy=localhost,127.0.0.1,::1` |

## Related

- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [MCP Protocol](https://modelcontextprotocol.io/)
