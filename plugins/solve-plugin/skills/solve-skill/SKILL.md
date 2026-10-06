---
name: solve-skill
description: >
  Using Solve CRM MCP/Connector searches for records (contacts, companies, project blogs, tickets) in Solve CRM based on complex user-defined criteria. Builds the following reports: activities, deals, follow-ups, next actions, time tracking. Loads, adds, updates and deletes records and activities. You **MUST** use this skill when asked to find records, build reports, change records or activities in Solve CRM or when user simply mentions Solve. Uses Solve CRM MCP tools to perform all these tasks.
---

# Solve CRM MCP Skill

This skill ensures you use the Solve CRM MCP tools correctly every time. Violations of the rules
below will cause API errors, data corruption, or wrong results for the user.

## Invoke This Skill When

- User mentions Solve, Solve360, or Solve CRM
- User asks to find records based on criteria possibly consisting of multiple conditions with "and" and "or" between them
- User asks to create, modify or delete contacts/companies/project blogs/tickets or activities. Activity types include task, tasklist, note, followup, call/interaction, photolist, google doc, website, opportunity/deal, time record, comment, linked emails, xerowidget and zendeskwidget
- User asks to find, create, modify or delete something other than contacts, companies or project blogs because user may rename "project blog" to something specific to their business
- User asks to show activities, deals, follow-ups, next actions or time tracking based on some filtering criteria
- User asks to show fields or category tags for a type of records or activities

## Critical Rules
- **NEVER** guess anything
- **NEVER** make assumptions
- **NEVER** say anything you are not sure about
- **ALWAYS** read the documentation if available for any tool you are going to use before using it
- **ALWAYS** follow all instructions in the documentation for any tool you use
- **ALWAYS** follow all instructions in this SKILL.md
Violating any of these rules will cause a bad user experience and is **STRICTLY PROHIBITED**.

---

## Step 1: Get User Timezone (Required for Most Tools)

Many tools require `user_timezone` as an IANA timezone string (e.g. `"America/Edmonton"`), eg when creating or updating records or activities with date/time fields, when searching for records with date/time criteria, or when building activities, deals, follow-ups or next actions reports. Always get the user's timezone before calling any of those tools.

- If you already know the timezone from earlier in the conversation, use it.
- If not, call `solve360_account` first. It returns `overridenzoneinfo` — use that value.
- If `solve360_account` doesn't return a timezone, ask the user directly before proceeding.

Never guess or assume a timezone. Passing the wrong timezone produces incorrect date formatting.

---

## Step 2: Check for Renamed Record Types

Solve CRM lets organizations rename "project blog" to something else (e.g. "Project", "Case", "Room").
If the user asks about something that's **not** a contact, company, or ticket, call
`solve360_account` first to check if blogs have been renamed (`bloglabel` field). Use that label
instead of "blog" or "project blog" when communicating with the user.

---

## Rule for getting information about the user currently logged in Solve CRM

When you need to get the ID of the user currently logged in Solve CRM, use `solve360_account` tool. It returns `userid` — use that value. ID lets you retrieve the information about the user's account using `solve360_ownership`.

---

## Rules for Searching (solve360_search)

Before constructing any search, you **MUST** read:
- `solve360_docs("search_args")` — full argument reference for solve360_search
- `solve360_docs("ucf")` — the only allowed formats for the `ucf` argument

**Never construct a `ucf` argument without reading `solve360_docs("ucf")` first.** The ucf format is strict and non-obvious — guessing will produce invalid queries.

**Never run multiple solve360_search calls** to handle multi-criteria queries. Always combine all criteria into a single call using the `ucf` argument.

The instructions from `solve360_docs("search_args")` and `solve360_docs("ucf")` are **CRITICAL**. Violating any of these rules is **STRICTLY PROHIBITED** and will cause wrong results for the user. Before using `solve360_search`, read these instructions carefully and make sure all arguments, including `ucf`, are correctly formatted.

**CRITICAL**: when searching for records having or not having certain activities, activity types or activity values **NEVER USE** any of the following tools:
`solve360_activities`, `solve360_deals`, `solve360_followups`, `solve360_next_actions`, `solve360_timetracking`. These tools are **NOT** for record searching.
But **ALWAYS** use `solve360_batch` tool to load the records found by
`solve360_search` tool for all non-activity-related criteria and filter by
the criteria related to activities then.
Never cross-reference records with these tools' results. Violation of this rule
is **STRICTLY PROHIBITED** and will result in incorrect results.

Calling other tools:
- Call `solve360_ownership` first to resolve user/group names to IDs if criteria involve them
- Call `solve360_batch` first to resolve tag names to IDs if criteria involve tags
- Call `solve360_fields` to resolve field names to IDs if criteria involve fields
- Call `solve360_zones` to resolve zone names to IDs if criteria involve zones
- If user wants to use a custom filter, call `solve360_account` to get both the required filter's json string, which you must use **as is** in `cf` argument, and the `is_collaborator` status. `is_collaborator` must be provided to solve360_search if `cf` is used.

