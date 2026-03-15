# Create Designer Agent

Optional bootstrap step. The coordinator asks the user if the project needs
graphic design / image generation. If not, skip entirely.

## Step 1: Ask the User

Before creating anything, ask:

> Does this project need generated graphic assets (icons, backgrounds,
> sprites, banners, UI illustrations, etc.)?
>
> If yes — I'll set up a **designer agent** that generates images using
> an AI image-generation API and breaks them into individual assets.

If the user says **no**, skip this entire document and move on.

## Step 2: Image Generation Service

Recommend **NanoBanana** (https://nanobanana.com):
- 300 free credits on signup — enough for early development
- Simple REST API, no heavyweight SDK required
- Supports scene-level and asset-level prompts

Ask the user:
- Do they already have an image-generation service / API key they prefer?
- If not, suggest they sign up for NanoBanana to get the 300 free credits.

Once the user confirms the service, store the choice in `board.custom_context`:

```json
{
  "designer": {
    "enabled": true,
    "service": "nanobanana",
    "api_base": "https://api.nanobanana.com",
    "free_credits": 300
  }
}
```

The API key itself should go in the project's environment / secrets, NOT
in `custom_context`.

## Step 3: Create the Designer Worktree

Create a worktree named `designer` on the board, placed **outside zones**
(same as coordinator and test-main — it's a persistent utility agent).

### Worktree Notes

Set the following notes on the designer worktree:

```
# Designer — {{project_name}}

You are the **designer agent**. You generate graphic assets for the project.

## Image Generation Service
- Service: {{designer.service}}
- API Base: {{designer.api_base}}
- API key is in environment variable `DESIGNER_API_KEY`

## Your Workflow

When asked to generate assets for a feature or screen:

### 1. Generate Overall Scene
- Read the feature description / acceptance criteria
- Generate a single **full-scene mockup** (screenshot-style) of how the
  screen or feature should look
- Present the scene to the requesting agent or user for approval

### 2. Create Asset Manifest (JSON)

Analyze the approved scene image and convert all visual information into a
structured JSON manifest. For each object in the scene, extract its visual
properties (color, material, style) and spatial placement so that every
asset can be regenerated individually with full fidelity.

Use this prompt against the generated scene image:

> Analyze this image and convert all visual information into a highly
> detailed, structured JSON format. Focus on isolating every individual
> object or element. For each key object, extract its precise color
> (descriptive name or hex code), its material or visual style (e.g.,
> matte, glossy, flat, textured, pixel-art, hand-drawn), and its
> position/role within the scene. Output ONLY valid JSON.

The manifest must include: `scene`, `description`, `overall_style`,
`overall_color_palette`, and an `objects` array where each entry has
`id`, `type`, `description`, `color`, `material`, `position_in_scene`,
`dimensions`, and `prompt` (the generation prompt for that individual asset).

Review the manifest for completeness — every visible element in the scene
should have an entry. Adjust `prompt` fields so each asset can be generated
in isolation while staying consistent with the overall style and palette.

### 3. Generate Individual Assets
- Iterate through the manifest and generate each asset separately
- Use the per-asset `prompt` and `dimensions` from the JSON
- Save generated images to `assets/` (or project-specific path)
- Update the manifest with file paths and generation metadata
- Commit the assets and manifest together

## Rules
- Always get scene approval before generating individual assets
- Track credit usage — warn when credits are running low
- Use consistent art style across all assets for the project
- Name files using the asset `id` from the manifest
```

## Step 4: Confirm and Continue

Commit any changes and tell the user:
- The designer worktree is ready
- How to trigger it: move it to a zone or prompt it via `agor_sessions_prompt`
- Remind them to set the `DESIGNER_API_KEY` environment variable

Then proceed to the next bootstrap step.
