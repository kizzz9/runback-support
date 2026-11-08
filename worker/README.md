# Runback お問い合わせ Worker

GitHub Pages のサポートページのフォーム送信を受け、Resend で開発者へメール転送する Cloudflare Worker。

- サイト本体は GitHub Pages のまま（この Worker は送信の受け口だけ）
- 宛先メールアドレスは Worker のシークレット（`CONTACT_TO`）に入れるだけで、ページにもコードにも出ない
- スパム対策: ハニーポット + Origin チェック（+ Turnstile は任意で後付け可）

## 構成

```
ブラウザ (kizzz9.github.io) ─POST─▶ Worker ─▶ (任意: Turnstile 検証)
                                        └─▶ Resend API ─▶ 開発者の Gmail
```

## 現在の状態（2026-11-08 作成）

- デプロイ済み: `https://moodvault-contact.nsmtkz9.workers.dev`
- シークレット登録済み: `CONTACT_TO`（届け先 Gmail）, `RESEND_API_KEY`（Resend 送信専用キー）
- Turnstile: **未設定（任意）**。`TURNSTILE_SECRET` が未設定なら Worker は検証をスキップする
- 送信元: `onboarding@resend.dev`（Resend 共有ドメイン）。宛先は Resend 登録アドレス（= `CONTACT_TO`）のみ許可される無料枠

## 運用コマンド

wrangler は API トークンで認証する。トークンは `worker/.cf_token`（gitignore 済み）に置き、各コマンドで読み込む:

```bash
cd worker
export CLOUDFLARE_API_TOKEN="$(cat .cf_token)"
export CLOUDFLARE_ACCOUNT_ID="93a6265a7aa9a36837fa7c06a43ea51c"

wrangler deploy            # 再デプロイ
wrangler secret list       # シークレット確認
wrangler tail              # ライブログ
```

## Turnstile を後から有効化する場合

1. Cloudflare ダッシュボード → Turnstile → Add widget（Domain: `kizzz9.github.io` / Managed）
2. `wrangler secret put TURNSTILE_SECRET`（Secret Key を入力）
3. `index.html` / `en/index.html` の Turnstile 用コメント（`<script ...turnstile...>` と `<div class="cf-turnstile" ...>`）を外し、`data-sitekey` を Site Key に置換
4. GitHub Pages を push

## メール到達性の改善（任意・将来）

独自ドメイン（例 `moodvault.app`）を取得して Resend で verify すると、送信元を `support@moodvault.app` にでき、迷惑メール判定されにくくなる。
