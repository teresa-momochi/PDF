# 教材 PDF 庫 — 設定與部署說明

一個極簡的網站:老師上傳一次 PDF,之後任何電腦打開網址就能看到清單並開啟。
檔案實際存在 Supabase Storage,網站本身只放在 GitHub Pages(純靜態，免費）。

---

## 一、Supabase 設定(約 5 分鐘)

### 1. 建立專案
前往 https://supabase.com → 建立一個新專案（免費方案即可）。

### 2. 建立資料表
進入 Supabase 後台 → **SQL Editor** → 貼上並執行：

```sql
create table pdfs (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  storage_path text not null,
  url text not null,
  size bigint,
  created_at timestamptz default now()
);

-- 開放讀寫（因為只用簡單密碼保護前端，這裡用最簡單的開放權限）
alter table pdfs enable row level security;

create policy "public read" on pdfs
  for select using (true);

create policy "public insert" on pdfs
  for insert with check (true);

create policy "public delete" on pdfs
  for delete using (true);
```

### 3. 建立 Storage Bucket
後台 → **Storage** → New bucket：
- 名稱：`pdfs`
- **Public bucket：打勾（開啟）**

建立後，進入該 bucket → Policies，新增允許上傳/刪除的政策（或直接用 Storage 預設的 public 政策）。最簡單做法：bucket 設為 public 後，在 Policies 分別新增：
- `SELECT`：allow all
- `INSERT`：allow all
- `DELETE`：allow all

> 因為上傳頁面已經有密碼保護擋在前端，這裡的資料庫/儲存權限不需要做得太複雜。

### 4. 取得連線資訊
後台 → **Project Settings → API**，複製：
- `Project URL`
- `anon public` key

---

## 二、修改 config.js

打開 `config.js`，填入剛剛複製的資訊：

```js
const SUPABASE_URL = "https://xxxxxxxx.supabase.co";
const SUPABASE_ANON_KEY = "你的 anon public key";
const UPLOAD_PASSWORD = "改成你自己的密碼";
const BUCKET_NAME = "pdfs";
```

---

## 三、部署到 GitHub Pages

1. 在 GitHub 建立一個新的 repository（例如 `pdf-library`）
2. 把這個資料夾裡的三個檔案（`index.html`、`upload.html`、`style.css`、`config.js`）上傳上去
3. 進入 repo → **Settings → Pages**
4. Source 選擇 `main` branch、根目錄 `/`，儲存
5. 等 1-2 分鐘，網址會出現在同一頁面，格式類似：
   `https://你的帳號.github.io/pdf-library/`

之後跨電腦只要打開這個網址：
- `index.html`：瀏覽 / 開啟所有 PDF（不需密碼）
- `upload.html`：上傳新 PDF 或刪除（需要密碼）

---

## 四、使用方式

- **上傳**：打開 `.../upload.html`，輸入密碼 → 選擇 PDF 檔案 → 可自訂顯示名稱 → 點「上傳」
- **瀏覽/開啟**：打開首頁 `.../`，點任一筆「開啟」即可在新分頁看 PDF
- **刪除**：在上傳頁面的清單中點「刪除」

---

## 五、之後可以擴充的方向（目前刻意不做，保持簡單）

- 分類 / 資料夾
- 搜尋
- 多人帳號權限（目前是共用一組密碼）
- 縮圖預覽

如果之後想把這個功能整合進「教材魔法書」，可以把 `pdfs` 這張表和 storage bucket 直接沿用，再加上目前計畫中的 BookRepository 架構即可。
