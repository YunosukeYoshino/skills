---
name: wp-env-cli
description: >
  wp-env環境でのWP-CLI操作スキル。プラグイン管理、DB操作、オプション設定、ユーザー管理、
  リライトルール、キャッシュ制御などをDockerコンテナ内で安全に実行する。
  Use PROACTIVELY when the user mentions "wp-cli", "wp-env", "WordPress設定",
  "プラグイン管理", "DB操作", "オプション変更", "ユーザー作成", "リライト",
  "WordPress管理", ".wp-env.json", "wp-envのプラグイン一覧",
  or any WordPress administration task via CLI.
  Do NOT trigger for Docker environment cleanup, disk space recovery,
  or container pruning — use docker-cleanup for those tasks.
---

# wp-env + WP-CLI Operations

wp-env（Docker）環境内でWP-CLIコマンドを安全に実行するスキル。

## Core Principle

**全てのWP-CLIコマンドは `wp-env run cli` 経由で実行する。**

このプロジェクトでは `bun run wp-env` でwp-envを管理しているため、
WP-CLIの実行は以下のパターンになる:

```bash
# 基本パターン
bun run wp-env run cli wp <command> [args...]

# 例
bun run wp-env run cli wp plugin list
bun run wp-env run cli wp option get blogname
```

`2>/dev/null` を付けてDockerのstderr出力を抑制すると見やすい:

```bash
bun run wp-env run cli wp plugin list 2>/dev/null
```

## Command Reference

### Plugin Management

```bash
# プラグイン一覧
bun run wp-env run cli wp plugin list --format=table 2>/dev/null

# 有効なプラグインのみ
bun run wp-env run cli wp plugin list --status=active --format=table 2>/dev/null

# プラグイン有効化
bun run wp-env run cli wp plugin activate <plugin-slug> 2>/dev/null

# プラグイン無効化
bun run wp-env run cli wp plugin deactivate <plugin-slug> 2>/dev/null

# プラグインインストール＋有効化（wp-env管理外のプラグイン追加時）
bun run wp-env run cli wp plugin install <plugin-slug> --activate 2>/dev/null
```

**注意**: このプロジェクトではプラグインは `.wp-env.json` の `plugins` 配列でURL管理している。
新しいプラグインを恒久的に追加する場合は `.wp-env.json` を編集すること。
`wp plugin install` はテスト目的や一時的な利用にのみ使う。

### Option Management

WordPress設定（wp_options テーブル）の読み書き。

```bash
# オプション取得
bun run wp-env run cli wp option get <key> 2>/dev/null

# JSON形式で取得（シリアライズされたオプション向け）
bun run wp-env run cli wp option get <key> --format=json 2>/dev/null

# オプション更新
bun run wp-env run cli wp option update <key> <value> 2>/dev/null

# JSON値で更新
bun run wp-env run cli wp option update <key> '<json>' --format=json 2>/dev/null

# シリアライズされたオプションの部分更新（patch）
bun run wp-env run cli wp option patch update <key> <sub-key> <value> --format=json 2>/dev/null

# オプション追加
bun run wp-env run cli wp option add <key> <value> 2>/dev/null

# オプション削除
bun run wp-env run cli wp option delete <key> 2>/dev/null

# オプション一覧（フィルタ付き）
bun run wp-env run cli wp option list --search="*yoast*" --format=table 2>/dev/null
```

#### Yoast SEO Options

Yoast SEOの主要オプション名:

| オプション名 | 用途 |
|---|---|
| `wpseo` | 一般設定 |
| `wpseo_titles` | タイトル・メタ・サイトマップ設定 |
| `wpseo_social` | ソーシャル（OGP）設定 |
| `wpseo_xml` | XMLサイトマップ設定 |

```bash
# Yoast全般設定を確認
bun run wp-env run cli wp option get wpseo --format=json 2>/dev/null | python3 -m json.tool

# 著者アーカイブ無効化
bun run wp-env run cli wp option patch update wpseo_titles disable-author true --format=json 2>/dev/null

# タクソノミーをnoindex
bun run wp-env run cli wp option patch update wpseo_titles noindex-tax-<taxonomy> true --format=json 2>/dev/null

# ページのメタディスクリプション設定テンプレート
bun run wp-env run cli wp option patch update wpseo_titles metadesc-page-<slug> "<description>" 2>/dev/null
```

### Database Operations

```bash
# DBエクスポート（プロジェクトのsql/ディレクトリへ）
bun run wp-contents:export
# 実体: bun run wp-env run cli wp db export sql/wpenv.sql

# DBインポート
bun run wp-contents:import
# 実体: wp-env run cli wp db reset --yes && wp-env run cli wp db import sql/wpenv.sql

# SQLクエリ直接実行
bun run wp-env run cli wp db query "SELECT * FROM wp_options WHERE option_name LIKE '%seo%'" 2>/dev/null

# DB検索置換（URLの移行などに）
bun run wp-env run cli wp search-replace '<old>' '<new>' --dry-run 2>/dev/null
bun run wp-env run cli wp search-replace '<old>' '<new>' 2>/dev/null
```

