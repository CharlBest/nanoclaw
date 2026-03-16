---
name: daily-selfie
description: "Timelapse selfie collector. Automatically processes photos sent by Charl or Anja — if the photo is a facial selfie, saves it to the timelapse archive. No trigger needed."
---

# Daily Selfie — Timelapse Collector

> This skill runs silently in the background. When a user sends a photo,
> check if it's a facial selfie and archive it for a timelapse project.

---

## Purpose

Charl and Anja are building a long-term timelapse of their faces — one
photo per day each. Every photo sent to the bot is checked: if it's a
front-facing selfie showing their face clearly, it gets saved to a
permanent archive outside the container.

---

## How It Works

When a message contains `[Photo]` with a `[saved: /workspace/group/photos/...]`
path, this skill activates automatically. No trigger word needed.

### Step 1: Read the Image

Use the Read tool to view the image file at the path shown in the message.
The path will look like `/workspace/group/photos/photo-2026-03-16-123.jpg`.

### Step 2: Determine If It's a Selfie

Look at the image and assess:
- Is it a **front-facing photo of a person's face**? (selfie or portrait)
- Is the face **clearly visible** and reasonably centered?
- Is it a **close-up or upper-body shot** (not a full landscape with a
  tiny person in the distance)?

If YES → proceed to Step 3.
If NO → ignore silently. Do not respond to the user. Wrap your entire
output in `<internal>` tags. The photo is just a regular photo, not a
selfie for the timelapse.

### Step 3: Identify the Person

Determine who sent the photo from the sender name in the message:
- **Charl** → save to `/workspace/extra/persisted_data/daily-selfie/charl/`
- **Anja** → save to `/workspace/extra/persisted_data/daily-selfie/anja/`

If the sender cannot be determined, use the sender name as the folder
(lowercase, no spaces).

### Step 4: Save the Photo

Copy the photo to the archive with a date-based filename:

```bash
cp /workspace/group/photos/<original-filename> /workspace/extra/persisted_data/daily-selfie/<person>/selfie-YYYY-MM-DD.jpg
```

Use today's date for the filename. This means one photo per person per day.
If a file for today already exists, overwrite it (they sent a better one).

### Step 5: Respond Briefly

Send a short confirmation via `mcp__nanoclaw__send_message`:
- "📸 Saved!" or "📸 Got it!" — keep it to 2-3 words.
- Do NOT send a long message. The whole point is this is frictionless.

Then wrap any remaining output in `<internal>` tags.

### Step 6: Clean Up

After saving to the archive, delete the temporary copy from the workspace:

```bash
rm /workspace/group/photos/<original-filename>
```

This keeps the workspace clean — the permanent copy lives in the archive.

---

## Edge Cases

- **Not a selfie** (landscape, food, screenshot, meme, etc.): ignore
  completely. Wrap output in `<internal>` tags. Do not respond.
- **Group photo with multiple faces**: still save it — it's part of the
  timelapse story.
- **Photo with sunglasses/hat**: still save — face is still visible.
- **Very dark or blurry photo**: skip it, don't save. Optionally respond
  with "📸 That one's a bit blurry — try again?"
- **Second selfie same day**: overwrite the existing one for that day.
- **Caption on the photo**: process it regardless of caption content.
  If the caption contains a question or request, answer it AND process
  the selfie.

---

## Important

- This skill has **NO trigger** — it activates on any `[Photo]` message.
- Keep responses **minimal** — this should feel automatic, not chatty.
- The archive path `/workspace/extra/persisted_data/daily-selfie/` maps to
  `/Users/charlbest/nanoclaw_persisted_data/daily-selfie/` on the host machine,
  outside the container. Files saved here persist permanently.
