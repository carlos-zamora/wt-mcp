# wt-mcp

An [MCP](https://modelcontextprotocol.io/) server that helps reference and manage [Windows Terminal](https://github.com/microsoft/terminal) settings and other terminal-related customizations. Built with C# and the [ModelContextProtocol C# SDK](https://www.nuget.org/packages/ModelContextProtocol).

> [!CAUTION]
> This tool is powered by AI, which can make mistakes. Always review proposed changes before applying them.

## Tools

| Tool area | What it does |
|---|---|
| **Settings** | Read, preview, and apply [JSON Patch (RFC 6902)](https://datatracker.ietf.org/doc/html/rfc6902) changes to `settings.json` across release channels (Stable, Preview, Canary, Dev) |
| **Fragments** | Create, read, update, and delete [fragment extensions](https://learn.microsoft.com/windows/terminal/json-fragment-extensions) — portable profiles, color schemes, and actions |
| **Oh My Posh** | Detect installation, list/read themes, preview and apply config patches, set active theme in the PowerShell profile |
| **Shell Integration** | Check and configure [shell integration](https://learn.microsoft.com/windows/terminal/tutorials/shell-integration) (OSC 133 sequences, autoMarkPrompts, scrollbar marks) |
| **Snippets** | Manage `.wt.json` per-directory snippets and `sendInput` actions in settings or fragments |

All mutating tools follow a **preview-then-apply** pattern — a `Preview*` call returns a unified diff for the user to review, then the corresponding write call applies the change.

## Setup

Add the following to your MCP configuration:

- **VS Code**: `<WORKSPACE>/.vscode/mcp.json`
- **Visual Studio**: `<SOLUTION>\.mcp.json`

```json
{
  "servers": {
    "wt-mcp": {
      "type": "stdio",
      "command": "dnx",
      "args": ["wt-mcp", "--prerelease", "--yes"]
    }
  }
}
```

The `--prerelease` flag is required while `wt-mcp` is in beta. Once a stable version is published, users can drop it and just use `["wt-mcp", "--yes"]`. To pin a specific version, use `"wt-mcp@0.1.0-beta"` instead.

No .NET runtime is required — the package is self-contained.

## Try it out

Once configured, try asking Copilot Chat things like:

- *"Change my Windows Terminal theme to Light"*
- *"Create a fragment with a new SSH profile"*
- *"What's my Oh My Posh theme? Change the git segment color to blue"*
- *"Set up shell integration for my terminal"*
- *"Add a snippet to restart the dev server in this project"*

## Developing locally

To run the MCP server from source, configure your IDE to use `dotnet run` instead of `dnx`:

```json
{
  "servers": {
    "wt-mcp": {
      "type": "stdio",
      "command": "dotnet",
      "args": ["run", "--project", "<PATH TO PROJECT DIRECTORY>"]
    }
  }
}
```

This requires the .NET 10 SDK. See the [VS Code](https://code.visualstudio.com/docs/copilot/chat/mcp-servers) or [Visual Studio](https://learn.microsoft.com/visualstudio/ide/mcp-servers) MCP docs for more details.

## Publishing to NuGet.org

```bash
dotnet pack -c Release
dotnet nuget push bin/Release/*.nupkg --api-key <your-api-key> --source https://api.nuget.org/v3/index.json
```

Package metadata is configured in `wt-mcp.csproj` and the server declaration in `.mcp/server.json`.

## More information

- [MCP Documentation](https://modelcontextprotocol.io/)
- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [MCP C# SDK](https://modelcontextprotocol.github.io/csharp-sdk)
- [MCP GitHub Organization](https://github.com/modelcontextprotocol)
