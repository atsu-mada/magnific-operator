# magnific-operator

An agent skill for Claude Code and Codex that operates the [Magnific](https://magnific.ai) AI creative suite through its **MCP server** (no browser automation). It is a *suite operator*: it takes a prepared generation handoff and executes it on Magnific with a cost quote and user approval before every paid step.

## What it does

- Image generation and edits, video generation (including **Seedance 2.5 via Magnific** with image / video / audio references or start/end keyframes)
- Image and video **upscaling** (mode choice, enum and output-size pitfalls, reuse of existing upscales)
- Local file **uploads** as creations, **Spaces** canvas build/run
- **Cost estimation** (`simulate_cost`, known estimate gaps) and a quote → approve → run gate
- Moderation-block handling, frame-matched **partial regeneration**, and versioned, verified downloads

It does **not** write general Seedance prompt craft, plan productions, build previz, click the Magnific web UI, delete or share content, or spend credits without explicit approval.

Numbers in this skill (rates, limits, gaps) were observed in 2026-09 — verify live.

## Install

### Claude Code (plugin)

```
/plugin marketplace add atsu-mada/magnific-operator
/plugin install magnific-operator@atsu-mada-magnific-operator
```

### Claude Code (manual)

```bash
git clone https://github.com/atsu-mada/magnific-operator.git ~/.claude/skills/magnific-operator
```

Invoke with `/magnific-operator`.

### Codex

```bash
git clone https://github.com/atsu-mada/magnific-operator.git ~/.codex/skills/magnific-operator
```

Invoke with `$magnific-operator`.

## Requirements

- A Magnific account with the **Magnific MCP server connected** in your client. Without it the skill stops and asks you to connect it.
- `ffmpeg` / `ffprobe` and `curl` for downloads, verification, and splicing.

## Related skills

| Skill | Role |
|---|---|
| [ai-video-production](https://github.com/atsu-mada/ai-video-production) | Preproduction; its `generation-handoff` (with `suite: magnific-operator`) is this skill's input; hosts the suite-operator registry |
| [previz-maker](https://github.com/atsu-mada/previz-maker) | Block previz passed as `@Video 1` — only after you approve it visually |
| [seedance-studio](https://github.com/atsu-mada/seedance-studio) | Seedance prompt craft, filter vocabulary, retake protocol |

Magnific-specific tool behaviour lives here; general Seedance knowledge lives in seedance-studio.

## License

[MIT](LICENSE)

---

## 日本語

Magnific の MCP サーバー経由で、画像・動画生成（Magnific 上の Seedance 2.5 を含む）、アップスケール、アップロード、Spaces キャンバス、コスト見積もりを実行する Claude Code / Codex 向けスキルです。Chrome は使いません。有料処理の前には必ず見積もりを提示し、ユーザーの承認を得てから実行します。プレビズを参照に使う場合は、ユーザーが映像を確認して承認していることが前提です。前工程は ai-video-production、プレビズは previz-maker、プロンプト作法は seedance-studio が担当します。数値は 2026-09 時点の観測値なので、実行前に最新の値を確認してください。
