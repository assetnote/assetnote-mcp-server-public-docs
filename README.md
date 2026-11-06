# Assetnote MCP Server (Beta)

The Assetnote MCP server connects an AI assistant (Claude, ChatGPT, Codex and other MCP clients) to your
Assetnote instance, so you can ask about your attack surface in plain language: which assets run a
technology, what is exposed, which asset group is worst, and more. If you allow it, the assistant can
also make changes, such as turning monitoring on or importing hosts, always previewing them first.

Assetnote hosts the server, so there is nothing to run yourself: you add Assetnote from your assistant's
directory and sign in.

## Contents

- [Connecting](#connecting)
  - [Claude](#claude)
  - [ChatGPT and Codex](#chatgpt-and-codex)
  - [Adding Assetnote by URL](#adding-assetnote-by-url)
  - [Signing in](#signing-in)
  - [Staying connected and disconnecting](#staying-connected-and-disconnecting)
- [What you can ask](#what-you-can-ask)
- [How results work](#how-results-work)
- [Making changes](#making-changes)
- [Available tools](#available-tools)
- [Troubleshooting](#troubleshooting)
- [Security and your data](#security-and-your-data)

## Connecting

Assetnote is listed in Claude's connectors directory and in the plugin directory that ChatGPT and Codex
share. Add it from there and [sign in](#signing-in); there is nothing to configure. Clients without a
directory can [add it by URL](#adding-assetnote-by-url).

### Claude

This works in claude.ai, the Claude desktop and mobile apps, Cowork and Claude Code.

1. Open **Customize → Connectors**. In the desktop app, select **Customize** in the sidebar, then
   **Connectors**.
2. Select **Discover**, search for `Assetnote` and open it.
3. Select **Connect to Claude** and [sign in](#signing-in).
4. To use it in a chat, select **+** in the message box, then **Connectors**, and turn Assetnote on.

On Team and Enterprise plans, an Owner adds Assetnote for the organization first: on its directory page
the button reads **Connect for your team** (or use **Organization settings → Connectors**). Each member
then selects **Connect to Claude** and signs in. On Team plans, members who can't add connectors see
**Request** instead, which asks the Owners.

Under **Customize → Connectors → Assetnote → Tool permissions**, each tool can be set to Always allow,
Needs approval or Blocked (on Team and Enterprise, Owners set this for the whole organization).

**Claude Code** uses the connectors you added in claude.ai when you sign in to it with your Claude
account. Run `/mcp` in Claude Code to check that Assetnote is connected.

### ChatGPT and Codex

1. In ChatGPT on the web or in the desktop app, open **Plugins**, search for `Assetnote` and open it.
2. Select the plus button to install it.
3. Connect when ChatGPT asks, during install or the first time you use Assetnote, and
   [sign in](#signing-in).
4. To use it, ask about your attack surface, or type `@` in the prompt box and select Assetnote.

On Business, Enterprise and Edu workspaces, if **Connect** is greyed out or says **Disabled by admin**,
ask a workspace admin to enable Assetnote for your workspace and role.

ChatGPT asks you to approve each tool that changes something (the [write tools](#write-tools) and
`export_start`) before it runs, and can remember your answer for the rest of the conversation.

**Codex** shares the plugin directory: install Assetnote from **Plugins** in the ChatGPT desktop app, or
run `/plugins` in the Codex CLI, then start a new session. The Codex IDE extension doesn't support
plugins; [add Assetnote by URL](#adding-assetnote-by-url) there.

Uninstalling the plugin leaves the connection signed in; disconnect it in ChatGPT's settings as well, or
[revoke it](#staying-connected-and-disconnecting).

### Adding Assetnote by URL

For clients without a directory, or an organization that doesn't use one:

```
URL:   https://mcp.assetnotecloud.com/mcp
Auth:  OAuth (your client starts the sign-in for you)
```

There is no API key to put in a configuration file: the credential is chosen when you
[sign in](#signing-in).

- **Claude Code** without a Claude account:

  ```sh
  claude mcp add --transport http --scope user assetnote https://mcp.assetnotecloud.com/mcp
  ```

  Then run `/mcp`, select `assetnote` and authenticate.
- **Codex** (CLI, IDE extension and the ChatGPT desktop app share one configuration):

  ```sh
  codex mcp add assetnote --url https://mcp.assetnotecloud.com/mcp
  codex mcp login assetnote
  ```

  Or add it to `~/.codex/config.toml`, then run `codex mcp login assetnote`:

  ```toml
  [mcp_servers.assetnote]
  url = "https://mcp.assetnotecloud.com/mcp"
  default_tools_approval_mode = "writes"
  ```

  `default_tools_approval_mode = "writes"` makes Codex ask before every tool that isn't marked read-only:
  the [write tools](#write-tools) and `export_start`.
- **Claude, as a custom connector:** **Customize → Connectors → + Add → Add custom connector**, with the
  name `Assetnote` and the URL above.
- **ChatGPT, as a custom MCP server** (Business, Enterprise and Edu admins, or Pro with read-only tools):
  **Plugins → + → Add custom MCP server**, with the URL above and **OAuth**.
- **VS Code (GitHub Copilot):** run **MCP: Add Server** from the Command Palette, choose **HTTP**, and
  enter the URL.
- **Cursor:** add the server to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

  ```json
  { "mcpServers": { "assetnote": { "url": "https://mcp.assetnotecloud.com/mcp" } } }
  ```
- **MCP Inspector**, to try the tools directly:

  ```sh
  npx @modelcontextprotocol/inspector --server-url https://mcp.assetnotecloud.com/mcp --transport http
  ```

### Signing in

1. Your client opens the Assetnote sign-in page in your browser.
2. Under **Assetnote instance**, enter the first part of your Assetnote address: `acme` for
   `https://acme.assetnotecloud.com`.
3. Choose what the client may do. Reading data and creating export files is always included. Turn on
   **Change data** (`assetnote:write`) only if you want the assistant to be able to make changes; you can
   add it later.
4. Choose **Continue to Assetnote**. Your own Assetnote instance opens; sign in as usual, including SSO
   and MFA. It shows your email, role and instance.
5. Pick the credential the connection uses: your personal API key, a pasted API key or machine token
   (`anmt_…`), or, for admins, a new machine token. Then approve.

Your client is then connected. Choose **Deny** at either step to cancel.

The assistant sees exactly what that credential can see: its Assetnote role and asset-group scope apply
to every call, and connecting never grants more than the credential already has.

**To allow changes later,** reconnect and turn on **Change data**. Some clients offer this the first time
the assistant tries a change.

**Connecting to one instance only.** `https://mcp.assetnotecloud.com/i/{instance}/mcp` (for example
`https://mcp.assetnotecloud.com/i/acme/mcp`) works the same way, but the instance is fixed and the
connection can only hold that instance's credential. Use it if you work with one instance and want to
skip entering it.

### Staying connected and disconnecting

- A connection stays signed in for **30 days from sign-in**; your client refreshes access in the
  background. After 30 days, sign in again.
- This browser remembers your approval for a client for 30 days, so signing in again goes straight to
  your Assetnote instance, which still asks you to confirm.
- If the credential stops working (rotated, deleted, or its user deactivated), the client asks you to
  reconnect.

To disconnect, remove the connector in your client, or open `https://mcp.assetnotecloud.com/disconnect`
to see your connections and revoke one or all of them. Revoking a connection does not rotate the
Assetnote key or token itself; do that in Assetnote if it may have been exposed.

## What you can ask

Ask in plain language; the assistant picks the tools. For example:

- "Which of my assets run Apache, and on which versions?"
- "Show my unresolved critical exposures."
- "Which asset group is worst?"
- "How many assets do we have in the Payments asset group?"
- "Which assets have TLS certificates expiring in the next 30 days?"
- "Explain this exposure and show the HTTP response that matched: <exposure link or id>"
- "Export all our domains as CSV."
- With **Change data** allowed: "Turn off notifications for staging.example.com" or "Tag these three
  assets with `pci`". The assistant shows you a preview and asks before changing anything.

## How results work

Results are kept small so a conversation doesn't fill up with raw data.

- **Lists return one page**: 20 rows by default, up to 50, with the total for the whole query, a
  ready-made call for the next page and a link to each row in the Assetnote UI. Paging stops at row 2,500.
- **"How many" questions** are answered from a list's total, reading one row.
- **"All of it" questions** are answered with an export. The assistant previews the export and tells you
  the row count, and should start it only after you agree. The file stays on your instance for 14 days:
  you download it from the Assetnote UI through the link the assistant gives you. The server never
  emails it. The assistant can check an export's progress for 24 hours after starting it.
- **Evidence** (HTTP requests, responses and patches) can be several MB, so it is returned in windows of
  up to 24,000 characters.
- **Exposures** default to unresolved findings and leave out hygiene findings, matching Assetnote's own
  statistics. Ask for another status, or for hygiene findings, to change that.

**Filters and sorts.** The assistant builds filters from your question. Under the hood, the list and
export tools take filters as `{field, op, value}` objects (with `rel` for a related record, such as
`preflightMetadata` or `tags`; each tool lists the related records it accepts) and sorts as `{field, dir}` with `dir` `ASC` or `DESC`. A
field is one name, on the record or on the related record in `rel`. Operators are
`EQ`, `NOT`, `LT`, `LTE`, `GT`, `GTE`, `CONTAINS` and `DOES_NOT_CONTAIN`. Dates take an ISO datetime or a
time relative to now, such as `-7d`, `-24h` or `+30d`. If Assetnote rejects a filter, the error names the
clause and lists the fields that can be filtered.

## Making changes

The write tools work only on a connection with **Change data** (`assetnote:write`), and only within what
your credential's Assetnote role allows.

- **Preview first.** Every write tool previews by default: it looks up each record and shows its current
  state and whether it would change, and nothing is changed. The change is made only by a second call with
  `confirm: true`. Ask the assistant to show you the preview before it confirms.
- **Explicit records only.** Changes take a list of ids (up to 500 assets, or one asset group), never a
  filter, so a change can't spread to records you didn't name. Records already in the requested state are
  not sent again. The exception is importing hosts: a host is skipped only once it is an asset, so
  importing the same hosts again before discovery has created them, or for names that don't resolve,
  sends them to discovery again.
- **Some changes start scanning.** Assetnote starts discovering or scanning targets when you:
  - turn monitoring on;
  - import hosts;
  - import ranges or ASNs into a monitored group;
  - promote a subdomain into a monitored group.

  The tool descriptions and previews say so. Import only targets your organisation is authorized to
  scan. Turning monitoring on also counts against your instance's daily monitoring-change budget; the
  preview shows what is left.
- **Some changes stop monitoring.** Adding a blacklist pattern to a group stops monitoring the group's
  assets whose host name or IP matches it. Removing the pattern does not turn their monitoring back on.
  The preview estimates how many monitored assets each new pattern matches.
- **Background changes.** Monitoring and notification changes to assets, and subdomain promotions, run as
  background tasks on your instance; `task_status` reports whether they have finished. Every other
  change is finished when the call returns.
- **If a change times out.** A change that times out or meets a server error may still have been
  applied. The error says so; check the records before repeating it.
- **Your client may ask too.** ChatGPT asks before each write tool, Codex does with the
  [approval setting](#adding-assetnote-by-url), and Claude follows the [tool permissions](#claude) you
  set. The `confirm: true` in a call is not your approval: approve only changes you have seen
  previewed.

## Available tools

All tool names start with `assetnote_`.

### Asset group tools

| Tool | What it does |
|---|---|
| `list_asset_groups` | A page of asset groups with asset, monitored, technology and service counts and the exposure rating. |
| `get_asset_group` | One group: counts, exposure rating, recon settings, notification settings and targets (email targets by id, Slack targets by channel); optionally its blacklisted hosts and port scan profile. |

### Asset tools

| Tool | What it does |
|---|---|
| `list_assets` | A page of assets, scoped to a technology (and version), a technology category (ids from `find_technology` or `list_technologies`), a service or TLS certificate (ids from `get_asset`'s services and certificates), an asset group or verified assets, and narrowed with filters. |
| `get_asset` | One asset; optionally its technologies, services, certificates, DNS records, tags, screenshots and crawled URLs. |

### Domain tools

| Tool | What it does |
|---|---|
| `list_domains` | A page of source domains with asset and exposure counts, monitoring state and importance; optionally for one asset group. |

### Exposure tools

| Tool | What it does |
|---|---|
| `list_exposures` | A page of vulnerabilities or indicators by triage status (unresolved, triaged, resolved, ignored or all), signature class, asset group or asset. |
| `get_exposure` | One exposure with the signature's description, recommendations and references and recent timeline events (what happened and when, without who or their comments); optionally CVE intelligence (CVSS, EPSS, CISA KEV, public exploits) and other affected assets. |
| `get_exposure_evidence` | The stored HTTP request or response, or code patch, for one exposure. |

### Technology tools

| Tool | What it does |
|---|---|
| `find_technology` | Technologies whose name contains some text, with their ids, asset counts, top versions and categories. |
| `list_technologies` | A page of detected technologies with asset and version counts and their categories; optionally one category. |

### Exports

| Tool | What it does |
|---|---|
| `export_preview` | The row count and scope of an export, without starting it. |
| `export_start` | Starts a CSV or JSON export of assets, exposures, indicators, technologies, domains or asset groups, with the same scope and filters as the list tools. Most clients ask you to approve it first. |
| `export_status` | Whether an export has finished, with a link to download it in the Assetnote UI. |

### Write tools

These need a connection with **Change data** (`assetnote:write`). The roles listed are the customer roles
this server lets make each change; Assetnote can still refuse a change your role doesn't allow.

| Tool | What it does | Roles |
|---|---|---|
| `set_assets_monitored` | Turns monitoring on or off for up to 500 assets. Turning monitoring on can start scanning. | CLIENT_TECH, CLIENT_OPERATOR, CLIENT_ADMIN |
| `set_assets_notifications` | Turns notifications on or off for up to 500 assets. | CLIENT_TECH, CLIENT_OPERATOR, CLIENT_ADMIN |
| `set_assets_sensitive` | Marks up to 500 assets as sensitive or not. | CLIENT_TECH, CLIENT_ADMIN |
| `verify_assets` | Verifies up to 500 assets. | CLIENT_TECH, CLIENT_OPERATOR, CLIENT_ADMIN |
| `add_asset_tags` | Adds tags, by name, to up to 500 verified assets. Current tags are kept. The tags must already exist; create them in the Assetnote UI. | CLIENT_TECH, CLIENT_OPERATOR, CLIENT_ADMIN |
| `clear_asset_tags` | Removes every tag from up to 500 assets. | CLIENT_TECH, CLIENT_OPERATOR, CLIENT_ADMIN |
| `whitelist_asset_risk` | Marks assets' Mass VHosting risk as not risky. It can't be undone. | CLIENT_TECH, CLIENT_ADMIN |
| `update_asset_group` | Changes one group's recon switches, notification settings and blacklist (up to 25 patterns added or removed per call). It can also unlink the group's email and Slack notification targets, or link one that another group already uses; connect new integrations in the Assetnote UI. Adding a blacklist pattern stops monitoring the assets it matches; the preview estimates how many. | CLIENT_TECH, CLIENT_ADMIN |
| `promote_subdomain` | Makes a subdomain a source domain of a group and moves the assets under it there, including assets in groups your credential can't see. In a monitored group this starts discovery and scanning. Not available to credentials scoped to some asset groups. | CLIENT_TECH, CLIENT_ADMIN |
| `import_hosts` | Adds up to 500 host names to a group and sends them to discovery, which starts scanning them. Assets appear only for names that resolve, after a few minutes or more. A host under a root domain that is already a source domain in another group is likely filed in that group; the preview names each root domain's group. | CLIENT_TECH, CLIENT_ADMIN |
| `import_domains` | Adds up to 500 domains to a group as unmonitored source domains. Nothing is scanned. | CLIENT_TECH, CLIENT_ADMIN |
| `import_ranges` | Adds up to 100 IPv4 ranges or addresses to a group. In a monitored group they are scanned. Ranges wider than /16 or in reserved space (such as loopback, multicast and the documentation blocks) are refused, and so is a range that would absorb one already in Assetnote; a range inside an existing one is skipped. Private (RFC 1918) ranges are allowed. | CLIENT_TECH, CLIENT_ADMIN |
| `import_asns` | Adds up to 10 ASNs to a group, each expanded into every IPv4 block Assetnote's ASN data lists for it (possibly hundreds). Import only ASNs your organisation holds, not your hosting, cloud or CDN provider's. In a monitored group they are scanned. Reserved ASNs (documentation, private and AS_TRANS 23456) are refused. | CLIENT_TECH, CLIENT_ADMIN |
| `task_status` | Whether a background change has finished, and how many records it changed. | Any |

## Troubleshooting

| You see | What it means |
|---|---|
| "This server cannot connect to that instance" (at sign-in) | The instance label is mistyped, or your instance isn't in the beta yet. Enter the first part of your Assetnote address; contact Assetnote to join the beta. |
| "This page expired" (at sign-in) | The sign-in was left open for over ten minutes, already used, or opened in another browser. Start again from your client. |
| "The linked Assetnote credential is no longer valid. Reconnect this account." | The credential was rotated, deleted, or its user deactivated. Reconnect from your client. |
| A role limit (`ROLE_LIMIT`) | Your credential's Assetnote role doesn't offer that data. The connection is fine. |
| `insufficient_scope` | The connection was made without **Change data**. Reconnect and turn it on, or let your client step up. |
| "This credential's current Assetnote role does not permit this change" | Your role can't make that change; ask an admin, or use a credential with a higher role. |
| "Assetnote requires this user to accept an updated agreement" | Sign in to the Assetnote UI, accept the agreement, then retry. |
| "Rate limit exceeded for this credential" | More than 60 calls in a minute. Wait a moment and retry. |
| "Export quota exceeded" | More than 10 exports started in a minute, for your credential or your instance. Retry in a minute. |
| A filter is rejected | The error names the clauses Assetnote rejected and lists the fields that can be filtered. |
| Rows past 2,500 are refused | Ask how many, or for an export of the rows. |
| Large evidence is refused | Responses over 8 MiB from Assetnote can't be returned; view the evidence in the Assetnote UI. |
| An export's link is no longer available from the assistant | The assistant can check an export for 24 hours after starting it. The file itself stays in the Assetnote UI for 14 days. |

## Security and your data

- The server talks only to your instance's Assetnote API, with the credential you chose. The credential
  is stored encrypted with your connection and is never shown to the assistant.
- The assistant can see only what your credential's role and asset-group scope allow.
- Results the assistant reads become part of your conversation, so your AI provider receives them as it
  does any other conversation content. Choose a client and plan that meet your organisation's data
  policies.
- Evidence, page titles and other stored content can contain text written by scanned targets or
  other users. The server marks it as data, not instructions, and the assistant should never act on it.
- Exports are created on your instance and downloaded from its UI by a signed-in user; the server never
  downloads or emails them.
