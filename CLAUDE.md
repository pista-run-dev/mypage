# CLAUDE.md — ぴすた / Pista パーソナルリンクハブ

このファイルは、このコードベースで作業する AI アシスタント（Claude など）向けのガイドラインです。

## プロジェクト概要

日本人ランニング愛好家向けのゼロ依存パーソナルリンクハブ（Linktree スタイル）です。純粋な HTML・CSS・バニラ JavaScript で構築されており、フレームワーク・ビルドツール・パッケージマネージャは一切使用していません。

**公開サイト**: https://pista-run-dev.github.io/mypage/
**デプロイ方法**: GitHub Pages（`main` にプッシュ → 自動デプロイ）

---

## リポジトリ構成

```
mypage/
├── index.html        # シングルページアプリのエントリーポイント（日本語、lang="ja"）
├── style.css         # 全スタイル — デザイントークン・テーマ・レイアウト・コンポーネント
├── sw.js             # オフライン/PWA キャッシュ用 Service Worker
├── manifest.json     # PWA マニフェスト（インストール可能アプリのメタデータ）
├── avatar.jpg        # プロフィール写真
├── icons/
│   ├── apple-touch-icon.png   # iOS ホーム画面アイコン
│   ├── icon-192.png           # Android/PWA アイコン
│   └── icon-512.png           # Android/PWA アイコン（マスカブルとしても使用）
└── README.md
```

**ビルドステップはありません**。ファイルは静的アセットとして直接配信されます。明示的に要求されない限り、ビルドパイプラインを導入しないでください。

---

## アーキテクチャ & 主要パターン

### テーマシステム

CSS 属性ベースのテーマシステムを採用しています：

- `<html data-theme="dark">` または `<html data-theme="light">` — デフォルトは `dark`
- CSS 変数は `style.css` 内の `[data-theme="dark"]` と `[data-theme="light"]` ブロックにスコープされています
- テーマ設定は `localStorage.getItem('theme')` / `localStorage.setItem('theme', ...)` で永続化
- `<meta name="theme-color">` タグ（id=`meta-theme-color`）はテーマ切替時に更新：dark → `#050510`、light → `#f2f2f7`
- テーマ切替ボタンは右下に固定（`position: fixed; bottom: ...; right: 20px`）

`prefers-color-scheme` メディアクエリは**使用しないでください** — サイトは `data-theme` 属性のみに依存しています。

### CSS デザイントークン

共有値はすべて `:root` の CSS カスタムプロパティとして定義されています（`style.css:15–36`）：

| 変数 | 値 | 用途 |
|---|---|---|
| `--gap` | `10px` | グリッド/カードの間隔 |
| `--r-card` | `22px`（モバイル）、`26px`（≥600px） | カードの角丸 |
| `--r-icon` | `12px` | ブランドアイコンバッジの角丸 |
| `--max-w` | `480px` | コンテンツの最大幅 |
| `--glass-blur` | `16px` | グラスモーフィズム用バックドロップブラー |
| `--blue` | `#0a84ff` | プライマリアクセント |
| `--purple` | `#bf5af2` | セカンダリアクセント |
| `--teal` | `#64d2ff` | ターシャリアクセント |

テーマスコープ変数（`--bg`、`--surface`、`--text`、`--text-2`、`--text-3`、`--border`、`--card-shadow`、`--card-inset`、`--mesh-opacity`）は `[data-theme="dark"]` と `[data-theme="light"]` ブロック内で別々に定義されています。

**必ずこれらの変数を使用してください** — `:root` に新しいブランドカラー定数を追加する場合を除き、新しいルールに16進数カラーをハードコードしないでください。

### Bento グリッドレイアウト

メインレイアウトは名前付きテンプレートエリアを持つ CSS Grid を使用しています（`style.css:136–150`）：

**モバイル（デフォルト、2カラム）:**
```
profile   profile
instagram x
note      runtrip
strava    youtube
spotify   spotify
teaser    teaser
```

**タブレット ≥600px（3カラム）:**
```
profile   profile   profile
instagram x         note
runtrip   strava    youtube
spotify   teaser    teaser
```

各カードは `.card--{platform}` クラスでグリッドエリアが割り当てられています。新しいカードを追加する場合は、名前付きグリッドエリアを割り当て、両方の `grid-template-areas` ブロックを更新してください。

### グラスモーフィズムカードパターン

すべてのカードは `.card` ベースクラスを共有しています（`style.css:152–163`）：
- `background: var(--surface)` — 半透明サーフェス
- `backdrop-filter: blur(var(--glass-blur))`（`-webkit-` プレフィックス付き）
- `border: 1px solid var(--border)`
- `box-shadow: var(--card-shadow), var(--card-inset)` — 外側シャドウ + インセットハイライト
- `cardEnter` キーフレームと `:nth-child()` による遅延（0.08s 刻み）で段階的な登場アニメーション

### カードコンポーネントの種類

1. **`.card--profile`** — 非インタラクティブ、アバター・自己紹介・自己ベストを表示
2. **`.card--link`** — `<a>` タグ、ホバー/アクティブ状態、プラットフォームへの外部リンク
3. **`.card--teaser`** — 非インタラクティブ、アニメーションバッジ付きで開発中アプリを告知

### リンクカードの構造

```html
<a class="card card--link card--{platform}"
   href="..."
   target="_blank"
   rel="noopener noreferrer"
   aria-label="{Platform} を開く">
  <div class="link-top">
    <span class="brand-icon">
      <svg ...></svg>
    </span>
    <span class="link-arrow" aria-hidden="true">↗</span>
  </div>
  <h2 class="link-name">{プラットフォーム名}</h2>
  <p class="link-desc">{日本語の説明}</p>
</a>
```

