# blender-pipeline

<h3 align="center">Eight specialist Blender playbooks behind one router: 383 description tokens became 86.</h3>

<p align="center">
  <a href="LICENSE"><img alt="license" src="https://img.shields.io/badge/license-MIT-3DA639?style=flat-square"></a>
  <img alt="agents" src="https://img.shields.io/badge/works%20in-pi%20%7C%20Claude%20Code%20%7C%20Codex%20%7C%20Cursor-4B5563?style=flat-square">
  <img alt="skillspector" src="https://img.shields.io/badge/SkillSpector-0%2F100%20SAFE-2EA043?style=flat-square">
  <img alt="tokens" src="https://img.shields.io/badge/session%20cost-~86%20tok-6E7681?style=flat-square">
</p>

<p align="center">
  <img alt="blender-pipeline demo" src="skills/blender-pipeline/assets/demo.gif" width="820">
</p>

## Install

One command for each agent — the skill is plain Markdown plus a script, so nothing is coupled to one ecosystem.

```bash
# Claude Code (plugin, stays updatable)
claude plugin marketplace add Appit-labs/appit-skills
claude plugin install blender-pipeline@appit-skills

# pi
pi install git:github.com/Appit-labs/blender-pipeline

# any agent that reads a skills directory
git clone git@github.com:Appit-labs/blender-pipeline.git
cp -r blender-pipeline/skills/blender-pipeline ~/.claude/skills/      # or ~/.codex/skills/, .cursor/skills/, …
```

## Why this exists

Eight separate skills cost 383 tokens of description on every session and ~8k if loaded together. One router plus on-demand references costs the one playbook you actually need.

## Use

- Model: `references/modeling.md`
- Character: `references/character.md`
- Shade / light / render: `references/materials.md`, `lighting.md`, `rendering.md`
- Ship: `references/export.md` — scale, pivot, normals, collision`

## Requires

- Blender 4.2+ (tested on 5.3) running as a GUI process — background mode has no viewport.
- A Blender MCP add-on listening on `127.0.0.1:9876`; check with `get_addon_status` first.
- For engine targets, that engine's import convention (Unity Y-up FBX, Godot glTF, Unreal Z-up FBX).

## What you get

- One intent per `execute_blender_code` call; `look` returns a real viewport PNG; failures come back with `isError: false`, so read the text.
- An add-on protocol older than the bridge reports `up_to_date: false` — update the add-on rather than debugging around it.
- The eight playbooks are the upstream Blender Skills pack by arjun988 (MIT) — see `NOTICE`. The router is original.

## Cost

| | tokens |
| --- | --- |
| always-on (the description, every session) | ~86 |
| on invoke | ~1k + one playbook |

The demo above is regenerated from a real run: `demos/blender-pipeline.json` holds the captured
output, `tools/make_demo.py` renders it. No invented screenshots.

## Security

Static analysis with SkillSpector: **0/100, SAFE, 0 open findings** against the committed
baseline. Every accepted finding is listed with its reason in [SECURITY.md](SECURITY.md).
The skill never uses `shell=True`, never reads or stores a credential, and never chooses a
network destination at runtime.

## Part of the Appit collection

Browser QA · CI triage · Postgres · runbooks · SSH · Trello · cloud browsers · OpenDesign · Blender —
one marketplace: [appit-skills](https://github.com/Appit-labs/appit-skills).

## License

MIT. See [LICENSE](LICENSE).

The eight `references/*.md` playbooks are the Blender Skills pack by
[arjun988](https://github.com/arjun988/blender-skills) (MIT); see [NOTICE](NOTICE).
