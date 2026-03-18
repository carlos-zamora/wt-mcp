# Plan: Collapse Preview+Apply into Single-Call Elicitation

## Goal

Replace the two-call `Preview*/Apply*` pattern with single `Apply*` tools that use MCP
server-side **elicitation** to show the diff and get user confirmation inside one tool call.

---

## Background

Currently every write operation is split into two MCP tools:
- `PreviewSettingsChange` → returns a diff string → Claude shows it → user says "yes" → Claude calls `ApplySettingsChange`

The two-call pattern means two user prompts. With elicitation, the server computes the diff,
pauses mid-execution to show it to the user and ask "Apply?", then either writes or aborts —
all within a **single MCP call** and a **single user prompt**.

---

## SDK Change Required

Upgrade `ModelContextProtocol` from `0.7.0-preview.1` to `1.1.0` (elicitation was added in
spec v2025-06-18; full C# SDK support landed in v1.0).

**File:** `wt-mcp.csproj`
```xml
<!-- before -->
<PackageReference Include="ModelContextProtocol" Version="0.7.0-preview.1" />
<!-- after -->
<PackageReference Include="ModelContextProtocol" Version="1.1.0" />
```

---

## Pairs to Merge (6 total)

| File | Remove (Preview tool) | Merge into (Apply tool) |
|---|---|---|
| `SettingsTools.cs` | `PreviewSettingsChange` | `ApplySettingsChange` |
| `OhMyPoshTools.cs` | `PreviewOhMyPoshConfigChange` | `ApplyOhMyPoshConfigChange` |
| `FragmentTools.cs` | `PreviewCreateFragment` | `CreateFragment` |
| `FragmentTools.cs` | `PreviewUpdateFragment` | `UpdateFragment` |
| `SnippetTools.cs` | `PreviewCreateWtJson` | `CreateWtJson` |
| `SnippetTools.cs` | `PreviewUpdateWtJson` | `UpdateWtJson` |

---

## Elicitation Pattern

Each merged Apply tool gets two new parameters injected by the SDK:
- `IMcpServer server` — used to call `ElicitAsync`
- `CancellationToken cancellationToken`

The method becomes `async Task<string>`.

A shared private record handles the boolean response:
```csharp
private record ConfirmApply
{
    public bool Confirm { get; init; }
}
```

### Elicitation call (all 6 tools follow this pattern)

```csharp
// 1. Compute diff (same as the old Preview tool)
var diff = ...; // e.g. SettingsHelper.UnifiedDiff(before, patched, label)

// 2. Elicit confirmation — shows diff + yes/no form to user
if (server.ClientCapabilities?.Elicitation is not null)
{
    var elicit = await server.ElicitAsync<ConfirmApply>(
        $"Review the change below and confirm whether to apply it.\n\n```diff\n{diff}\n```",
        cancellationToken: cancellationToken);

    if (!elicit.IsAccepted || elicit.Content?.Confirm != true)
        return "Change cancelled — nothing was written.";
}
// else: client doesn't support elicitation, apply directly (graceful fallback)

// 3. Write
...
return $"Applied. Written to: {path}";
```

### Fallback when client doesn't support elicitation

If `server.ClientCapabilities?.Elicitation` is null, apply directly and include the diff in
the success response so the user still sees what changed.

---

## Tool Description Updates

Remove all cross-references to the deleted Preview tools:
- "Always call PreviewSettingsChange first …" → removed
- "After showing the diff, call this tool immediately …" → removed
- New description: "Applies a JSON Patch to … Shows a diff for confirmation before writing."

---

## Files to Change

1. **`wt-mcp.csproj`** — bump SDK version
2. **`Tools/SettingsTools.cs`** — merge `PreviewSettingsChange` → `ApplySettingsChange`
3. **`Tools/OhMyPoshTools.cs`** — merge `PreviewOhMyPoshConfigChange` → `ApplyOhMyPoshConfigChange`
4. **`Tools/FragmentTools.cs`** — merge `PreviewCreateFragment` → `CreateFragment`, `PreviewUpdateFragment` → `UpdateFragment`
5. **`Tools/SnippetTools.cs`** — merge `PreviewCreateWtJson` → `CreateWtJson`, `PreviewUpdateWtJson` → `UpdateWtJson`

No new files. No changes to `Helpers/`, `Program.cs`, or `.mcp/server.json`.

---

## Risks / Notes

- **Client support:** Elicitation requires the MCP client to declare `elicitation` in its
  capabilities. Claude Code does support this. The graceful fallback ensures non-supporting
  clients still work (apply-without-preview).
- **`AddSnippet` in SnippetTools.cs** already embeds a preview inline (lines ~349, ~415) and
  doesn't have a separate Preview tool — leave it unchanged.
- **`SetOhMyPoshTheme` in OhMyPoshTools.cs** runs PowerShell, no patch/diff involved — leave
  it unchanged.
- **`DeleteFragment`** has no preview tool currently — leave it unchanged.