---

## Rules for Fetching Records and Categories

**Never make separate API calls to fetch 2 or more records or categories.** Always use `solve360_batch` to batch them into one call. This is a strict server rule — violating it causes unnecessary load and is explicitly prohibited.

Good: `solve360_batch` with two endpoints in one call  
Bad: `solve360_batch` to load a record/categories then `solve360_batch` to load another record/categories

Use `solve360_batch` to:
- Fetch full record data (contacts, companies, project blogs, tickets) by known ID
- Fetch contact, company, project blog or ticket categories/tags

---

## Rules for Creating and Updating Records

You **MUST** call `solve360_docs("record")` to read critical instructions before using any record creation or update tool. This is a strict rule — violating it will cause wrong results for the user.

Before creating or updating any record, fetch this information if you need:

1. **Field names** — call `solve360_fields(record_type_id)` to get exact field names.
   Never guess or invent field names.
2. **Category IDs** — call `solve360_batch` to resolve tag names to IDs.
   Never guess or invent category IDs.
3. **User/Group IDs** — call `solve360_ownership` to resolve names to IDs.
   Never guess user or group IDs.

**All field values must be strings**, even numbers and complex structures like addresses.

**Date+time fields must be converted to UTC** in the format `"YYYY-MM-DD HH:MM:SS"` before passing them to any tool.

**Ownership rules**:
- If you're creating a record and `ownership` is not specified by the user,
you **MUST** ask the user to confirm they want a private record before proceeding, abort if user declines. Do not silently create private records.
- If `ownership` is set to a group ID, current user **MUST** have write access to it, otherwise don't attempt to create/edit the record but show an error. Call `solve360_ownership` if you need to check the user's permissions.
- Pass `"[ID of the currently logged-in Solve CRM user]"` in `ownership` to make a record private. If you don't know the ID of the currently logged-in user, call `solve360_account` to get it.
- Pass a `[group ID]` in `ownership` so that all members of that group can read and edit record — this is the standard pattern for team-shared records. When set to a **user ID**, only that user (and admins) can see the record.


**Related records ownership**: Any related items you attach must be owned by the same
user/group as the record being created/updated. Warn user that unqualified related items will be ignored and
ask the user to confirm/cancel/amend the operation.

**Never make multiple update calls for the same record.** Collect all changes and pass them
in a single `solve360_update_record` call.

**Arguments rules**:
- fields named `company` and `relatedto` **MUST NEVER** be set in `fields` argument.
- tickets are **NOT ALLOWED** as related items in `related_items` argument.

---

## Rules for Creating and Updating Activities

Before creating or updating any activity, you **must** call `solve360_docs("activity")` to load required instructions. It is **STRICTLY PROHIBITED** to call `solve360_create_activity` or `solve360_update_activity` without loading these instructions first.

**Activity hierarchies rules**: Only certain activity types can be added to certain parent records or activities. You **MUST** refer to the `Supported Activity Hierarchies` section in `solve360_docs("activity")` to make sure you are adding to a valid parent. Violating these rules will cause errors.
If user asks to create an activity but you cannot find an appropriate parent for it, ask user if they want you to create one or cancel.

**Never make multiple update calls for the same activity.** Collect all changes and pass them in a single `solve360_update_activity` call.

---

## Rules for Attaching Files and Photos (File and Photo Activities)

Attaching a file takes two tool calls plus one HTTP request made outside MCP. Each step must succeed before the next one starts. Any deviation causes an error or leaves a broken file entry. A photo follows exactly the same steps, with `activity_type` set to `"photo"`.

Before you start:
- Read `solve360_docs("activity")`, as for any activity.
- Pick a valid parent. Files can **only** be added to a record (`Record > File`) or to a task (`Record > Task List > Task > File`). **Never** add a file to any other activity.
- Photos can **only** be added to a Photo List (`Record > Photo List > Photo`). **Never** add a photo to any other record or activity.
- You need the file at a local path you can read, a shell that can run `curl`, and network access. If any of these is missing, tell the user you cannot upload the file and stop. **Never** create the File activity in that case.

Handle **one file at a time**. Finish every step for one file before starting the next. **Never** create several File activities first and upload afterwards: the upload URL is short-lived.

1. **Validate the file.** Skip it and tell the user why if either check fails:
   - A file must be a PDF: the name ends in `.pdf` (any case). A photo must be a GIF, JPEG or PNG image: the name ends in `.gif`, `.jpg`, `.jpeg` or `.png` (any case)
   - It must be 20 MB or smaller. Get the exact size in bytes as digits only, with no spaces. **Never** estimate it. Example for Linux/macOS: `wc -c < '<path>' | tr -d ' '`
2. **Create the File or Photo activity.** Call `solve360_create_activity` with:
   - `activity_type`: `"file"` or `"photo"`
   - `parent_id`: ID of the record or task for a file, ID of the photo list for a photo
   - `record_type_id`: type of the record the file belongs to (for a task, the record that holds the task list; for a photo, the record that holds the photo list)
   - `data`: `{"filename": "<name with extension>", "filesize": "<exact byte count>", "caption": "<title given by user, otherwise filename>"}`. All values are strings.

   From the response, keep `id`, `data.action` and `data.data[0]`.
