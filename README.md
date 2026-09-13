# chengfeng-videocut-cut

Review Chinese talking-head retakes before committing word-level cuts.

Produce one reviewable cut ledger from real media and transcript evidence. It does not render a finished video or modify the original media.

中文：识别口误、重复与多次 take，生成可审核的删词方案。 独立 Skill 文件包，不包含 Runtime、Studio、FFmpeg 或渲染浏览器。

## Quick Start

Source preview **0.1.0-beta.1**. Requires Node.js 18+, npm, Git and access to GitHub. These commands install Skill files only; they do not install or start a workbench. Review the package before executing it.

```sh
npx -y github:Agentchengfeng/chengfeng-videocut-cut#v0.1.0-beta.1 plan --host codex
npx -y github:Agentchengfeng/chengfeng-videocut-cut#v0.1.0-beta.1 install --host codex
npx -y github:Agentchengfeng/chengfeng-videocut-cut#v0.1.0-beta.1 doctor --host codex
```

The default target is your user home. For an isolated test, create an empty directory first and append `--target-root "<existing-test-directory>"` to all three commands. Omit `--host codex` to install only the neutral Agent directory. A full commit SHA can replace the tag for immutable retrieval.

Files go to `.agents/skills/chengfeng-videocut-cut` under the target home; Codex mode also creates a precise `.codex/skills/chengfeng-videocut-cut` entry. Identical installs are reused. A conflicting name, version or locally modified copy is rejected without overwriting it. No automatic upgrade or uninstall is provided.

## Use with an Agent

Once your host discovers the Skill, ask it to use `$chengfeng-videocut-cut` for your task. The actual method is [SKILL.md](.agents/skills/chengfeng-videocut-cut/SKILL.md); the root SKILL.md is an installer entry, not a duplicate business method. A new host session may be needed. Installing files is not proof that a host has loaded them.

For project operations, this Skill uses [the workbench contract](.agents/skills/chengfeng-videocut-cut/references/shared/plugin-access.md) and probes actual CLI capabilities. Runtime >=0.5.9 alone does not guarantee every command; use a compatible workbench and an explicit project/endpoint. Other workbenches require their own verified adapter. Missing software is not silently downloaded.


## Preview limits

- File installation, host loading, Runtime compatibility and a completed video workflow are separate checks.
- `doctor` reports `runtime: not-checked` and `hostLoaded: not-checked`; these fields are not a claim of readiness.
- Windows and real host/business end-to-end validation are not claimed by this source preview.
- No cloud transcription, account access, telemetry, media upload or automatic Issue submission runs during installation.
- Source previews do not publish an entire workbench release or promise a one-command full suite.

## Package maintenance

The canonical method lives at `.agents/skills/chengfeng-videocut-cut/`. Shared contracts and installer/audit helpers are generated snapshots from the workbench, with provenance where applicable. See [PUBLISHING.md](PUBLISHING.md) for upload boundaries. In a source checkout run `npm run check:package` and `npm run check:upload` before a release. Tests of file boundaries do not replace manual content/license review.

This repository is intentionally `private: true` in package.json to prevent accidental npm-registry publication. GitHub installation and `npm pack --ignore-scripts` are still supported. For changes, open a focused Issue or pull request with a minimal reproducible example; never attach secrets or private media.

## Community & Support / 官方来源

Created and maintained by **成峰 / AI产品自由**.

- [This repository](https://github.com/Agentchengfeng/chengfeng-videocut-cut) and [GitHub Issues](https://github.com/Agentchengfeng/chengfeng-videocut-cut/issues): source, bugs and reproducible reports.
- [Author on GitHub](https://github.com/Agentchengfeng), [X](https://x.com/chengfeng240928): project updates.
- 小红书、公众号、B站、抖音 / 视频号：**AI产品自由**。Following is optional and never required for installation.
- Original Skill lineage: [chengfeng-videocut-skills](https://github.com/Agentchengfeng/chengfeng-videocut-skills); shared workbench: [chengfeng-videocut](https://github.com/Agentchengfeng/chengfeng-videocut).

## License

Project-owned material is provided under **Apache-2.0**. Preserve [LICENSE](LICENSE), [NOTICE.md](NOTICE.md), [CITATION.cff](CITATION.cff) and any bundled third-party notices. Upstream provenance does not imply endorsement. Runtime and optional animation assets have their own distribution and licensing scope.