### User Management

```bash
# ユーザー一覧
bun run wp-env run cli wp user list --format=table 2>/dev/null

# ユーザー作成
bun run wp-env run cli wp user create <username> <email> --role=<role> 2>/dev/null

# パスワード更新
bun run wp-env run cli wp user update <user-id> --user_pass=<password> 2>/dev/null
```

### Post / Page Management

```bash
# 固定ページ一覧
bun run wp-env run cli wp post list --post_type=page --format=table 2>/dev/null

# 投稿一覧
bun run wp-env run cli wp post list --post_type=post --format=table 2>/dev/null

# カスタム投稿タイプ一覧
bun run wp-env run cli wp post list --post_type=<cpt> --format=table 2>/dev/null

# ページのメタ情報取得
bun run wp-env run cli wp post meta list <post-id> --format=table 2>/dev/null

# ページのメタ情報更新
bun run wp-env run cli wp post meta update <post-id> <key> <value> 2>/dev/null
```

### Rewrite Rules

```bash
# リライトルール再生成
bun run wp-env run cli wp rewrite flush 2>/dev/null

# パーマリンク構造変更
bun run wp-env run cli wp rewrite structure '/%postname%/' 2>/dev/null

# リライトルール一覧
bun run wp-env run cli wp rewrite list --format=table 2>/dev/null
```

### Cache Management

```bash
# オブジェクトキャッシュクリア
bun run wp-env run cli wp cache flush 2>/dev/null

# トランジェントキャッシュ削除
bun run wp-env run cli wp transient delete --all 2>/dev/null
```

### Theme Management

```bash
# テーマ一覧
bun run wp-env run cli wp theme list --format=table 2>/dev/null

# テーマ有効化
bun run wp-env run cli wp theme activate <theme-slug> 2>/dev/null
```

### Taxonomy / Term Management

```bash
# タクソノミー一覧
bun run wp-env run cli wp taxonomy list --format=table 2>/dev/null

# ターム一覧
bun run wp-env run cli wp term list <taxonomy> --format=table 2>/dev/null

# ターム作成
bun run wp-env run cli wp term create <taxonomy> '<term-name>' 2>/dev/null
```

### Scaffold & Maintenance

```bash
# PHPファイル実行
bun run wp-env run cli wp eval-file wp-content/themes/wpenv/<path-to-file> 2>/dev/null

# PHP式を評価
bun run wp-env run cli wp eval '<php-code>' 2>/dev/null

# サイト情報
bun run wp-env run cli wp site url 2>/dev/null
bun run wp-env run cli wp option get siteurl 2>/dev/null
bun run wp-env run cli wp option get home 2>/dev/null

# WordPress/PHP バージョン
bun run wp-env run cli wp core version 2>/dev/null
bun run wp-env run cli wp cli version 2>/dev/null
```

## wp-env Lifecycle

```bash
# 起動
bun run wp-start
# 実体: wp-env start

# 停止
bun run wp-stop
# 実体: wp-env stop

# 完全リセット（DB含む全データ削除）
bun run wp-clean
# 実体: wp-env clean all
```

## Safety Rules

### Destructive Commands (確認必須)

以下のコマンドはデータ消失のリスクがある。実行前に必ずユーザーに確認する:

- `wp db reset` - DB全削除
- `wp db import` - DB上書き
- `wp search-replace` - DB内一括置換（`--dry-run` を先に実行）
- `wp plugin uninstall` - プラグインデータ含めて削除
- `wp site empty` - コンテンツ全削除
- `wp post delete` - 投稿削除

### Safe Commands (確認不要)

以下は読み取り専用または安全な操作:

- `wp * list` - 一覧表示系
- `wp * get` - 取得系
- `wp option get/list` - オプション読み取り
- `wp cache flush` - キャッシュクリア
- `wp rewrite flush` - リライト再生成
- `wp plugin activate/deactivate` - 有効化/無効化

### Best Practices

1. **DB変更前はエクスポート**: `bun run wp-contents:export` でバックアップ
2. **search-replaceは--dry-run**: 本番前に必ずドライランで影響範囲確認
3. **オプション変更前はget**: 現在値を確認してから更新
4. **JSON出力を活用**: `--format=json` + `python3 -m json.tool` でパース

## Project-Specific Notes

### Mappings (.wp-env.json)

```json
{
  "mappings": {
    "sql": "./sql",
    "wp-content/uploads": "./wordpress/uploads",
    "wp-content/themes/wpenv/": "./wordpress/themes"
  }
}
```

- テーマは `wordpress/themes/wp-theme/` がアクティブテーマ
- `wp eval-file` のパスは `wp-content/themes/wpenv/wp-theme/<file>` になる
- SQLダンプは `sql/wpenv.sql` に保存

### Plugin Management

プラグインは `.wp-env.json` の `plugins` 配列でWordPress.orgのURLとして管理。
`wordpress/plugins/` ディレクトリは `.gitignore` で除外済み。
新しいプラグインを追加するときは `.wp-env.json` を編集し、`wp-env start` で反映。
