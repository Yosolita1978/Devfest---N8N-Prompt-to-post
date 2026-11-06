# Workshop runbook: DevFest, November 7, 2026

The host's plan for the 45-minute session: what to prepare, what to do minute by minute, and what to do when something breaks.

**Host:** Cristina (Seattle n8n Ambassadors) · **Form URL (published):** `https://yosola.app.n8n.cloud/form/prompt-to-poster`

---

## Prep checklist

Status as of **October 27, 2026**.

| ☐/☑ | Task | Notes |
|---|---|---|
| ☑ | Build and test the workflow end to end | All 8 nodes green on October 27 |
| ☑ | Confirm Edit Image runs on the n8n instance | Works on n8n Cloud 2.43.5 |
| ☐ | Fix known card issues | Text and people in images, tagline cut off. See [known issues](decisions-and-findings.md#known-issues). |
| ☐ | **Publish** the workflow and test the permanent URL with the editor closed | [setup-guide §9](setup-guide.md#9-publish-for-the-real-event) |
| ☐ | Create the QR code for the form URL | For the slides |
| ☐ | Load test: 40 or more simultaneous submissions | Check Gemini quotas, especially the image model |
| ☐ | Test the starter prompts against safety filters | Use the list below |
| ☐ | Pre-generate 10 backup cards | Keep them in a separate Drive folder, `DevFest Backup Cards` |
| ☐ | Write the take-home page and its QR code | Link to this repository, the gallery, and the prompt |
| ☐ | Update the session title and abstract (Imagen → Gemini image) | Tell the organizers |
| ☐ | Recheck model status the week before | [How to check](decisions-and-findings.md#how-to-check-model-status-next-time) |
| ☐ | Run the demo on a personal hotspot, not venue wifi | |
| ☐ | Check that the billing account and the billing budget alert are set | |

## Starter prompts

Fictional talks only. Test each one before the day.

| Talk title | Topic | Vibe |
|---|---|---|
| Teaching Robots to Salsa | Reinforcement learning for motion, explained with dance | retro |
| The Quiet Cloud | Designing calm, low-alert infrastructure | minimal |
| Debugging the Universe | What telescopes teach us about logging | cosmic |
| My Toaster Has an API | IoT for the kitchen, and what went wrong | playful |
| Ship It Friday | Continuous delivery without fear | bold |

---

## Before the session (30 minutes before)

1. Connect to your **personal hotspot**.
2. Open these tabs in Chrome:
   - **n8n editor** with the workflow (for the canvas tour)
   - **n8n Executions** tab for the workflow (to watch submissions come in)
   - **Google Drive → DevFest Gallery**, in **grid view** (the gallery)
   - **Google Drive → DevFest Backup Cards** (plan B)
3. Submit one test card **from your phone**, using the permanent URL, to warm everything up.
4. Have the slides with the **QR code** ready.

> Don't open the form in desktop Chrome. It shows a blank page there ([why](troubleshooting.md#the-form-page-is-blank-in-desktop-chrome)). Use your phone for all live submissions.

---

## Minute by minute

| Minutes | Block | What you do | What's on screen |
|---|---|---|---|
| 0–5 | **Hook** | Scan the QR code with your phone and submit "Teaching Robots to Salsa" (retro). While it runs (about 30 seconds), introduce yourself. | Slides, then Drive with the new card |
| 5–12 | **The core idea** | Explain prompt chaining. Show a raw vibe prompt next to a Gemini-written prompt, with their images. | Slides |
| 12–22 | **Walk the canvas** | Tour the nodes. Build or show **Gemini - write copy**: the system instruction, the `responseSchema`, and why structured output keeps the chain from breaking. Show the image node's request and the **Pick image data** expression. | n8n editor |
| 22–32 | **The room makes cards** | Show the QR slide. Watch **Executions**. In Drive, open new cards on the laptop **first**, then put Drive on the projector and call out favorites. | QR slide, then Drive (after review) |
| 32–39 | **Break it and fix it** | See the scenarios below. | n8n editor + Drive |
| 39–45 | **Take-home + Q&A** | Show the take-home QR: this repository, the gallery link, and the Gemini prompt. | Slides |

### Break it and fix it: scenarios

1. **Remove the "no text" rule** from the system instruction. The new card has signs and lettering that clash with the headline band. Point out that the model writes text *well* now, and that's exactly why you keep control of it.
2. **Trigger a safety block** with a pre-tested prompt. **Pick image data** fails, because there's no image in the answer. Open the image node's OUTPUT and read the reason. ([#14](troubleshooting.md#14-no-image-in-the-answer))
3. **Change the vibes live:** add a new option (for example `vaporwave`) to the form's dropdown and submit with it.

Undo changes 1 and 3 afterwards. Better: duplicate the workflow before the session and break the copy, not the published one.

---

## Plan B

| Problem | What you see | What you do |
|---|---|---|
| Quota or rate limit | Red nodes in Executions, `429` or `RESOURCE_EXHAUSTED` | Say "the room broke the quota!" and switch to the **backup cards** folder |
| Slow generation | Cards take more than a minute | Talk through the canvas while they come in. Show backup cards if needed. |
| Safety block on a real submission | **Pick image data** fails for one execution | Make it a teaching moment: open the execution and show the reason |
| Inappropriate card | You see it during review | Don't project it. Move it out of the gallery folder. |
| Model "not found" | 404 on a Gemini node | A model was retired. Fall back to backup cards, and fix it after the session ([#7](troubleshooting.md#7-model-not-found-retired-model)). |
| Wifi drops | Nothing loads | Host: hotspot. Attendees: cellular data. |
| Form blank on an attendee's phone | Rare. It's the desktop Chrome issue. | Ask them to open the link in their phone's default browser |

---

## After the session

1. Count the cards in **DevFest Gallery**. That's the success metric.
2. Review the gallery and remove anything inappropriate **before** sharing the link publicly.
3. **Unpublish** the workflow (or stop sharing the form URL), so it doesn't keep generating cards and costs.
4. Check the billing report for the day's cost, and note it here for next time.
