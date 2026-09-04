# claude-config

Claude Code のオーケストレーター委譲設定の正本。各リポジトリは `.claude/agents` に git submodule として取り込む。

## 中身

- `researcher.md` / `designer.md` / `implementer.md` / `reviewer.md` — サブエージェント定義（`.claude/agents/` 直下に置かれるため、ルート直下にある）
- `rules/orchestration.md` — 委譲方針（各リポジトリの `CLAUDE.md` から `@.claude/agents/rules/orchestration.md` で import する）

## リポジトリへの導入

```bash
# .gitignore で .claude/* を無視している場合は先に !.claude/agents/ を追加する
git submodule add <このリポジトリの URL> .claude/agents
```

`CLAUDE.md` に次の 1 行を追加する:

```
@.claude/agents/rules/orchestration.md
```

Claude Code on the web では、環境設定の setup script に `git submodule update --init --recursive` を入れる。

## 更新の流れ

1. このリポジトリで直して `main` に push する。
2. 各リポジトリで `git submodule update --remote .claude/agents` を実行し、ポインタの変更をコミットする。

clone 直後に submodule が空なら `git submodule update --init`。
