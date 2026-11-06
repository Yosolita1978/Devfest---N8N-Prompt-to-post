# Setup guide

This guide sets up the Prompt to Poster workflow from scratch, written for someone using Google Cloud and n8n for the first time.

Every part ends with a ✅ **Check**. Don't move on until the check passes. If something looks different from what's described here, look it up in [troubleshooting.md](troubleshooting.md).

> Google renames console buttons from time to time. In 2026, **Vertex AI** also shows up as **"Agent Platform"** in the console. If a name doesn't match, search for the technical ID given in each step. IDs don't change.

**Time needed:** about 45 minutes the first time.

## Contents

1. [What you need](#1-what-you-need)
2. [Google Cloud: project and billing](#2-google-cloud-project-and-billing)
3. [Google Cloud: enable the Vertex AI API](#3-google-cloud-enable-the-vertex-ai-api)
4. [Google Cloud: create the service account](#4-google-cloud-create-the-service-account)
5. [n8n: the Vertex AI credential](#5-n8n-the-vertex-ai-credential)
6. [Google Drive: the gallery folder](#6-google-drive-the-gallery-folder)
7. [n8n: import and connect the workflow](#7-n8n-import-and-connect-the-workflow)
8. [First test run](#8-first-test-run)
9. [Publish for the real event](#9-publish-for-the-real-event)

---

## 1. What you need

| Item | Notes |
|---|---|
| A Google account | For Google Cloud and Google Drive |
| A Google Cloud project **with billing** | Gemini calls cost money per request. Free vouchers like *Google AI Pro* are personal subscriptions and **cannot** pay for a Cloud project. |
| An n8n instance | n8n Cloud works. This was tested on n8n Cloud 2.43.5. |
| A phone | For testing the form (see [why](troubleshooting.md#the-form-page-is-blank-in-desktop-chrome)) |

## 2. Google Cloud: project and billing

A **project** is a folder in Google Cloud. Everything you create (APIs, the service account, the bill) lives inside it.

1. Go to **https://console.cloud.google.com** and sign in.
2. Click the project box at the top left, next to the Google Cloud logo, and choose your project. Or click **New project** to create one.
3. In the same window, copy the **Project ID** (it looks like `gdg-workflow-512920`). You'll paste it into the workflow later.
4. Make sure billing is on. Open this link, replacing `PROJECT_ID` with yours, then pick a billing account and click **Set account**:
   `https://console.developers.google.com/billing/enable?project=PROJECT_ID`

✅ **Check:** the top bar shows your project, and the billing page doesn't ask you to link an account anymore.

> **"This billing account has reached its project limit"?** Go to https://console.cloud.google.com/billing/projects, open the **⋮** menu on an old project you don't need, click **Disable billing**, then try again. This takes 2 minutes. The quota increase form takes days.

## 3. Google Cloud: enable the Vertex AI API

An **API** is a door that lets programs, like n8n, talk to a Google service. Every door starts closed.

1. Open https://console.cloud.google.com/apis/library/aiplatform.googleapis.com
2. Check that the project box at the top still shows your project.
3. Click **Enable** and wait about 30 seconds.

✅ **Check:** the page shows **API Enabled** or a **Manage** button.

> Searching "Vertex AI API" in the console search may only show **"Vertex AI (Agent Platform)"**. That's why the direct link above is easier.

## 4. Google Cloud: create the service account

A **service account** is a robot user. n8n logs in as this robot to call Gemini, so your personal password is never used.

### 4.1 Create the robot

1. In the console search bar, type **Service Accounts** and open **Service Accounts – IAM & Admin**.
2. Click **+ Create service account**.
3. Fill in:
   - **Name:** `n8n-devfest`
   - **ID:** leave the value it fills in automatically
   - **Description:** `Used by n8n to call Gemini for the DevFest workshop`
4. Click **Create and continue**.

### 4.2 Give it one role

1. Click **Select a role**. A long list appears. Ignore it and use the search box.
2. Type `Agent Platform` and choose **Agent Platform User**.
   - This is the role that used to be called **Vertex AI User**. Its technical ID is `roles/aiplatform.user`.
   - Don't pick anything with **Service Agent**, **Admin**, or **Viewer** in its name.
3. Leave **IAM condition** empty, then click **Continue**, then **Done**.

✅ **Check:** the list shows `n8n-devfest@PROJECT_ID.iam.gserviceaccount.com`.

### 4.3 Download the key

1. Click the service account's email, then open the **Keys** tab.
2. Click **Add key**, then **Create new key**, then **JSON**, then **Create**.
3. A `.json` file downloads to your **Downloads** folder.

✅ **Check:** the Keys tab shows one key, and the file is in Downloads.

> 🔐 **This file is a password.** Anyone who has it can spend money on your project. Keep it out of this repository (the `.gitignore` blocks `*.json` key files), out of chats, and off the projector. If it leaks, delete it from the Keys tab and the leaked copy stops working right away.

## 5. n8n: the Vertex AI credential

A **credential** is n8n's locked box for passwords. The workflow uses it without anyone seeing the key.

1. Open the key file with **TextEdit** (right-click, then **Open With**, then **TextEdit**). Don't edit or save it.
2. In n8n, open **Workflows**, click the small **▾** next to **Create Workflow**, then **Create Credential**.
3. Search for `Google Service Account`, choose **Google Service Account API**, then **Continue**.
4. Fill in:

   | Field | Value |
   |---|---|
   | Region | leave the default |
   | Service Account Email | the `client_email` value from the key file, without quotes |
   | Private Key | the `private_key` value, from `-----BEGIN PRIVATE KEY-----` to `-----END PRIVATE KEY-----\n`, without quotes. The `\n` parts are fine. |
   | Impersonate a User | off |
   | Set up for use in HTTP Request node | **on** |
   | Scope(s) | `https://www.googleapis.com/auth/cloud-platform` |

5. Rename the credential (top left) to `Vertex AI - n8n-devfest` and click **Save**.
6. Close TextEdit and choose **Don't Save**.

✅ **Check:** n8n saves the credential with no red error.

## 6. Google Drive: the gallery folder

1. Go to https://drive.google.com. Click **+ New**, then **New folder**, and name it `DevFest Gallery`.
2. Right-click the folder, then **Share**, then **Share** again. Under **General access**, choose **Anyone with the link** with the role **Viewer**, and click **Done**.
3. Open the folder and copy the ID from the address bar. It's the part after `folders/`:
   `https://drive.google.com/drive/folders/`**`16WZoW6IE1fGLYBw9RlAHVnslFquZqyhG`**

✅ **Check:** you have the folder ID copied.

## 7. n8n: import and connect the workflow

### 7.1 Import

1. In n8n, click **Create Workflow**.
2. Click **⋯** at the top right, then **Import from File...**, and choose `workflow/prompt-to-poster.json`.
   - On a Mac, press **Cmd + Shift + G** in the file window to type a folder path.

✅ **Check:** you see 8 nodes in a row, from **Talk idea form** to **Save to gallery**.

### 7.2 Update your project and folder

If you're **not** using the `gdg-workflow-512920` project, change it in two places:

- **Gemini - write copy:** the URL contains `/projects/gdg-workflow-512920/`
- **Gemini image - draw background:** same

Then, in **Save to gallery**, paste your folder ID into **Folder** (mode **By ID**).

### 7.3 Pick the credentials

Credentials are never stored in workflow files, so pick them again after every import:

| Node | Credential | Why |
|---|---|---|
| Gemini - write copy | Vertex AI - n8n-devfest | The robot calls Gemini, billed to the project |
| Gemini image - draw background | Vertex AI - n8n-devfest | Same |
| Save to gallery | Google Drive (OAuth2) with your account | Robots have **no Drive storage**, so files must be saved as you |

For **Save to gallery**:

1. Set **Authentication** to **OAuth2 (recommended)**, not Service Account.
2. In **Credential to connect with**, click **+ Create new credential**, then **Sign in with Google**, choose your account, and allow access.
3. Rename it to `Google Drive - yosola` (or your name) and click **Save**.

If Drive later complains that the **Google Drive API has not been used in project …**, open the link in the error, click **Enable**, and wait 2 to 5 minutes.

Press **Cmd + S** to save the workflow.

✅ **Check:** none of the 8 nodes shows a warning triangle.

## 8. First test run

1. Click **Execute Workflow** at the bottom of the canvas.
2. If a blank popup window opens in Chrome, close it. See [troubleshooting](troubleshooting.md#the-form-page-is-blank-in-desktop-chrome).
3. On your **phone**, open the test URL:
   `https://yosola.app.n8n.cloud/form-test/prompt-to-poster`
4. Fill in:
   - **Talk title:** `Teaching Robots to Salsa`
   - **Topic:** `Reinforcement learning for motion, explained with dance`
   - **Vibe:** `retro`
5. Tap **Make my poster 🎨**.

✅ **Check:**

- The phone shows a thank-you message right away.
- In n8n, all 8 nodes turn green after 20 to 30 seconds.
- A file like `20261027-151857-retro.png` appears in **DevFest Gallery**.

To inspect the card in n8n, double-click **Add headline and tagline** and click **View** in the OUTPUT panel.

## 9. Publish for the real event

The test URL only works while the editor is listening, and for one submission at a time. For the workshop, publish the workflow:

1. Click **Publish** at the top right.
2. The permanent form URL is:
   `https://yosola.app.n8n.cloud/form/prompt-to-poster` (no `-test`)
3. Submissions no longer appear on the canvas. Find them in the workflow's **Executions** tab.

✅ **Check:** with the n8n editor **closed**, submit from your phone and confirm a new card appears in Drive.

After publishing, create the QR code for the permanent URL. See [workshop-runbook.md](workshop-runbook.md).

## Keeping the repository in sync

After changing the workflow in n8n, export it and replace the file in this repository:

1. In n8n, click **⋯**, then **Download**.
2. Save it over `workflow/prompt-to-poster.json`.

The export includes credential **names**, but never keys or tokens.
