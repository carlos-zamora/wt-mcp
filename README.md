# wt-mcp

An [MCP](https://modelcontextprotocol.io/) server that lets AI assistants manage [Windows Terminal](https://github.com/microsoft/terminal) configuration. Built with C# and the [ModelContextProtocol C# SDK](https://www.nuget.org/packages/ModelContextProtocol).

## Tools

| Tool area | What it does |
|---|---|
| **Settings** | Read, preview, and apply [JSON Patch (RFC 6902)](https://datatracker.ietf.org/doc/html/rfc6902) changes to `settings.json` across release channels (Stable, Preview, Canary, Dev) |
| **Fragments** | Create, read, update, and delete [fragment extensions](https://learn.microsoft.com/windows/terminal/json-fragment-extensions) — portable profiles, color schemes, and actions |
| **Oh My Posh** | Detect installation, list/read themes, preview and apply config patches, set active theme in the PowerShell profile |
| **Shell Integration** | Check and configure [shell integration](https://learn.microsoft.com/windows/terminal/tutorials/shell-integration) (OSC 133 sequences, autoMarkPrompts, scrollbar marks) |
| **Snippets** | Manage `.wt.json` per-directory snippets and `sendInput` actions in settings or fragments |

All mutating tools follow a **preview-then-apply** pattern — a `Preview*` call returns a unified diff for the user to review, then the corresponding write call applies the change.

## Supported platforms

The server is built as a self-contained application (no .NET runtime required on the target machine) for:

`win-x64` · `win-arm64` · `osx-arm64` · `linux-x64` · `linux-arm64` · `linux-musl-x64`

To add platforms, update `<RuntimeIdentifiers>` in `wt-mcp.csproj`.

## Developing locally

To test this MCP server from source code (locally) without using a built MCP server package, you can configure your IDE to run the project directly using `dotnet run`.

```json
{
  "servers": {
    "wt-mcp": {
      "type": "stdio",
      "command": "dotnet",
      "args": [
        "run",
        "--project",
        "<PATH TO PROJECT DIRECTORY>"
      ]
    }
  }
}
```

Refer to the VS Code or Visual Studio documentation for more information on configuring and using MCP servers:

- [Use MCP servers in VS Code (Preview)](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)
- [Use MCP servers in Visual Studio (Preview)](https://learn.microsoft.com/visualstudio/ide/mcp-servers)

## Try it out

Once configured, try asking Copilot Chat things like:

- *"Change my Windows Terminal theme to Light"*
- *"Create a fragment with a new SSH profile"*
- *"What's my Oh My Posh theme? Change the git segment color to blue"*
- *"Set up shell integration for my terminal"*
- *"Add a snippet to restart the dev server in this project"*

## Publishing to NuGet.org

Before publishing, update the package metadata in `wt-mcp.csproj` (`<PackageId>`, `<Description>`, etc.) and the server declaration in `.mcp/server.json`. See [configuring inputs](https://aka.ms/nuget/mcp/guide/configuring-inputs) for details.

1. Run `dotnet pack -c Release` to create the NuGet package.
2. Publish with `dotnet nuget push bin/Release/*.nupkg --api-key <your-api-key> --source https://api.nuget.org/v3/index.json`

## Using from NuGet.org

Once published, configure the server in your IDE:

- **VS Code**: Create a `<WORKSPACE DIRECTORY>/.vscode/mcp.json` file
- **Visual Studio**: Create a `<SOLUTION DIRECTORY>\.mcp.json` file

For both VS Code and Visual Studio, the configuration file uses the following server definition:

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

## More information

.NET MCP servers use the [ModelContextProtocol](https://www.nuget.org/packages/ModelContextProtocol) C# SDK. For more information about MCP:

- [Official Documentation](https://modelcontextprotocol.io/)
- [Protocol Specification](https://spec.modelcontextprotocol.io/)
- [GitHub Organization](https://github.com/modelcontextprotocol)
- [MCP C# SDK](https://modelcontextprotocol.github.io/csharp-sdk)
