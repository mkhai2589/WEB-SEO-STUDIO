# cau-truc-thu-muc.md

# Cau Truc Thu Muc – Local SEO Studio 2026

## Tong quan

Ung dung desktop chay 100% local, dung Tauri v2 lam wrapper.
Frontend: HTML5 + TailwindCSS v3 + Vanilla ES6.
Database: Dexie.js (IndexedDB).

---

## Cay Thu Muc Day Du

local-seo-studio/
│
├── index.html # Entry point chinh, layout nen, routing SPA
│
├── public/ # Static assets, thu vien tu host
│ ├── tinymce/ # TinyMCE 6.8+ self-hosted (giai nen tu goi npm)
│ │ ├── tinymce.min.js
│ │ └── plugins/ # image, link, code, table, lists, advlist
│ ├── favicon.ico
│ └── logo.svg
│
├── src/
│ │
│ ├── db/ # Tang du lieu – Dexie.js
│ │ ├── khoi-tao-db.js # Khoi tao instance Dexie, export db
│ │ └── schema-dexie.js # Dinh nghia tat ca store va index
│ │
│ ├── modules/ # Cac module chinh cua ung dung
│ │ │
│ │ ├── content/ # Module Viet Content SEO
│ │ │ ├── content-editor.js # TinyMCE init, floating toolbar, Zen Mode logic
│ │ │ ├── content-form.js # H1, slug, meta desc, focus keyword, status, image
│ │ │ ├── seo-panel.js # Right panel: diem SEO, accordion, checklist, AIO tab
│ │ │ ├── aio-preview.js # Preview giao dien Google AI Overview
│ │ │ └── cluster-suggest.js # Goi y cluster tu dong (debounce 5s)
│ │ │
│ │ ├── topic/ # Module Quan ly Topic/Pillar/Cluster/Entity
│ │ │ ├── topic-sidebar.js # Accordion da cap: Topic > Pillar > Cluster
│ │ │ ├── keyword-kanban.js # Kanban 3 cot: Chua dung / Dang dung / Da publish
│ │ │ ├── entity-list.js # Bang entity: ten, loai, tan suat
│ │ │ └── topic-actions.js # Tao topic moi, import CSV/Excel, export
│ │ │
│ │ ├── graph/ # Module Internal Link Graph
│ │ │ ├── link-graph.js # vis-network init, render, filter, context menu
│ │ │ ├── graph-drawer.js # Right drawer slide-in khi click node
│ │ │ └── link-suggest.js # Logic de xuat link dua tren similarity
│ │ │
│ │ ├── notes/ # Module Ghi Chu Da Dang
│ │ │ ├── notes-list.js # Sidebar danh sach note, tim kiem, filter
│ │ │ ├── note-editor.js # EasyMDE cho note thuong
│ │ │ ├── case-study-form.js # Form 4 textarea + drag drop anh
│ │ │ ├── process-editor.js # Danh sach buoc SortableJS
│ │ │ └── vscode-bridge.js # Goi Tauri invoke('open_in_vscode', ...)
│ │ │
│ │ └── dashboard/ # Module Dashboard & Bao cao
│ │ ├── dashboard-widgets.js # Render cac widget, drag drop sap xep
│ │ ├── charts-init.js # Chart.js: line chart SEO score, pie chart cluster
│ │ └── report-export.js # Xuat CSV toan bo, tao backup, restore
│ │
│ ├── utils/ # Utility – logic nang chay trong Worker
│ │ ├── seo-score-worker.js # Web Worker: tinh diem SEO 2026 (100 diem)
│ │ ├── tfidf-worker.js # Web Worker: TF-IDF, cosine similarity
│ │ ├── readability-vn.js # Kiem tra do doc tieng Viet (cau, tu trung binh)
│ │ ├── entity-detect.js # Phat hien entity tu tu dien DB + compromise.js
│ │ ├── duplicate-checker.js # Shingling k=9 + cosine, tra ve duplicateRisk 0-1
│ │ ├── intent-classifier.js # Phan loai intent: Informational/Commercial/Transactional
│ │ ├── faq-generator.js # Parse H2/H3 sinh FAQ + schema FAQPage
│ │ ├── schema-generator.js # Xuat JSON-LD: Article, FAQPage, HowTo, Author
│ │ ├── competitor-gap.js # TF-IDF so sanh doi thu, tim gap
│ │ └── auto-backup.js # Tu dong backup moi 5 phut hoac sau 10 thay doi
│ │
│ ├── ui/ # Thanh phan giao dien chung
│ │ ├── components/
│ │ │ ├── modal.js # Generic modal (dong bang ESC)
│ │ │ ├── toast.js # Thong bao nhanh (success/error/warning)
│ │ │ ├── sidebar.js # Sidebar trai collapsible
│ │ │ ├── header.js # Header: dark mode toggle, lang switch, ten module
│ │ │ └── context-menu.js # Right-click context menu (dung trong graph)
│ │ ├── hotkeys.js # Ctrl+S, Ctrl+K, Ctrl+E, Ctrl+Shift+F, F11
│ │ ├── zen-mode.js # An/hien sidebar + right panel, phim Esc
│ │ └── i18n.js # Dich ngon ngu VI/EN, luu preference vao Dexie
│ │
│ └── main.js # Khoi tao app, routing SPA, lazy load module
│
├── src-tauri/ # Tauri v2 backend (Rust)
│ ├── src/
│ │ └── main.rs # Entry Rust: command open_in_vscode, save_image, backup
│ ├── Cargo.toml # Dependencies Rust
│ ├── tauri.conf.json # Cau hinh Tauri: app name, permissions, bundle
│ └── icons/ # Icon ung dung (.ico, .icns, .png)
│
├── backups/ # Thu muc backup tu dong (Tauri tao, git ignore)
│ └── .gitkeep
│
├── output/ # File xuat: WordPress HTML, Schema JSON-LD
│ └── .gitkeep
│
├── package.json # npm scripts: dev, build, tauri dev, tauri build
├── vite.config.js # Vite config (optional, neu dung Vite)
├── tailwind.config.js # TailwindCSS v3 config
├── postcss.config.js # PostCSS (Tailwind + Autoprefixer)
│
├── cau-truc-thu-muc.md # File nay (tai lieu cau truc)
├── ban-dac-ta-ky-thuat.md # Dac ta ky thuat goc
└── .gitignore

