# claude-config

Claude Code のオーケストレーター委譲設定（サブエージェント定義 + 委譲方針）の正本。各リポジトリは `.claude/agents` に git submodule として取り込む。

## なぜ submodule か

Claude Code on the web はリポジトリにコミットされたファイルしか読まない（`~/.claude/` のユーザースコープ設定はクラウド VM に存在しない）。複数リポジトリで同じ設定を使うため、正本をここに 1 つだけ置き、各リポジトリからは submodule で参照する。コピー配布はしない。

## 構成

| パス | 役割 |
|---|---|
| `researcher.md` `designer.md` `implementer.md` `reviewer.md` | サブエージェント定義。取り込み先で `.claude/agents/` 直下に置かれる必要があるため、ルート直下にある |
| `rules/orchestration.md` | 委譲方針。取り込み先の `CLAUDE.md` から import する |
| `README.md` | このファイル。`.claude/agents/` に frontmatter 無しの md があってもエージェント読込は壊れない（検証済み） |

サブディレクトリ内の md はエージェントとして読まれない（検証済み）。方針文書を `rules/` に置いているのはそのため。

## 導入手順

### 1. submodule を追加する

```bash
# .gitignore が .claude/* を無視している場合は、先に例外を追加する。
# 末尾スラッシュ無しで書くこと。`!.claude/agents/` だと未作成ディレクトリに効かず submodule add が拒否される。
echo '!.claude/agents' >> .gitignore

git submodule add <このリポジトリの HTTPS URL> .claude/agents
```

SSH ではなく HTTPS を使う。web の VM は HTTPS でしか取得できない。

### 2. CLAUDE.md から方針を import する

`CLAUDE.md` に次の 1 行を置く（相対パスは CLAUDE.md の位置基準）。

```
@.claude/agents/rules/orchestration.md
```

### 3. Claude Code on the web の環境設定に setup script を登録する

web の VM は submodule を自動では初期化しない。環境設定（Environment settings）の setup script に次を保存する。

```bash
set -e
for d in "$HOME"/*/ /home/user/*/; do
  if [ -d "$d/.git" ] || [ -f "$d/.git" ]; then
    (cd "$d" && git submodule update --init --recursive)
  fi
done
```

注意点:

- setup script はリポジトリの **1 つ上のディレクトリ**で実行される。`git submodule update` を直接書くと「not a git repository」で失敗する。上のループ形式はリポジトリを探して `cd` するので、どの環境でも同じ内容で使える。
- 環境ごとの設定なので、セッションを開くときにその環境が選ばれていることを確認する。
- submodule を持たないリポジトリでは何もせず正常終了する。

### 4. 動作確認

新規セッションで次を確認する。

- `git submodule status` の先頭に `-` が付いていない（未初期化なら `-` が付く）
- Agent ツールの一覧に researcher / designer / implementer / reviewer がある
- 「オーケストレーション方針」の §0〜§6 が読み込まれている

## 更新の流れ

1. このリポジトリで直して `main` に push する。編集は独立した clone で行う（取り込み先の `.claude/agents/` 内を直接編集しない）。
2. 各リポジトリで `git submodule update --remote .claude/agents` を実行し、ポインタの変更をコミットして push する。

## 制約

- **エージェント定義はセッション起動時にしか読まれない。** submodule を後から init しても既存セッションには反映されない。新規セッションを開く。
- **public リポジトリである。** web のプロキシ認証はセッション対象のリポジトリにしか付かず、private submodule は clone できない（検証済み）。そのため public にしている。**本名・アカウント名・メールアドレス・所有リポジトリ名・秘密情報をこのリポジトリに書かない。** コミット author は GitHub の noreply アドレスを使う。
- 取り込み先ごとにポインタ更新のコミットが必要になる。web はコミット済みの gitlink しか見ないため、これは避けられない。
- `.claude/agents/` の再帰読込と `effort` frontmatter は公式ドキュメントに記載が無い。前者は「読まれない」ことを実測で確認しているが、Claude Code の更新で挙動が変わる可能性がある。
