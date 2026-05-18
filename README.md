# X Clone

シンプルなつぶやき投稿サービス。Supabase + 素のHTML/JS + GitHub Pages で動作します。

## セットアップ手順

### 1. Supabase プロジェクトを作成

[supabase.com](https://supabase.com) でプロジェクトを作成し、以下のSQLをSupabase SQL Editorで実行してください。

```sql
CREATE TABLE posts (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  user_email TEXT,
  content TEXT NOT NULL CHECK (char_length(content) <= 280),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

CREATE POLICY "誰でも投稿を読める" ON posts
  FOR SELECT USING (true);

CREATE POLICY "ログイン済みユーザーは自分の投稿を作成できる" ON posts
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "ユーザーは自分の投稿を削除できる" ON posts
  FOR DELETE USING (auth.uid() = user_id);
```

### 2. Supabase の設定値を入力

`index.html` と `feed.html` の以下の行を書き換えてください：

```js
const SUPABASE_URL = 'YOUR_SUPABASE_URL'      // ← Project URL
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY'  // ← anon/public キー
```

これらの値は **Project Settings → API** から確認できます。

> **注意**: `anon` キーはクライアントサイドでの使用を想定した公開キーです。
> `service_role` キーは絶対にここに書かないでください。

### 3. メール確認を無効化（開発用）

Supabase Dashboard → Authentication → Settings → "Enable email confirmations" をOFFにすると、確認メールなしでサインアップできます。

### 4. GitHub Pages にデプロイ

1. GitHubにこのリポジトリをpush
2. Settings → Pages → Source: "Deploy from branch" → `main` / `/(root)`
3. 発行されたURL（例: `https://yourname.github.io/x-clone/`）をコピー

### 5. Supabase にサイトURLを登録

Supabase Dashboard → Authentication → URL Configuration:

- **Site URL**: `https://yourname.github.io/x-clone/`
- **Redirect URLs**: `https://yourname.github.io/x-clone/feed.html`

## キーの取り扱い

| キー | 種類 | GitHubへのコミット |
|---|---|---|
| `SUPABASE_URL` | 公開 | OK（クライアント用） |
| `SUPABASE_ANON_KEY` | 公開 | OK（RLSで保護） |
| `SUPABASE_SERVICE_ROLE_KEY` | **シークレット** | **絶対禁止** |