---

## Ghi chu quan trong

### Quy tac dat ten file

- Tat ca ten file: **tieng Viet khong dau, chu thuong, ngan noi bang gach ngang**
- Vi du: `seo-score-worker.js`, `khoi-tao-db.js`, `entity-detect.js`

### Luong du lieu chinh

TinyMCE editor
→ debounce 800ms
→ seo-score-worker.js (Web Worker)
→ cap nhat seo-panel.js
→ luu vao Dexie contents
→ tinh lai similarities (tfidf-worker.js)

### Mo hinh Tauri API (thay Node.js)

| Hanh dong | Tauri Command                              |
| --------- | ------------------------------------------ |
| Mo VSCode | `invoke('open_in_vscode', {path})`         |
| Luu anh   | `invoke('save_image', {data, filename})`   |
| Backup DB | `invoke('write_backup', {json, filename})` |
| Doc file  | `invoke('read_file', {path})`              |

### Che do Web (khong co Tauri)

Neu chay tren trinh duyet (khong co Tauri wrapper):

- Luu anh: chuyen sang base64, luu vao IndexedDB
- Backup: download JSON qua `<a download>`
- VSCode: hien thi thong bao "Can chay Tauri desktop app"

### Phu thuoc ngoai (CDN hoac self-hosted)

| Thu vien     | Phien ban | Cach dung        |
| ------------ | --------- | ---------------- |
| TailwindCSS  | v3.4+     | CDN hoac build   |
| Dexie.js     | v3.2+     | CDN / npm        |
| TinyMCE      | 6.8+      | Self-hosted      |
| vis-network  | 9.1       | CDN              |
| Chart.js     | 4.4       | CDN              |
| EasyMDE      | 2.18+     | CDN              |
| SortableJS   | 1.15+     | CDN              |
| compromise   | 14.x      | CDN (intent/NLP) |
| PapaParse    | 5.x       | CDN (CSV import) |
| Highlight.js | 11.x      | CDN              |

File nay duoc cap nhat moi khi them module moi. Phien ban hien tai: v1.0 – Module 1 (Layout)

---

**`cau-truc-thu-muc.md`** – Tài liệu kiến trúc toàn bộ dự án, mô tả cây thư mục, quy tắc đặt tên, luồng dữ liệu, Tauri API mapping. Thuộc: **Tài liệu nền tảng dự án**.
