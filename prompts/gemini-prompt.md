# Gemini prompt (the first link in the chain)

Gemini turns a talk idea and a vibe into three things:

1. **A safety decision**: `is_appropriate` and `reason`. If the idea isn't right for a family tech event, the workflow stops here.
2. **Copy** for the card: `headline` and `tagline`.
3. **Art direction** for the Gemini image model (`gemini-3.1-flash-image`): `image_prompt`.

The third output is the prompt chaining idea: one model writes the prompt for the other.

## Two safety checks

The workflow checks every idea twice before drawing anything:

1. **Blocked words?** (an If node right after the form) looks for a fixed list of swear words, slurs, and political words in the title and topic. It's instant and free, but easy to get around with misspellings.
2. **Is it appropriate?** (an If node after **Read Gemini copy**) acts on Gemini's own `is_appropriate` decision. Gemini understands meaning, so it catches what the word list misses, like "tr*mp" or "our new city leader".

Both checks stop rejected ideas quietly: the participant still sees the form's thank-you message, but no poster is made. Each rejection, and Gemini's `reason`, shows up in **Executions**.

## System instruction

```text
You are an art director who designs promo cards for tech conference talks, and the safety reviewer for a family-friendly tech event.
First decide if the idea is appropriate. Set is_appropriate to false if the title or topic includes, mentions, or hints at: politics of any kind (elections, voting, candidates, politicians, political parties, government policy, protests, or the recent mayoral election); hate, slurs, or insults about any person or group; harassment or mocking of real people; sexual content; violence, weapons, or self-harm; drugs; or anything else not suitable for a family tech event.
Treat misspellings, symbols, and coded words (for example tr*mp or p0litics) the same as the real words. When in doubt, set is_appropriate to false.
- reason: one short sentence explaining your decision.
If is_appropriate is false, set headline, tagline, and image_prompt to empty strings. If is_appropriate is true, write:
- headline: a punchy headline, 8 words max. It can be the title or a sharper version of it.
- tagline: one sentence, 15 words max, that makes people want to attend.
- image_prompt: art direction for an image model, 60 to 100 words. Describe the subject, art style, color palette, lighting, and composition. Match the vibe. Keep the main subject in the upper two thirds of the frame.
Rules for image_prompt: no text, letters, numbers, or words in the image. No logos or brand marks. No people at all; use objects, robots, animals, landscapes, or abstract shapes instead. Keep it safe for a family audience.
```

(In the workflow this is one paragraph. It's split into lines here so it's easier to read.)

## User message

It's built from the form fields:

```text
Talk title: Teaching Robots to Salsa
Topic: Reinforcement learning for motion, explained with dance
Vibe: retro
```

## Response schema (structured output)

Vertex AI must return JSON in exactly this shape. Without a schema, Gemini sometimes adds markdown or extra words, and the next node in the chain breaks.

`propertyOrdering` makes Gemini answer `is_appropriate` first, so it decides before it writes anything.

```json
{
  "type": "OBJECT",
  "properties": {
    "is_appropriate": { "type": "BOOLEAN", "description": "false if the idea breaks any safety rule" },
    "reason": { "type": "STRING", "description": "One short sentence explaining the decision" },
    "headline": { "type": "STRING", "description": "8 words max. Empty if not appropriate." },
    "tagline": { "type": "STRING", "description": "15 words max. Empty if not appropriate." },
    "image_prompt": { "type": "STRING", "description": "Art direction for the image model. No text, logos, or people. Empty if not appropriate." }
  },
  "required": ["is_appropriate", "reason", "headline", "tagline", "image_prompt"],
  "propertyOrdering": ["is_appropriate", "reason", "headline", "tagline", "image_prompt"]
}
```

## Example output

An appropriate idea:

```json
{
  "is_appropriate": true,
  "reason": "A fun, family-friendly talk about robotics and machine learning.",
  "headline": "Teaching Robots to Salsa",
  "tagline": "How reinforcement learning taught a robot to find the beat.",
  "image_prompt": "A chrome 1970s-style robot mid-spin on a glowing checkered dance floor, retro poster illustration with halftone texture, warm palette of tangerine, mustard yellow, and teal, soft disco-ball light spots, low camera angle, robot centered in the upper two thirds, simple dark gradient background, no text, no logos, no people."
}
```

A rejected idea (title: "What our new city leader means for tech"):

```json
{
  "is_appropriate": false,
  "reason": "The idea is about local politics and the recent election.",
  "headline": "",
  "tagline": "",
  "image_prompt": ""
}
```
