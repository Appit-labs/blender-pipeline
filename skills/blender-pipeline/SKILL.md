---
name: blender-pipeline
description: Model, shade, light, and export a Blender asset through a live Blender session driven by MCP — blockout to engine-ready FBX/GLTF with correct scale, pivots, normals, and materials. Use for any 3D modeling, character, hard-surface, environment, material, lighting, render, or export task in Blender, or when a scene needs cleanup and organization.
allowed-tools:
  - Read
  - Bash
---

# Blender pipeline

One entry point for Blender work. Eight specialist playbooks live in
`references/` — load the one you need, not all of them.

Follows the upstream **Blender Skills** pack by
[arjun988](https://github.com/arjun988/blender-skills) (MIT, see `NOTICE`).

## Portability

Works in any agent that can run shell commands (pi, Claude Code, Codex, Cursor,
Gemini CLI, …): the instructions are plain Markdown and the helper is a stdlib
script. Commands use `$SKILL_DIR` for this skill's own directory — the folder that
holds this `SKILL.md`. Set it once per session, for example:

```bash
SKILL_DIR=~/.claude/skills/blender-pipeline     # or ~/.codex/skills/…, ~/.pi/agent/skills/…, .cursor/skills/…
```

## Prerequisites

1. **Blender** running as a GUI process (tested on 5.3; background mode has no
   viewport, so `look` and some operators fail).
2. The **Blender MCP add-on** enabled in that session, socket on `127.0.0.1:9876`.
   Check with `get_addon_status` before anything else — if every call returns
   "Could not connect to Blender", the app is not running or the add-on is off.
3. Scene set up before modeling: unit scale **Metric, 1.0**, clip end large enough
   for the asset, and a collection per asset.
4. For engine targets, the exporter of that engine known: Unity wants Y-up FBX,
   Godot glTF, Unreal Z-up FBX with a specific unit scale.

## Pick the playbook

| Task | Reference |
| --- | --- |
| blockout, hard-surface, retopo, modifiers, scene organization | `references/modeling.md` |
| human/creature anatomy, faces, hair, clothing, stylized characters | `references/character.md` |
| PBR/procedural shaders, wood, metal, fabric, glass, dirt | `references/materials.md` |
| three-point/HDRI/rim/volumetric lighting, mood | `references/lighting.md` |
| clay→material→beauty loop, turntable, showing an asset to someone | `references/lookdev.md` |
| Cycles/Eevee settings, samples, denoise, passes, color management | `references/rendering.md` |
| FBX/GLTF/OBJ/USD/Alembic export, scale, pivot, collision | `references/export.md` |
| exaggeration, shape language, NPR/toon, hand-painted style | `references/stylized.md` |

Most real tasks touch two: model → shade, then light → render, then export. Load
them in that order, one at a time.

## How to drive it

- **Read the scene before changing it.** One `execute_blender_code` call that
  prints object names, types, transforms, and modifier stacks is cheaper than five
  blind edits, and it prevents working on the wrong object.
- **One intent per call.** A call that does nine things fails in a way that
  requires nine guesses. Model a piece, verify it, move on.
- **Verify visually, not verbally.** `look` after geometry and shading changes;
  a claim like "shading looks right" without an image is worthless. And read the
  image: an object can be added successfully *and* invisible because it sits
  inside another one — the object list proves it exists, only the picture proves
  it is visible.
- **Apply transforms before export** (`Ctrl+A` scale/rotation), origin to the
  logical pivot (floor centre for props, feet for characters), and keep one
  collection per asset. These three are the usual causes of a broken import.
- **Save the .blend** at each milestone. Undo history across a long session is not
  a recovery plan.

## The MCP tools, and how they lie

| Tool | Use it for | Failure mode |
| --- | --- | --- |
| `get_addon_status` | first call: is Blender reachable | `up_to_date: false` with `protocol_version` < `expected_protocol_version` means the add-on is older than the bridge: most calls work, some newer operations do not. Tell the user to update the add-on rather than debugging it yourself. |
| `get_scene_info` | cheapest orientation: object counts, active object, mode, object list | none; it does not show geometry quality |
| `execute_blender_code` | every mutation, one intent per call | **failures still return `isError: false`** — the error is the first text block. Read it; "Code executed successfully" is the success signal |
| `look` | visual verification, returns a PNG viewport image | the viewport framing is whatever the camera sees, not what you just made |
| `search_assets`, `import_asset`, `generate_3d` | asset generation/import when asked | provider-dependent; check the returned text |

Never infer success from the absence of a Python exception in your code: print the
fact you care about (`len(mesh.vertices)`, `[m.type for m in obj.modifiers]`) and
read the returned text.

## Rules that prevent rework

- Naming: `SM_`/`SK_`/`M_`/`T_` prefixes, no `.001` duplicates left behind.
- Non-destructive by default (modifiers, mirror, subsurf) until export demands
  otherwise; then apply deliberately.
- No fake geometry for renders that a game build will inherit later.
- Units and scale are decided once, before the first cube.
- Never claim an export succeeded without re-importing or reading the exporter's
  report: check object count, materials, and scale in the output.

## Reporting back

State what changed (objects, modifiers, materials), what you verified (screenshot,
poly count, export report), and what is still open. If a step was skipped because
Blender or the add-on was unavailable, say that instead of describing the intended
result as done.