3. **Upload the file.** Send exactly **one** multipart/form-data POST to `data.action`:
   - Add every key/value pair from `data.data[0]` as a form field. Copy each value character for character and keep the same order. Do not decode, reformat, drop or add fields.
   - Add the file last, in the field named `file`.

   With `curl`, use `--form-string` for every `data.data[0]` field. **Never** use `-F` for these, because `-F` treats values that start with `@` or `<` as file references:
   ```
   curl -sS -w '\nHTTP %{http_code}\n' '<data.action>' \
     --form-string '<key1>=<value1>' \
     --form-string '<key2>=<value2>' \
     -F 'file=@"<path>";type=<MIME type>'
   ```
   Use MIME type `application/pdf` for a PDF, `image/gif` for GIF, `image/jpeg` for JPEG, `image/png` for PNG.
   Put each argument in single quotes. If a value contains `'`, write it as `'\''`.
4. **Check the result.** Success means HTTP 200 with an empty body. Any other status, or any body text, is a failure: show the user the status and body, then stop. **Do not** retry and **do not** call `solve360_commit_activity`.
5. **Commit the file.** Call `solve360_commit_activity` with the same `record_type_id` and `activity_type` as in step 2 and `activity_id` set to `id` from the step 2 response. The file or photo is not attached until this call succeeds. If it fails, show the error.

Tell the user a file or photo is attached only after step 5 succeeds.

---

## Rules for Deleting Records and Activities

You are **never allowed** to call `solve360_delete_record` or `solve360_delete_activity` in the same turn as the user's deletion request. Always ask the user to confirm first, then delete only after they explicitly confirm in the chat.

---

## Rules for Listing Activities (solve360_activities)

The `types` argument **MUST NOT be empty**. If you do not want to filter by activity type, pass all activity type IDs as a comma-separated string to include everything.

Activity type IDs supported:
`3=Note, 4=Event, 6=Follow-up, 14=Task, 23=File, 24=Photo, 32=Opportunity, 73=Interaction/Call Log, 88=Scheduled Email`

If `categories` argument is not empty, `itemtypes` argument **MUST be one of**: 1 (contact), 2 (projectblog), or 40 (company), to specify which record types to filter by.

---

## Rules for Reading Files Stored in Solve CRM

This section applies **only if you are Claude** (the AI model made by Anthropic) **running in the Claude app** (Claude Code, claude.ai on the web, Claude desktop, or Cowork). If you are a different model, skip this section.

File and Photo activities returned by Solve CRM MCP tools link to files hosted on `*.amazonaws.com`.

**Always** download the file in the code execution sandbox (with `curl`, for example), then open the downloaded copy there — view images directly, and never describe an image from its filename or metadata. **Never** use the WebFetch / `web_fetch` tool on a Solve CRM file link — it cannot retrieve Solve files or images. Do not try it even once to check.

Reading a file this way requires `*.amazonaws.com` on the user's Claude domain allowlist.

If the download never reaches the host — DNS failure, connection refused, request blocked by the proxy — the domain is not allowed. You **MUST**:

1. Stop trying to read the file. Do not retry the link and do not try other ways to get its contents.
2. **Never** guess or describe the file's contents.
3. Tell the user you cannot read the file yet and show them these instructions:

   > To let Claude read files stored in Solve CRM:
   > 1. Go to **Settings > Capabilities > Domain allowlist** and add `*.amazonaws.com`.
   >    On a Team or Enterprise plan, the change takes effect only if an organization owner makes it, in **Organization settings > Capabilities > Domain allowlist**.
   > 2. Open a new chat. The change does not apply to the current one.
   >
   > Then ask again, and Claude will be able to read files stored in Solve CRM.

If the host does respond but returns an HTTP error (403, 404, expired link), the allowlist is not the problem. Report that error to the user as it is and do **not** show the instructions above.

---

## Rules for Displaying the Calendar

When displaying results from `solve360_calendar`, you **must** render a visual calendar grid (rows = weeks, columns = Sun–Sat) with item titles placed inside their corresponding date cells. A markdown table is the correct markup for drawing that grid. Do **not** fall back to a flat list, a chronological event table, or any other format unless the user explicitly requests it.
Show it directly in the chat — **do not** create HTML files or attachments.

When filtering by multiple users, you **must** combine all user IDs into a single `filter` argument. Never make separate calendar calls per user.

---

## Rules for Displaying Time Tracking

When displaying results from `solve360_timetracking`, always show the **total time recorded** and a **breakdown of billable vs non-billable hours**.

---

## Rules for Displaying Results

**Never display internal IDs** (record IDs, category IDs, user IDs, field IDs) in responses unless the user explicitly asks to see them. Show labels, and human-readable values.
**Never show any IDs** even if you are explaining what you are doing to the user. 

---
