# Decisions and findings

Why the workflow is built the way it is, and what we learned while building it. Last updated **October 27, 2026**.

## Contents

- [Model findings](#model-findings)
- [Platform findings](#platform-findings)
- [Design decisions](#design-decisions)
- [Known issues](#known-issues)
- [Impact on the workshop proposal](#impact-on-the-workshop-proposal)

---

## Model findings

### Current models

| Role in the chain | Model ID | Status | Locations | Retirement |
|---|---|---|---|---|
| Writes copy + art direction | `gemini-3.8-flash` | GA | `global`, `us`, `eu` | September 2, 2027 or later |
| Draws the background | `gemini-3.1-flash-image` (nicknamed "Nano Banana 2") | GA | `global`, `us`, `eu` | not checked |

Sources: [Gemini 3.8 Flash](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/gemini/3-8-flash) · [Gemini 3.1 Flash Image](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/gemini/3-1-flash-image)

### Models from the original plan that no longer work

| Model | What happened | Source |
|---|---|---|
| `gemini-2.5-flash` | Retired on Vertex AI on **October 20, 2026** | [Gemini 2.5 Flash model page](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/gemini/2-5-flash) |
| `imagen-4.0-generate-001` | Retired on Vertex AI. Third-party trackers give **June 30, 2026** (Gemini API: August 17, 2026). Google's replacement is the Gemini image models. | [Model versions and lifecycle](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/learn/model-versions?hl=en) |

The workshop proposal was written for Imagen. Both models changed in the months before the event.

### Newer models only run on `global`

Both current models list **`global`** (plus the multi-regions `us` and `eu`) and **not** `us-central1`. Calling them on a regional endpoint returns a 404 that looks exactly like "model doesn't exist".

The global endpoint also uses a different host:

```
https://aiplatform.googleapis.com/v1/projects/PROJECT_ID/locations/global/publishers/google/models/MODEL_ID:generateContent
```

Note there's no `us-central1-` prefix in the host.

### Imagen is retired: Gemini draws the images now

The image request is shaped like a normal Gemini request, not like Imagen's:

| | Imagen (old) | Gemini image (now) |
|---|---|---|
| Method | `:predict` | `:generateContent` |
| Request | `instances` + `parameters` | `contents` + `generationConfig` |
| Aspect ratio | `parameters.aspectRatio` | `generationConfig.imageConfig.aspectRatio` |
| Output modes | image | `responseModalities: ["TEXT", "IMAGE"]`. The docs' examples always use both. |
| Image in answer | `predictions[0].bytesBase64Encoded` | one of the `candidates[0].content.parts`, as `inlineData.data` |
| Block people | `personGeneration: "dont_allow"` | no equivalent setting found, so it's handled through the prompt |

Because the image part's position in `parts` isn't guaranteed (a text part may come first), the **Pick image data** node finds it:

```js
{{ $json.candidates[0].content.parts.find(part => part.inlineData).inlineData.data }}
```

Supported aspect ratios: 1:1, 3:2, 2:3, 3:4, 1:4, 4:1, 4:3, 4:5, 5:4, 1:8, 8:1, 9:16, 16:9, 21:9. Resolutions: 512, 1K, 2K, 4K. A 3:4 image came back at about **2.15 MB as PNG**.

### The image model draws legible text

The proposal says image models *"still garble lettering"*. That's no longer true for `gemini-3.1-flash-image`: our first card included clean, readable signs ("SALSA NIGHT", "RHYTHM CLUB"). n8n still adds the headline and tagline itself, but for a different reason now. See [decision D3](#d3-n8n-adds-the-text-not-the-image-model).

### How to check model status next time

1. Open the model's own page under https://docs.cloud.google.com/gemini-enterprise-agent-platform/models and read **Model ID**, **Launch stage**, **Locations**, and **Retirement date**.
2. Don't trust model lists from summaries or memory. During this build, a guessed model ID and a guessed region both failed.
3. Recheck the week before the event.

---

## Platform findings

| Finding | Details |
|---|---|
| **Vertex AI is shown as "Agent Platform"** | The console search shows *Vertex AI (Agent Platform)*. The role *Vertex AI User* is now **Agent Platform User** (`roles/aiplatform.user`). Docs live under `docs.cloud.google.com/gemini-enterprise-agent-platform/`. |
| **Edit Image works on n8n Cloud** | The multi-step resize, rectangle, and text operations ran fine on n8n Cloud 2.43.5. This was a prep checklist risk, now cleared. |
| **n8n form pages don't display in the host's desktop Chrome** | The server returns a valid form. It works on phones. The cause in Chrome isn't confirmed. See [troubleshooting](troubleshooting.md#the-form-page-is-blank-in-desktop-chrome). |
| **The n8n Form trigger shows up as "n8n Form"** | In the trigger search, and fields now have a separate **Custom Field Name**. |
| **Service accounts can't write to Drive** | They have no storage quota. Drive must use OAuth2 with a real account. |
| **Billing and APIs take a few minutes** | After linking billing or enabling an API, wait 2 to 5 minutes before retrying. |
| **AI Pro vouchers aren't Cloud credits** | The GDG *Google AI Pro* codes are personal Google One subscriptions for attendees. They can't pay for a Cloud project. |

---

## Design decisions

### D1. HTTP Request nodes instead of n8n's AI nodes

**Decision:** both Gemini calls use plain **HTTP Request** nodes with one **Google Service Account** credential.

**Why:**
- Both steps look the same, so one pattern gets explained on stage, not three nodes (LLM Chain + Chat Model + Output Parser).
- The JSON body *is* the lesson: the audience sees the system instruction, the schema, and the chained `image_prompt`.
- Switching models means editing one URL, which mattered when two models retired before the event.

**Trade-off:** the Gemini response is less friendly, so the **Read Gemini copy** node parses it.

### D2. Structured output with a response schema

**Decision:** Gemini must return JSON with `headline`, `tagline`, and `image_prompt`, enforced through `responseMimeType: "application/json"` plus `responseSchema`.

**Why:** without a schema, the model sometimes adds markdown or extra words, and the next node breaks. With it, `JSON.parse` always works. See [`prompts/gemini-prompt.md`](../prompts/gemini-prompt.md).

### D3. n8n adds the text, not the image model

**Decision:** the image model draws only the background. **Add headline and tagline** draws a dark band and writes the text.

**Why (updated):** the original reason, garbled lettering, mostly doesn't apply anymore (see [findings](#the-image-model-draws-legible-text)). It's still the better design:
- The headline is spelled **exactly** as Gemini wrote it, every time.
- Every card has the same layout, which looks good in a gallery.
- Teaching value: the chain stays visible as copy → art → composition.

**Detail:** the node **resizes to 896×1280 first**, so the band and text land in the same place whatever size the model returns.

### D4. The form responds as soon as someone submits

**Decision:** **Respond When = Form Is Submitted**, with no Form Ending node.

**Why:** the attendee gets an instant thank-you instead of waiting 20 to 30 seconds. The Form Ending page also froze (see [troubleshooting #13](troubleshooting.md#13-the-last-node-waits-forever)).

### D5. No gallery link for attendees during the session

**Decision:** the thank-you message doesn't link to the gallery. The host opens Drive on the laptop, reviews new cards, and only then projects them. The gallery link is shared at the end, in the take-home.

**Why:** it matches the proposal's risk plan, *"Host reviews every card before projecting; nothing auto-displays."* Anyone with the link sees every card, including unreviewed ones.

### D6. Two Google credentials

| Credential | Type | Used by |
|---|---|---|
| Vertex AI - n8n-devfest | Google Service Account API (scope `cloud-platform`) | both Gemini nodes |
| Google Drive - yosola | Google Drive OAuth2 | Save to gallery |

**Why:** the robot is limited to Vertex AI and billed to the project. Drive needs a real account with storage ([troubleshooting #12](troubleshooting.md#12-service-accounts-do-not-have-storage-quota)).

### D7. No static page

**Decision:** QR code → n8n form on phones, and the gallery is the Drive folder in Chrome.

**Why:** the n8n form works on phones, and Drive works in Chrome. A static gallery page would need its own Drive access and hosting, which means more pieces that can break on the day.

---

## Known issues

| Issue | Seen on | Planned fix |
|---|---|---|
| Images contain text (signs, name tags) | First card: "SALSA NIGHT", "RHYTHM CLUB", "BEATRIZ" | Add a fixed "no text, no signs" line to the image request |
| People appear in the background | First card: a crowd | Add a fixed "no people" line to the image request. There's no `personGeneration` setting anymore. |
| The tagline's third line is cut off | First card | Smaller font, or start the band and text higher |
| No handling when the image is blocked | Not seen yet | Error branch, or the planned teaching moment ([troubleshooting #14](troubleshooting.md#14-no-image-in-the-answer)) |
| Drive folder ID is in the workflow file | `workflow/prompt-to-poster.json` | Replace it with a placeholder before publishing the repository, since the folder is "anyone with the link" |

---

## Impact on the workshop proposal

| Proposal says | Reality now | Suggested change |
|---|---|---|
| "Gemini and **Imagen** on Vertex AI" | Imagen is retired | "Gemini text and Gemini image models on Vertex AI" |
| "Image models still garble lettering" | The new model writes clean text | "We composite the text so it's exact and consistent" |
| Five nodes | Eight nodes (three are glue: parse, pick image, base64 → file) | Keep "five steps" on slides, and mention the glue nodes in the canvas tour |
| Break-it: "remove 'no text' → garbled lettering" | Lettering won't be garbled anymore | Show instead that removing "no text" puts **unwanted signs** on the card, so the text can clash with the overlay |
| Form completion page links to the card | Removed | Phone shows a thank-you, and the host projects reviewed cards |

Tell the DevFest organizers about the title and abstract change (Imagen → Gemini image).