### ブランドアイコンカラー

各プラットフォームは `:root` にカラー変数と対応する `.card--{platform} .brand-icon` ルールを持ちます。ブランド固有のホバーグロー効果は、`[data-theme="dark/light"] .card--{platform}:hover` セレクターを使ってダーク・ライトテーマで別々に定義されています。

現在のプラットフォームとカラー変数：
- `--c-instagram` — グラデーション（135deg、オレンジ → ピンク → マゼンタ）
- `--c-x` — `#000`（黒、ボーダー付き）
- `--c-note` — `#41c9b4`（ティール）
- `--c-runtrip` — `#3AB483`（グリーン）；白背景のアイコンバッジを使用
- `--c-strava` — `#FC4C02`（オレンジ）
- `--c-youtube` — `#FF0000`（レッド）
- `--c-spotify` — `#1DB954`（グリーン）

### アニメーション

| キーフレーム | 時間 | 用途 |
|---|---|---|
| `cardEnter` | 0.6s | カード登場（translateY + scale、段階的） |
| `meshShift` | 12s 無限ループ交互 | 背景メッシュグラデーションの動き |
| `blink` | 2s 無限ループ | ティーザーバッジの点滅ドット |

イージング規約：動きには `cubic-bezier(0.22, 1, 0.36, 1)`、カラー/透明度のトランジションには `ease`。

### Service Worker（sw.js）

キャッシュ名：`pista-v1`。戦略：**キャッシュファースト + バックグラウンド再検証**（stale-while-revalidate パターン）。

- `install`：`/mypage/`・`index.html`・`style.css`・`manifest.json`・両アイコンを事前キャッシュ
- `activate`：現在の `CACHE_NAME` 以外のすべてのキャッシュを削除
- `fetch`：キャッシュがあれば即座に返し、同時にネットワークから取得してキャッシュを更新

**新しいアセットを追加する場合**、オフラインで動作させるには `sw.js` の `ASSETS` 配列にパスを追加し、キャッシュバージョンを更新してください（`pista-v1` → `pista-v2`）。

---

## CSS 命名規則

- **ブロック**: `.card`、`.bento`、`.footer`、`.theme-toggle`
- **モディファイア**（ダブルダッシュ）: `.card--profile`、`.card--link`、`.card--teaser`、`.card--instagram`
- **エレメント**（ダブルアンダースコア）: `.pb__label`、`.pb__time`、`.teaser-badge__dot`
- **状態/ユーティリティ**: `.dot--active`
- クラス名はすべてケバブケース

---

## HTML 規則

- 言語：`lang="ja"`（日本語）
- デフォルトテーマ：`<html>` に `data-theme="dark"`
- すべての外部リンクに `target="_blank"` と `rel="noopener noreferrer"` が必要
- すべてのインタラクティブ要素に日本語の `aria-label` が必要（例：`aria-label="Instagram を開く"`）
- 装飾用 SVG アイコンには `aria-hidden="true"` を使用
- 見出し階層：プロフィール名に `<h1>`、カードタイトルに `<h2>`
- SVG アイコンはインライン埋め込み（外部アイコンライブラリは使用しない）

---

## JavaScript 規則

- バニラ JS のみ — ライブラリ・フレームワーク不使用
- テーマスクリプトは IIFE パターンを使用し、`</body>` の後（ファイル末尾）に配置
- Service Worker の登録は `</body>` の前にシンプルなインラインスクリプトで記述
- `getElementById` / `setAttribute` による直接 DOM 操作
- モジュールシステムなし — `import`/`export` 不使用

---

## レスポンシブブレークポイント

| ブレークポイント | 動作 |
|---|---|
| デフォルト（モバイル） | 2カラム Bento グリッド、`--r-card: 22px` |
| `min-width: 600px` | 3カラム Bento グリッド、`--r-card: 26px` |
| `max-width: 340px` | プロフィールカードが縦並びになり、名前が折り返される |

`100svh`（セーフビューポート高さ）と `env(safe-area-inset-bottom)` を使用してノッチ/ホームバーの安全領域に対応しています。

---

## デプロイ

`main` ブランチにプッシュ → GitHub Pages が自動デプロイ。

- ビルドステップ不要
- CI/CD パイプラインなし — デプロイは直接実行
- PWA スコープは `/mypage/` — `sw.js` と `manifest.json` 内のすべてのアセットパスはこのプレフィックスが必要

---

## 新しいソーシャルカードの追加手順

1. **CSS**: `style.css` の `:root` に `--c-{platform}: ...` カラー変数を追加
2. **CSS**: `.card--{platform} .brand-icon { background: var(--c-{platform}); }` ルールを追加
3. **CSS**: ダーク・ライトテーマのホバーグロールールを追加
4. **CSS**: `.card--{platform} { grid-area: {platform}; }` ルールを追加
5. **CSS**: モバイルと ≥600px の両方の `grid-template-areas` ブロックに `{platform}` を追加
6. **HTML**: 正しい構造で `<a class="card card--link card--{platform}">` ブロックを追加
7. **sw.js**: 新しいアセットファイルを追加しない限り変更不要

---

## 既知の問題 / メモ

- YouTube と Spotify のカードは現在 `href="#"` のまま（プレースホルダー、実際のプロフィールに未接続）
- コミットは SSH キー（`/home/claude/.ssh/commit_signing_key.pub`）で GPG 署名されています
