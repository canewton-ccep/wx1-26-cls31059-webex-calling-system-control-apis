# Wx1 '26-CLS31059-Webex Calling System Control APIs

A [Bruno](https://www.usebruno.com/) API collection covering OAuth, Calling/CDR, Provisioning,
Phone Control, Reports, and Webhooks against the Webex APIs.

> **Session credit:** This collection is the companion artefact to session **CLS-31059**,
> delivered by **Carl Newton** at **WebexOne 2026**.

> **Note:** The parameter/variable descriptions and per-request documentation throughout this
> collection were written by an AI assistant (Claude) at the request of the repository owner, to
> accompany the WebexOne 2026 "Webex Calling API Overview" presentation (CLS-31059). Please verify
> against the linked developer.webex.com pages before relying on any specific detail.

## What's inside

| Folder | Covers |
|---|---|
| `CDR` | Detailed Call History and Live Stream Detailed Call History (rolling-window requests with auto-computed timestamps) |
| `Hardware Inventory` | Listing/finding devices by user, workspace, or location; Hot Desking sessions |
| `Phone Control` | Personal-line Call Controls (dial/hang up) and workspace-device xAPI commands (PhoneOS/RoomOS) |
| `Provisioning` | People, Locations, Licenses, Phone Numbers, and end-to-end new-user provisioning |
| `Reports` | The four-step async Reports API flow (templates → create → status → download) |
| `Webhooks` | Creating, listing, and deleting webhooks, with worked filter/scope examples |

This collection ships with **no real credentials anywhere in its files**. Every OAuth field and
every bearer token is a reference to a Bruno **Environment variable** (`{{clientId}}`,
`{{clientSecret}}`, `{{callbackUrl}}`, `{{authorizationUrl}}`, etc.) instead of a literal value, so
the repo is safe to clone, fork, and share — you supply the real values once, locally, in a Bruno
Environment (Bruno's built-in per-user, git-safe credential store).

## Before you start — using your own registered Integration with Bruno

### Step 1 — Register your own Integration

1. Go to [developer.webex.com/my-apps](https://developer.webex.com/my-apps) → Create New App →
   Integration (see the "Register the Integration" steps in the companion deck).
2. Select whichever scopes you need and set a Redirect URI. For quick testing you can reuse
   Bruno/Postman's public redirect catcher, `https://oauth.pstmn.io/v1/callback` — or register your
   own if you're running a real callback listener.
3. Once saved, developer.webex.com shows you a ready-made **Authorization URL** (with your Client
   ID, redirect URI, and chosen scopes already baked in) alongside your **Client ID** and **Client
   Secret**. Copy all three — you'll paste them into your Environment in Step 2, never into this
   repository's files.

### Step 2 — Use the shipped Template Environment to hold your credentials

This collection ships with a ready-made `environments/Template.bru` — you don't need to add these
variables by hand.

1. In Bruno, open this collection, then click the environment selector in the top-right (usually
   reads "No Environment") and pick **Template** from the list.
2. Right-click **Template** in the environment list and choose **Clone** (or **Rename**) to make
   your own personal copy first — e.g. "My Org" — so the pristine template stays available for
   teammates who clone this repo after you.
3. Click **Configure** to open it for editing, and fill in the blank fields:

   | Variable | Secret? | Value |
   |---|---|---|
   | `authorizationUrl` | no | Paste the full Authorization URL exactly as shown on your Integration's page on developer.webex.com — it already contains your Client ID, redirect URI, and chosen scopes, so there's nothing to assemble by hand |
   | `clientId` | no | Your Integration's Client ID from Step 1 |
   | `callbackUrl` | no | Your registered Redirect URI — pre-filled with the default `https://oauth.pstmn.io/v1/callback`; change it if you registered your own, and make sure it exactly matches the redirect URI baked into `authorizationUrl` |
   | `clientSecret` | **yes** | Your Integration's Client Secret from Step 1 |
   | `cdrToken` | **yes** | A personal access token with the `spark-admin:calls_read` scope, for the CDR requests |
   | `personalAccessToken` | **yes** | A personal access token for the personal-line Call Controls requests |
   | `workspaceOrServiceAppToken` | **yes** | A user or Service App token with `spark:xapi_commands`, for the PhoneOS/RoomOS xAPI requests |

   The four `clientSecret`/`cdrToken`/`personalAccessToken`/`workspaceOrServiceAppToken` rows are
   already flagged as **secret** in the template (they ship in `environments/Template.bru` as a
   `vars:secret [ ... ]` list with no values at all) — Bruno will prompt you to enter each one in
   its own masked field, then store it in OS-level encrypted storage (Keychain/DPAPI, or AES-256 as
   a fallback) rather than writing it back into the `.bru` file. That's what makes it safe to
   commit the environment file itself even after you've filled it in.

4. Select your environment from the dropdown before running anything.

### Step 3 — How it's wired up

This collection's root `auth:oauth2` block already references `{{authorizationUrl}}`,
`{{clientId}}`, `{{clientSecret}}`, and `{{callbackUrl}}` in place of literal values — no scopes or
query-string assembly live in this repository at all; they only exist in whatever you paste into
`authorizationUrl`. Once your Environment is selected, open the collection's Auth tab and click
"Get Access Token" — Bruno will run the full Authorization Code flow using your Environment's
values and cache the resulting access/refresh token for you (`auto_fetch_token`/
`auto_refresh_token` are already on). None of this ever touches `collection.bru`, so the file stays
safe to commit and share.

## Collection variables (defined in `collection.bru`'s `vars:pre-request`)

- **locations** — Example site/location name, retained for any ad-hoc filtering you want to do
  against the CDR requests.
- **CDRToken** — References the Environment variable `{{cdrToken}}` (see Step 2 above). Required
  for the CDR feed/stream requests.
- **personaltoken** — References the Environment variable `{{personalAccessToken}}`. Belongs to
  the end user whose personal line you want to control (dial/hang up) via the Call Controls API.
- **deviceid** — Example `callingDeviceId` for a workspace/desk phone (a Cisco 9861), used by the
  "Get Device Members" request to show which lines/users are associated with that device.
- **personid** — Example Person ID (Agent User) used throughout the Provisioning and Hardware
  Inventory folders as the "known good" user to query.
- **workspacetoken** — References the Environment variable `{{workspaceOrServiceAppToken}}`, used
  by the PhoneOS/RoomOS xAPI requests to control workspace devices.
- **webexDeviceId** — Empty by default. Populate with a device's `webexDeviceId` (from List Devices)
  before running the PhoneOS Cloud API Dial/Hang Up requests.
- **callingDeviceId** — Example `callingDeviceId` for a personal desk phone (Cisco 9851), used by
  the WxC User Phone Dial example.
- **locationId** — Empty by default. Populate with a Location ID (from List Location) before
  running "Find devices by location".
- **workspaceId** — Empty by default; kept for ad-hoc use when you want to try a workspace ID of
  your own choosing against "Find Hardware for WorkspaceID".
- **reportid** — Example Report ID, used to chain the Reports folder's "Get Report Status"/"Get
  Report" requests after "Create Report" returns a real one.
- **workspaceid** — Example Workspace (Place) ID, used as a pre-populated worked example by "Get
  Devices For User".
- **webhookId** — Example Webhook ID used by "Delete Webhook". Replace with the ID of the webhook
  you actually want to remove (from "List Webhooks").
- **testUserEmail** — Example user email used as the filter for "Get People".
- **webhookTargetUrl** — Shared example destination (a webhook.site bin) used by the four
  webhook-creation requests in the Webhooks folder. Replace with your own receiving endpoint.

## Environment variables (shipped as a template — see `environments/Template.bru` and Step 2 above)

- **authorizationUrl** — The full Authorization URL copied verbatim from your Integration's page on
  developer.webex.com. Used directly as `auth:oauth2.authorization_url` — this collection never
  assembles or lists scopes itself; whatever scopes you selected when registering the Integration
  are already encoded in this one URL.
- **clientId** — Your registered Integration's Client ID. Used by `auth:oauth2.client_id` for the
  code→token exchange (separate from `authorizationUrl`, which already carries its own copy of this
  value).
- **clientSecret** *(secret)* — Your registered Integration's Client Secret. Used by
  `auth:oauth2.client_secret` for the code→token exchange only; never appears in any URL.
- **callbackUrl** — Your registered Redirect URI. Used by `auth:oauth2.callback_url`, which Bruno
  matches against the redirect it receives after you approve access — must exactly match the
  redirect URI baked into `authorizationUrl`.
- **cdrToken** *(secret)* — A personal access token with the `spark-admin:calls_read` scope.
  Referenced by the collection variable `CDRToken`, used by every CDR request.
- **personalAccessToken** *(secret)* — A personal access token belonging to the end user whose
  personal line you want to control. Referenced by the collection variable `personaltoken`.
- **workspaceOrServiceAppToken** *(secret)* — A user or Service App token with
  `spark:xapi_commands`. Referenced by the collection variable `workspacetoken`.

## Keeping this in git

`environments/Template.bru` is committed with this repository on purpose — every plain variable in
it is blank, and every secret variable is only ever listed by name in its `vars:secret [ ... ]`
array, never given a value in the file, so it's safe to ship as-is. This repo's `.gitignore` also
excludes every other `environments/*.bru` file by default, so cloning/renaming the template for
your own personal use won't accidentally get committed.

Once you clone/rename the template and fill in the secret fields through Bruno's UI, those real
values live in OS-level encrypted storage, not in the `.bru` file — so even your personal copy
stays safe to commit if you choose to.

## License

Licensed under the [MIT License](./LICENSE).
