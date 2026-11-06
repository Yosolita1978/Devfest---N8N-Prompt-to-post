# Troubleshooting

Every problem in this file actually happened while building the workflow on **October 27, 2026**. They're listed in the order you'd meet them.

**How to read an n8n error:** double-click the red node. The **OUTPUT** panel shows a short message, and **Error details → From HTTP Request** shows Google's full answer. Google's answer usually names the real cause.

## Quick table

| # | Where | Error or symptom | Cause | Fix |
|---|---|---|---|---|
| 1 | Google Cloud console | Search shows "Vertex AI (Agent Platform)", but no "Vertex AI API" | Vertex AI was renamed in the console | [Use the direct API link](#1-cant-find-the-vertex-ai-api) |
| 2 | Service account roles | There's no **Vertex AI User** role | The role was renamed | [Pick **Agent Platform User**](#2-there-is-no-vertex-ai-user-role) |
| 3 | Browser | Form page is blank, with a download icon in the address bar | Desktop Chrome doesn't display n8n form pages | [Use a phone](#the-form-page-is-blank-in-desktop-chrome) |
| 4 | Gemini node | `This API method requires billing to be enabled` | The project has no billing account | [Link billing](#4-billing-is-not-enabled) |
| 5 | Linking billing | Billing account has reached its project limit | Too many projects on one billing account | [Unlink an old project](#5-billing-account-project-limit) |
| 6 | Gemini node | Prompt says `Vibe: undefined` | Extra space in the form's **Field Name** | [Fix the field name](#6-vibe-undefined) |
| 7 | Gemini node | `404 … gemini-2.5-flash was not found` | Model retired on October 20, 2026 | [Use `gemini-3.8-flash`](#7-model-not-found-retired-model) |
| 8 | Gemini node | `404 … gemini-3.8-flash was not found` in `us-central1` | The model only runs on `global` | [Use the global URL](#8-model-not-found-wrong-location) |
| 9 | Image node | `404 … imagen-4.0-generate-001 was not found` | Imagen 4 is retired | [Use `gemini-3.1-flash-image`](#9-imagen-not-found) |
| 10 | Save to gallery | `Node does not have any credentials set` | No credential picked after import | [Pick a credential](#10-no-credentials-set) |
| 11 | Save to gallery | `Google Drive API has not been used in project …` | Drive API switched off | [Enable the Drive API](#11-drive-api-not-enabled) |
| 12 | Save to gallery | `Service Accounts do not have storage quota` | Drive node logs in as the robot | [Switch to OAuth2](#12-service-accounts-do-not-have-storage-quota) |
| 13 | Show the card | Run never finishes; the node waits forever | Form Ending page waits for the form screen | [Removed the node](#13-the-last-node-waits-forever) |
| 14 | Image node or Pick image data | `Cannot read properties of undefined` | The model returned no image (for example, a safety block) | [See below](#14-no-image-in-the-answer) |

---

## 1. Can't find the Vertex AI API

**Symptom:** searching "Vertex AI API" in the console only shows **Vertex AI (Agent Platform)** (a product page) and **Vertex Image Segmentation** (a model).

**Fix:** open the API page directly. Its technical name, `aiplatform.googleapis.com`, didn't change:
https://console.cloud.google.com/apis/library/aiplatform.googleapis.com

## 2. There is no "Vertex AI User" role

**Symptom:** searching `Vertex AI User` in **Select a role** shows only *Service Agent* roles.

**Cause:** the role was renamed during the Agent Platform rebrand.

**Fix:** search `Agent Platform` and pick **Agent Platform User**. It's the same role, `roles/aiplatform.user`.

**Don't pick:** *Vertex AI Service Agent*, *Custom Code Service Agent*, *Rapid Eval Service Agent* (those are for Google's own internal accounts), or *Provisioned Throughput Admin* (that's for buying reserved capacity).

## The form page is blank in desktop Chrome

**Symptom:** **Execute Workflow** opens a popup at `…/form-test/…` that stays empty, with a **download icon** in the address bar. A normal Chrome tab shows the same. The run stays "waiting" forever.

**What we checked:** fetching the same URL with `curl` returns a complete HTML form, with the title, all three fields, and the vibe options. **n8n is working.** The browser receives the page but doesn't display it.

**Likely cause:** something in the presenter's desktop Chrome, most likely an extension. The exact cause wasn't confirmed.

**Fix:** test the form **on a phone**. That's also what attendees use. The n8n editor and Google Drive work normally in the same Chrome.

**Related:** an n8n *Form Ending* page (shown after the workflow finishes) has the same problem. See [#13](#13-the-last-node-waits-forever).

## 4. Billing is not enabled

```
Forbidden - perhaps check your credentials?
This API method requires billing to be enabled. Please enable billing on project #gdg-workflow-512920 …
```

**Good sign:** the credential works. Google accepted the login and is now asking about money.

**Fix:** open `https://console.developers.google.com/billing/enable?project=PROJECT_ID`, pick the billing account, click **Set account**, and wait about 5 minutes.

**Not a fix:** *Google AI Pro* vouchers from the GDG program are personal Google One subscriptions, not Cloud billing credits.

## 5. Billing account project limit

**Symptom:** linking billing fails because the billing account already pays for too many projects.

**Fix (2 minutes):** go to https://console.cloud.google.com/billing/projects, open **⋮** on an old project you don't need, and click **Disable billing**. Then link the new project.

**Slow alternative:** the *Request Billing Quota Increase* form. It's reviewed manually and can take days.

## 6. Vibe: undefined

**Symptom:** the Gemini request shows `Vibe: undefined`, and the form trigger's output shows `" Vibe": "retro"`.

**Cause:** the form's **Custom Field Name** was typed as ` Vibe`, with a leading space. Later nodes look up `Vibe` exactly.

**Fix:** retype the field names exactly: `Talk title`, `Topic`, `Vibe`. Check the INPUT panel of **Gemini - write copy**. The keys must match with no spaces at either end.

## 7. Model not found (retired model)

```
404 - Publisher model `projects/…/locations/us-central1/publishers/google/models/gemini-2.5-flash` was not found or your project does not have access to it.
```

**Cause:** `gemini-2.5-flash` was **retired on Vertex AI on October 20, 2026**.

**Fix:** use `gemini-3.8-flash` (GA, retirement September 2027 or later). See [model findings](decisions-and-findings.md#model-findings).

**Tip:** when a model returns 404, check its page under https://docs.cloud.google.com/gemini-enterprise-agent-platform/models. AI summaries of model lists can be wrong or out of date, so read the model's own page.

## 8. Model not found (wrong location)

```
404 - Publisher model `projects/…/locations/us-central1/publishers/google/models/gemini-3.8-flash` was not found …
```

**Cause:** the model name is right, but the model is only available in **`global`** (and the multi-regions `us` and `eu`), not `us-central1`.

**Fix:** change **both** the host and the location in the URL:

```diff
- https://us-central1-aiplatform.googleapis.com/v1/projects/PROJECT_ID/locations/us-central1/publishers/google/models/gemini-3.8-flash:generateContent
+ https://aiplatform.googleapis.com/v1/projects/PROJECT_ID/locations/global/publishers/google/models/gemini-3.8-flash:generateContent
```

## 9. Imagen not found

```
404 - Publisher model `…/models/imagen-4.0-generate-001` was not found …
```

**Cause:** Imagen 4 is retired on Vertex AI.

**Fix:** use `gemini-3.1-flash-image` on `global`. The request format is different from Imagen, so the model name isn't the only change. See [decisions-and-findings.md](decisions-and-findings.md#imagen-is-retired-gemini-draws-the-images-now).

## 10. No credentials set

```
Node does not have any credentials set
```

**Cause:** workflow files never include credentials, so after every import, each Google node needs one picked again.

**Fix:** see the table in [setup-guide.md §7.3](setup-guide.md#73-pick-the-credentials).

## 11. Drive API not enabled

```
Google Drive API has not been used in project 1084386918147 before or it is disabled.
```

**Fix:** open the link in the error message, check the project name at the top, click **Enable**, and wait 2 to 5 minutes. The number in the error is a project **number**, not its name.

## 12. Service Accounts do not have storage quota

```
Forbidden - perhaps check your credentials?
Service Accounts do not have storage quota. Leverage shared drives …
```

**Cause:** the Drive node is logged in with the **Google Service Account** credential. Service accounts have no Drive storage, so they can't create files, not even inside a folder shared with them. n8n may pick this credential automatically if it's the only Google credential it has.

**Fix:** in **Save to gallery**, set **Authentication** to **OAuth2 (recommended)** and sign in with your own Google account.

## 13. The last node waits forever

**Symptom:** the card is saved to Drive, but **Show the card** (Form Ending, *Redirect to URL*) never finishes.

**Cause:** a Form Ending node waits for the form page to come back and collect the result. In desktop Chrome that page never loads (see [above](#the-form-page-is-blank-in-desktop-chrome)). Even on a phone, the attendee would wait the full 20 to 30 seconds.

**Fix (the current design):** the node was removed, and **Talk idea form → Respond When** is set to **Form Is Submitted**. The phone shows a thank-you right away, and the chain keeps running in the background.

## 14. No image in the answer

**Symptom:** **Pick image data** fails with an error like `Cannot read properties of undefined (reading 'inlineData')`.

**Cause:** the image model answered with text only, for example because a safety filter blocked the prompt. The expression looks for a part with `inlineData` and doesn't find one.

**What to do:** open **Gemini image - draw background**, read the text part and any `finishReason` in its OUTPUT, and soften the prompt.

> Not yet handled automatically. This is planned as the *"break it and fix it"* teaching moment in the workshop.
