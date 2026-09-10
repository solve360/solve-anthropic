# Solve CRM plugin for Claude

Work with your [Solve CRM](https://solve360.com) — contacts, companies, projects,
and tickets — directly from Claude, using natural language. Search and retrieve records,
create and update them, add and modify activities (tasks, notes, follow-ups, deals),
and pull calendar, deals, and time-tracking insights.

This plugin connects Claude to the Solve CRM remote MCP server and ships a skill that teaches Claude how to use its tools correctly —
building valid queries, resolving names to IDs,
and following Solve CRM's rules when changing records and activities.

---

## What's included

| Component | Name | Purpose |
|---|---|---|
| Remote MCP server | `solve-mcp` (`https://mcp.solve360.com`) | Exposes the Solve record, activity, and insight tools |
| Skill | `solve-skill` | Governs correct tool use across searching, reporting, and record/activity changes |

The skill is **model-invoked**:
Claude applies it automatically whenever your request involves Solve CRM — searching,
reporting, or changing records and activities. You don't need to call it explicitly.

Supported record types: **contacts**, **companies**, **projects** (internally `projectblogs`;
your Solve CRM admin may have renamed these to Properties, Jobs, Sites, Cases, etc.),
and **tickets**.

Activities attached to those records — tasks, notes, follow-ups, events,
opportunities/deals and call logs among others — can also be listed, created, updated,
and deleted.

---

## Requirements

- **A Solve CRM account.** Authentication uses your own Solve login via OAuth;
  the plugin can read, create, and modify records and activities in your live Solve data,
  scoped to what your account is permitted to do.
- **A Claude client that supports plugins** — Claude Code (CLI, IDE extension,
  or desktop app), or the Claude desktop app's Cowork mode.

---

## Installation

### Claude Code

Add the marketplace, then install the plugin:

```bash
/plugin marketplace add solve360/solve-anthropic
/plugin install solve-plugin@norada
```

`solve-plugin@norada` is `plugin-name@marketplace-name`.

Then turn on automatic updates,
so you receive new versions of the plugin without reinstalling:

1. Run `/plugin`.
2. Press **Tab** until you reach the **Marketplaces** tab.
3. Select `norada` and press Enter.
4. Choose **Enable auto-update**.

If there's no **Enable auto-update** entry in that menu, update Claude Code and try again —
older versions don't offer it. To update: run `claude update`,
or `brew upgrade claude-code` if you installed via Homebrew, then restart Claude Code.

Without this, Claude Code keeps the version you installed:
marketplaces added by hand don't auto-update by default.

**Updating manually.** To pull the latest version at any time,
whether or not auto-update is on, run:

```bash
/plugin install solve-plugin@norada
```

If the summary says `Run /reload-plugins to activate.`, run that command —
otherwise the new version loads the next time you start Claude Code.

Installing is not enough on its own —
the Solve CRM MCP server uses OAuth and needs a one-time sign-in:

1. Run `/mcp`.
2. Select `plugin:solve-plugin:solve-mcp` — it shows `needs authentication` — and press Enter.
3. In the menu that opens, choose **Authenticate**.
4. A browser window opens; sign in to Solve and approve access.
5. Return to Claude — you'll see `Authentication successful.
   Connected to plugin:solve-plugin:solve-mcp.` and the Solve CRM tools are ready.

If the browser doesn't open automatically, copy the URL shown and open it manually.
If the redirect fails after you sign in,
paste the full callback URL from your browser's address bar into the prompt that appears in Claude Code.

This is a one-time step. To sign out, run `/mcp`, select `plugin:solve-plugin:solve-mcp`,
then choose **Clear authentication**.

### Claude web/desktop (Cowork)

Add the marketplace:

1. Click the **+** to the left of the Chat/Cowork selector, then select **Add plugins**.
2. In the window that appears, click **+** again and choose **Add from a repository**.
3. Paste `https://github.com/solve360/solve-anthropic.git` into the form and click **Sync**.

Install the plugin:

4. The marketplace now appears — click it, then click **Install**.

Install and connect the MCP connector:

5. Click **Manage**, then open the **Connectors** tab. If you see **Connect**, click it and go to step 8. Otherwise, click **Install**.
6. Click **Add** in the dialog that appears.
7. A **Connect** button appears next to the solve360 connector — click it.
8. A browser window opens; sign in to your Solve360 account and authorize the connector.

The plugin is now ready to use in Cowork.

**Updating.** To move to a newer version of the plugin:

1. Click **Customize** in the sidebar, then open the **Plugins** tab.
2. Click **Solve360** — its page has an **Update** button.
3. If **Update** is enabled, click it and go to step 7.
4. If **Update** is disabled, click the **solve-anthropic** link on that page,
   then open the **Personal** tab.
5. Click the **...** icon next to **solve-anthropic** and choose **Check for updates**.
6. Return to the **Solve360** page.
   **Update** is now enabled if a new version is available — click it.
7. Restart Cowork. The new version number is shown after the relaunch.

### Local development / testing

From the marketplace root, load the plugin directly without installing:

```bash
claude --plugin-dir ./plugins/solve-plugin
```

The MCP server still needs authenticating —
follow the `/mcp` steps under [Claude Code](#claude-code) above.

---

## Usage

Just ask in natural language.
The skill handles translating your request into the correct Solve360 tool calls. Examples:

- "Find contacts named Aaron"
- "Show me companies tagged Client"
- "Search for tickets assigned to Mike"
- "List projects created this month"
- "Look up a contact by email john@example.com"
- "Which contacts are flagged and have overdue tasks?"
- "Show full details for that contact"
- "Create a contact for Jane Doe at Acme"
- "Update Acme's phone number"
- "Add a follow-up task on that ticket for next Tuesday"
- "What's on my calendar this week?"

The skill combines multiple criteria into a single search,
resolves names/tags to the IDs the API needs.

---

## How it works

Solve has a specific query model (search modes, filter modes,
and a combined `ucf` structure for multi‑criteria queries),
plus strict rules for writing records and activities. Left to guess,
an assistant will often issue several separate searches, malformed filters,
or invalid field values. The bundled `solve-crm` skill encodes the correct rules —
single combined calls, name→ID resolution, date/timezone handling,
ownership confirmation before creating private records,
and explicit confirmation before deletes — so Claude uses the API the way Solve expects.

---

## Support

Maintained by **Norada Corp**. Questions or issues:
[support@norada.com](mailto:support@norada.com) · [Solve MCP](https://solve360.com/api/mcp-server-for-ai-agents/)

---

## License

See [LICENSE](./LICENSE).
