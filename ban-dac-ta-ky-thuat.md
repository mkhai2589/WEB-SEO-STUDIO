1. TỔNG QUAN
   • Tên tool: Local SEO Studio 2026
   • Loại ứng dụng: Desktop app chạy 100% local (offline-first) – sử dụng Tauri v2 để đóng gói và tương tác hệ thống file.
   • Đối tượng sử dụng: SEO specialist, content writer, WordPress developer tại Việt Nam.
   • Ngôn ngữ giao diện: Tiếng Anh / Tiếng Việt (có nút toggle, lưu preference bằng Dexie).
   • Ngôn ngữ soạn thảo nội dung: Hỗ trợ cả tiếng Anh và tiếng Việt (TinyMCE spell-check, readability riêng).
   Mục tiêu SEO 2026 hỗ trợ:
   • Topic Cluster & Topical Authority
   • Entity SEO
   • E E A T (Experience – Expertise – Authoritativeness – Trustworthiness)
   • AI Overview (AIO) readiness
   • Semantic search & conversational intent
   Chức năng chính:
1. Quản lý Topic / Pillar / Cluster / Entity toàn diện.
1. Viết content với chấm điểm SEO real-time (theo chuẩn 2026).
1. Đồ thị Internal Link thông minh + đề xuất tự động.
1. Ghi chú đa dạng (note, case study, quy trình) + tích hợp VSCode.
1. Xuất code WordPress sạch + Schema JSON-LD đầy đủ.

---

2. KIẾN TRÚC KỸ THUẬT
   Thành phần Công nghệ Ghi chú
   Frontend HTML5 + TailwindCSS v3 + JavaScript ES6 (Vanilla) Không framework nặng, bundle nhẹ
   Desktop wrapper Tauri v2 (Rust backend) Thay thế hoàn toàn Node.js, xử lý file system, spawn VSCode, lưu ảnh
   Database Dexie.js (wrapper cho IndexedDB) Hỗ trợ indexing, versioning, transaction batch
   Editor chính TinyMCE 6.8+ (self-hosted) Plugin: image, link, code, table, lists, advlist
   Markdown editor (note) EasyMDE (CodeMirror) Kết hợp Highlight.js
   Đồ thị vis-network 9.1 Force-directed layout, zoom, pan, filter
   Biểu đồ Chart.js 4.4 Line, pie, bar
   Xử lý nặng Web Worker TF-IDF, cosine similarity, SEO scoring, duplicate detection
   Build tool Vite (khuyến nghị) hoặc HTML thuần -
   Lưu ý quan trọng:
   • Không dùng Node.js + Express. Mọi tương tác file system (đọc/ghi ảnh, backup, spawn VSCode) đều qua Tauri API.
   • Ảnh upload được lưu vào thư mục $APP_DATA/images/ (Tauri).
   • Backup tự động lưu vào $APP_DATA/backups/.

---

3. CẤU TRÚC THƯ MỤC
   text
   local-seo-studio/
   ├── public/ # TinyMCE self-hosted, static assets
   ├── src/
   │ ├── db/
   │ │ ├── dexie.js # Khởi tạo Dexie
   │ │ └── schema.js # Định nghĩa các store và index
   │ ├── modules/
   │ │ ├── content/ # Editor, SEO score panel, AIO preview
   │ │ ├── topic/ # Quản lý topic, cluster, entity, keyword Kanban
   │ │ ├── graph/ # vis-network + internal link logic
   │ │ ├── notes/ # EasyMDE, VSCode integration
   │ │ └── dashboard/ # Widgets, charts
   │ ├── utils/
   │ │ ├── seoScore.js # Hàm tính điểm (chạy trong worker)
   │ │ ├── tfidfWorker.js # Worker tính similarity
   │ │ ├── readabilityVN.js
   │ │ ├── entityDetect.js
   │ │ └── duplicateChecker.js
   │ ├── ui/
   │ │ ├── components/ # Modal, toast, sidebar, header
   │ │ ├── hotkeys.js
   │ │ ├── zenMode.js
   │ │ └── i18n.js
   │ └── main.js # Khởi tạo app, routing
   ├── tauri/ # Cấu hình Tauri v2 (src-tauri)
   │ ├── Cargo.toml
   │ └── tauri.conf.json
   ├── backups/ # Thư mục lưu backup (do Tauri tạo)
   ├── output/ # Export WordPress, schema
   └── index.html

---

4. DATABASE SCHEMA (Dexie.js)
   javascript
   // db/schema.js
   import Dexie from 'dexie';

export const db = new Dexie('LocalSEOStudio2026');
db.version(1).stores({
notes: '++id, type, title, tags, category, updatedAt',
contents: '++id, topicId, clusterId, status, seoScore, updatedAt, slug',
topics: '++id, name, pillarPageId',
clusters: '++id, topicId, name',
keywords: '++id, clusterId, keyword, type',
keyword_usage: '++id, keywordId, contentId, timestamp',
internal_links: '++id, sourceContentId, targetContentId, anchorText, createdAt',
similarities: '++id, sourceId, targetId, score, updatedAt',
entities: '++id, clusterId, entityName, type, frequency, lastUpdated',
eeat_signals: '++id, contentId, authorName, authorBio, experienceSignals, citations',
backups: '++id, timestamp, jsonData',
});

// Mỗi object có thêm createdAt, updatedAt (tự động set)
Chi tiết các trường (mẫu):
contents:
javascript
{
id: Number,
h1: String,
slug: String,
metaDesc: String,
focusKeyword: String,
topicId: Number,
clusterId: Number,
bodyHtml: String,
bodyText: String, // plain text cho phân tích
authorName: String,
authorBio: String,
experienceSignals: Array, // ["đã làm dự án X", ...]
citations: Array, // [{url, title}]
status: 'draft'|'publish',
seoScore: Number,
featuredImagePath: String, // đường dẫn Tauri
internalLinksCount: Number,
duplicateRisk: Number, // 0-1
createdAt: Date,
updatedAt: Date
}
entities:
javascript
{
id: Number,
clusterId: Number,
entityName: String,
type: 'Person'|'Concept'|'Brand'|'Product'|'Location'|'Other',
frequency: Number,
lastUpdated: Date
}
eeat_signals:
javascript
{
id: Number,
contentId: Number,
authorName: String,
authorBio: String,
experienceSignals: [String],
citations: [{url: String, title: String}]
}
internal_links:
javascript
{
id: Number,
sourceContentId: Number,
targetContentId: Number,
anchorText: String,
createdAt: Date
}
similarities: lưu kết quả cosine similarity giữa các bài (để tránh tính lại).
backups: mỗi bản backup là toàn bộ DB export JSON.

---

5. CHI TIẾT TỪNG MODULE
   5.1. Module Ghi chú (Notes)
   Giao diện:
   • Sidebar trái (collapsible): danh sách notes dạng bảng – cột: Icon loại, Tiêu đề, Tags, Ngày cập nhật.
   • Ô tìm kiếm + filter theo loại (note/case study/process).
   • Nút “+ Tạo note mới” → modal chọn loại.
   Khi chọn note:
   • Khu vực chính hiển thị editor phù hợp:
   o Note thường: EasyMDE (Markdown), tags input (autocomplete từ tags cũ).
   o Case study: form với 4 textarea (Vấn đề, Giải pháp, Kết quả, Bài học) + drag drop ảnh.
   o Quy trình: danh sách bước dùng SortableJS (kéo thả), mỗi bước có checkbox + mô tả ngắn + nút xóa.
   Hành động trên mỗi dòng:
   • Sửa, Xóa, Xuất file Markdown, Mở trong VSCode (gọi Tauri spawn).
   Công nghệ: EasyMDE, Highlight.js, SortableJS, Tauri FS.

---

5.2. Module Viết Content – SEO Real-time
Bố cục 3 cột (có thể thu gọn):
Cột trái (collapsible, 20% chiều rộng)
• Thanh tìm kiếm Topic / Cluster (autocomplete – gõ chữ hiện gợi ý realtime).
• Quick E E A T fields:
o Author Name (input)
o Author Bio (textarea ngắn)
o Experience Signals (thêm dạng chip, mỗi chip là một chuỗi)
• Danh sách 5 bài viết cùng cluster (click mở bài đó).
Cột giữa (Editor – 55-60% chiều rộng)
• TinyMCE full chiều cao, các thanh công cụ cơ bản.
• Floating toolbar (hiện khi bôi đen text):
o “Gợi ý Internal Link” → popup danh sách bài tương đồng (dựa trên similarity) → chèn link với anchor đã chọn.
o “Thêm Semantic / Entity” → hiện danh sách từ khóa/entity từ cluster hiện tại → click để chèn.
o “Chèn Atomic Answer” → chèn template tóm tắt ngắn (<60 từ) dưới H2.
• Nút Zen Mode (góc trên phải editor) → ẩn cả 2 cột bên, chỉ còn editor + floating toolbar. Nhấn Esc hoặc nút để thoát.
• Form header phía trên editor:
o H1 (input) – tự sinh slug
o Slug (input + nút “Auto generate”)
o Meta description (textarea, counter 160 ký tự + nút “Gợi ý từ nội dung”)
o Focus Keyword (input)
o Status: Draft / Publish (radio)
o Featured Image: drag drop zone + preview (lưu qua Tauri FS)
Cột phải (Right panel – 20-25%, có thể thu gọn)
• Tổng điểm SEO (số to 0-100 + thanh progress màu) + nút “Refresh score”.
• Accordion các nhóm (mặc định chỉ mở nhóm có vấn đề nghiêm trọng nhất):
o Độ dài & Cấu trúc Heading
o Semantic & Entity Coverage
o E E A T Signals
o Internal Links
o Readability (VN/ENG)
o AIO Readiness
• Checklist chi tiết (✅/❌) và 3 gợi ý thiếu hụt hàng đầu (ví dụ: “Thiếu H2 chứa focus keyword”, “Thêm ít nhất 2 citation”, “Atomic answer còn thiếu”).
• Tab AIO Readiness:
o Preview giao diện giống Google AI Overview (tóm tắt + citation).
o Checklist: atomic answers dưới mỗi H2, schema ready, conversational tone.
Real time behavior:
• Mọi thay đổi trong TinyMCE → debounce 800ms → gửi plain text + metadata sang Web Worker → tính lại điểm SEO → cập nhật right panel.
• Tự động highlight entity & semantic keyword trong editor (màu nền nhẹ + tooltip).
• Khi publish: cập nhật keyword_usage, tính lại similarities với tất cả bài khác, cập nhật duplicateRisk.
Công nghệ: TinyMCE, Web Worker, Tauri FS, custom highlight.

---

5.3. Module Quản lý Topic / Pillar / Cluster / Entity
Bố cục:
• Sidebar trái (accordion đa cấp, luôn visible):
o Level 1: Danh sách Topic (click mở).
o Level 2: Pillar Page (hiển thị link + nút “Viết bài Pillar” hoặc “Mở bài”).
o Level 3: Các Cluster (có thể drag drop để sắp xếp lại thứ tự). Mỗi cluster hiển thị icon số bài viết.
• Khu vực chính (tabbed):
Tab “Từ khóa” – Kanban 3 cột (Chưa dùng | Đang dùng | Đã publish).
o Mỗi card keyword có tên, loại (chính/đồng nghĩa), nút xem bài viết sử dụng.
o Drag drop keyword giữa các cột để thay đổi trạng thái (cập nhật keyword_usage thủ công).
Tab “Entity List” – Bảng các entity của cluster hiện tại.
o Cột: Entity Name, Type, Frequency, nút Edit/Xóa.
o Nút “+ Thêm Entity” (modal: tên, type).
Tab “Bài viết” – Danh sách bài viết thuộc topic/cluster (bảng: Tiêu đề, Điểm SEO, Trạng thái, nút Sửa/Xóa/Preview).
• Nút hành động chung (góc trên phải):
o “+ Tạo Topic mới” (modal: Tên, Mô tả, Pillar page – dropdown chọn content, Màu sắc).
o “Import CSV/Excel” (đọc file có cột: cluster_name, keyword, synonyms, type).
o “Export toàn bộ” (JSON hoặc CSV).
Cơ chế đếm “đã dùng / chưa dùng”:
• Tự động: khi publish bài, quét nội dung (bodyText, focusKeyword, metaDesc) tìm keyword trong DB → ghi vào keyword_usage.
• Thủ công: trong Kanban, có nút “Đánh dấu đã dùng” trên mỗi card.
Công nghệ: HTML5 drag drop, PapaParse, SheetJS, Dexie.

---

5.4. Module Internal Link Graph & Đề xuất
Giao diện toàn màn hình:
• vis-network (force directed) chiếm hầu hết không gian.
• Toolbar trên cùng:
o Nút “Filter: Chỉ bài mồ côi”, “Chỉ Pillar”, “Zoom to fit”.
o Thanh tìm kiếm node (gõ tiêu đề → highlight node).
• Màu node:
o Xanh đậm: nhiều inbound link (≥3).
o Đỏ: mồ côi (0 inbound).
o Vàng: ít inbound (1-2).
• Click node → mở Right drawer (slide in):
o Thông tin bài viết (tiêu đề, điểm SEO, số link đi/đến).
o Danh sách “Top 5 đề xuất link đến bài này” (dựa trên similarity).
o Mỗi đề xuất có nút “Chèn link” → tự động thêm link vào bài hiện tại (cập nhật DB và editor nếu đang mở).
• Bảng “Bài mồ côi” (dưới graph) – sortable theo tiêu đề, điểm SEO, có nút “Mở bài” và “Gợi ý link mạnh nhất”.
• Right click trên node → context menu: Preview bài | Edit bài | Đề xuất link | Xóa node (chỉ xóa khỏi đồ thị tạm thời).
Cơ chế tính similarity & đề xuất:
• Mỗi khi lưu bài (publish), chạy Web Worker tính cosine similarity giữa bài hiện tại với tất cả bài khác dựa trên TF IDF của toàn bộ nội dung (không giới hạn 500 từ).
• Lưu kết quả vào bảng similarities.
• Đề xuất khi similarity > 0.45 và chưa có link giữa 2 bài.
Công nghệ: vis-network, Web Worker (TF IDF), Dexie.

---

5.5. Module Dashboard & Báo cáo
Bố cục: grid widget có thể kéo thả (drag drop) để sắp xếp lại vị trí.
• Widget 1 – Tổng quan:
o Số content, số note, số topic, số cluster.
o % AIO Opportunity = (số bài đạt đủ tiêu chí AIO / tổng số bài) \* 100.
• Widget 2 – Biểu đồ điểm SEO trung bình theo tuần/tháng (Chart.js line).
• Widget 3 – Biểu đồ tròn phân bố bài theo cluster (Chart.js pie).
• Widget 4 – Top 5 bài có điểm SEO cao nhất (danh sách click mở bài).
• Widget 5 – Top 5 bài mồ côi cần xử lý (có nút “Mở bài” và “Gợi ý link”).
• Nút cố định: “Xuất báo cáo CSV toàn bộ”, “Tạo backup ngay”, “Restore backup”.
Công nghệ: Chart.js, HTML5 drag drop (cho widget), Dexie.

---

6. CÁCH TÍNH ĐIỂM SEO REAL-TIME 2026 (TỔNG 100 ĐIỂM)
   Hàm chạy trong Web Worker, nhận đầu vào là object chứa tất cả dữ liệu cần thiết.
   javascript
   // utils/seoScore.js (chạy trong worker)
   export async function calculateSEO2026(data) {
   const {
   wordCount, focusKeyword, clusterKeywords, entities, headings,
   images, internalLinks, authorBio, citations, experienceSignals,
   bodyText, duplicateRisk
   } = data;

let score = 0;

// 1. Độ dài & Cấu trúc (25đ)
if (wordCount >= 2000) score += 15;
else if (wordCount >= 1500) score += 10;
else if (wordCount >= 1000) score += 5;
else score += 0;
if (headings.h2 >= 2 && headings.h3 >= 1) score += 10;
else if (headings.h2 >= 1 && headings.h3 >= 1) score += 5;

// 2. Keyword & Semantic/Entity (30đ)
const density = (bodyText.match(new RegExp(focusKeyword, 'gi')) || []).length / wordCount _ 100;
if (density >= 0.5 && density <= 2.5) score += 5;
else if (density > 0) score += 3;
const semanticScore = calculateSemanticCoverage(bodyText, clusterKeywords, entities); // 0-1
score += Math.round(semanticScore _ 25);

// 3. E-E-A-T (15đ)
if (authorBio && authorBio.length > 80 && experienceSignals.length > 0) score += 10;
else if (authorBio && authorBio.length > 40) score += 5;
if (citations.length >= 3) score += 5;
else if (citations.length >= 1) score += 2;

// 4. Internal link & Readability VN (15đ)
if (internalLinks >= 1) score += 8;
const readabilityOk = calculateReadabilityVN(bodyText);
score += readabilityOk ? 7 : 3;

// 5. AIO & Schema Readiness (10đ)
const hasAtomic = hasAtomicAnswers(headings, bodyText);
const schemaReady = data.schemaReady; // true nếu đã export schema hoặc có cấu trúc phù hợp
if (hasAtomic && schemaReady) score += 10;
else if (hasAtomic) score += 5;

// 6. Duplicate penalty
if (duplicateRisk > 0.75) score = Math.max(0, score - 15);
else if (duplicateRisk > 0.5) score = Math.max(0, score - 5);

return Math.min(100, Math.max(0, Math.round(score)));
}
Hàm hỗ trợ (cần implement):
• calculateReadabilityVN(text):
o Tách câu (dùng regex cho dấu chấm, chấm than, chấm hỏi).
o Tính số từ trung bình mỗi câu (lý tưởng 18 25).
o Tính % câu dài >30 từ (lý tưởng <25%).
o Trả về true nếu cả hai chỉ số đều trong vùng lý tưởng.
• calculateSemanticCoverage(bodyText, clusterKeywords, entities):
o Kết hợp TF IDF cosine giữa bodyText và clusterKeywords (coi clusterKeywords như một tài liệu).
o Tính % entity overlap: số entity trong bài / số entity của cluster.
o Điểm = 0.7*cosine + 0.3*entityOverlap.
• hasAtomicAnswers(headings, bodyText):
o Với mỗi H2, kiểm tra đoạn văn ngay sau (tối đa 100 ký tự) có tóm tắt ngắn (<60 từ) trả lời trực tiếp chủ đề H2 không.
o Trả về true nếu ít nhất 70% H2 có atomic answer.
Màu thanh điểm:
• Đỏ: 0 49
• Cam: 50 69
• Xanh lá: 70 89
• Xanh dương: 90 100

---

7. CƠ CHẾ ĐỀ XUẤT CLUSTER TỰ ĐỘNG
   Khi người dùng nhấn nút “Gợi ý cluster” (hoặc real time sau 5 giây không gõ):
1. Lấy tiêu đề H1 + bodyText (plain).
1. Duyệt tất cả cluster trong DB.
1. Mỗi cluster, tính điểm phù hợp = cosine similarity (TF IDF) giữa bài viết hiện tại và văn bản ghép từ tên cluster + danh sách keyword của cluster.
1. Nhân thêm hệ số entity bonus (nếu entity của cluster xuất hiện trong bài).
1. Chọn 3 cluster có điểm cao nhất, ngưỡng tối thiểu 0.45.
1. Nếu không có cluster nào đạt ngưỡng, hiển thị “Chưa có cluster phù hợp – bạn có thể tạo cluster mới”.
   Hiển thị:
   • Trong module viết bài, cạnh dropdown cluster có nút “Gợi ý cluster”.
   • Popup hiển thị danh sách: “Cluster A (topic X) – độ phù hợp 87%”, mỗi dòng có nút “Chọn”.
   Real time tự động: Sau 5 giây không gõ, nếu điểm của cluster hiện tại thấp hơn cluster gợi ý top1 ít nhất 15%, hiển thị thông báo nhẹ “Bạn có muốn đổi sang cluster phù hợp hơn?”.

---

8. CÁC TÍNH NĂNG MỚI 2026 (BẮT BUỘC)
   8.1. Duplicate Detection
   • Dùng shingling (k shingle = 9) + cosine similarity giữa bài hiện tại và các bài đã publish.
   • Nếu similarity > 0.75 → cảnh báo đỏ trong right panel, đồng thời highlight đoạn trùng trong editor (màu nền đỏ nhạt).
   • Lưu giá trị duplicateRisk vào bảng contents.
   8.2. Intent Classifier
   • Sử dụng compromise.js (NLP nhẹ) + heuristics.
   • Phân loại thành: Informational (hỏi “là gì”, “cách”), Commercial (“so sánh”, “tốt nhất”), Transactional (“mua”, “giá”).
   • Hiển thị một dropdown gợi ý ở sidebar trái (có thể chỉnh tay).
   8.3. FAQ / People Also Ask (PAA) Generator
   • Nút “Generate FAQ” trong editor.
   • Parse các H2, H3 và nội dung xung quanh, dùng câu hỏi mẫu (“là gì”, “như thế nào”, “tại sao”) để tạo danh sách câu hỏi.
   • Hiển thị popup cho phép chỉnh sửa, sau đó nhấn “Chèn FAQ + Schema” → tự động thêm vào cuối bài và tạo schema FAQPage.
   8.4. Schema Markup Generator
   • Nút “Export Schema JSON-LD” trong editor.
   • Cho phép chọn loại: Article, FAQPage, HowTo, Author, Speakable.
   • Tự động lấy dữ liệu từ bài viết (tiêu đề, meta desc, author, câu hỏi FAQ nếu có).
   • Xuất ra đoạn script JSON-LD, người dùng copy dán vào WordPress.
   8.5. Entity Highlight & Coverage
   • Dùng entityDetect.js (dựa trên từ điển entity từ DB + compromise).
   • Trong khi soạn thảo, tự động highlight các entity đã biết (màu nền vàng nhạt).
   • Tính entity coverage = (số entity xuất hiện / tổng số entity của cluster) \* 100%, hiển thị trong right panel.
   8.6. Competitor Gap Analysis
   • Cho phép import file text hoặc CSV chứa nội dung đối thủ (hoặc URL – nhưng local nên chỉ nhập text).
   • Tool tính TF IDF cho nội dung đối thủ, so sánh với bài hiện tại → tìm ra các chủ đề / entity mà đối thủ có nhưng mình thiếu.
   • Hiển thị dưới dạng danh sách gợi ý bổ sung.
   8.7. Auto Backup & Versioning
   • Cứ mỗi 5 phút hoặc sau 10 thay đổi (ghi, sửa, xóa), tự động export toàn bộ DB thành JSON + timestamp.
   • Lưu vào thư mục backups/ (qua Tauri FS).
   • Giữ lại tối đa 20 bản backup gần nhất (xoá cũ tự động).
   • Dashboard có nút “Restore từ backup” (chọn file JSON và ghi đè DB).

---

9. TÍCH HỢP VSCODE
   • Mọi đối tượng note và content đều có nút “Mở trong VSCode” (icon).
   • Khi click, gọi Tauri command:
   rust
   // Tauri backend (Rust) #[tauri::command]
   fn open_in_vscode(path: String) -> Result<(), String> {
   std::process::Command::new("code")
   .arg("--new-window")
   .arg(path)
   .spawn()
   .map_err(|e| e.to_string())?;
   Ok(())
   }
   • Frontend: gọi invoke('open_in_vscode', { path: fullPath }).
   • File được lưu tạm vào $APP_DATA/temp_notes/ hoặc $APP_DATA/temp_content/ với phần mở rộng .md (cho note) hoặc .html (cho content).
   Yêu cầu: Người dùng phải cài VSCode và có command line code trong PATH (Tauri sẽ kiểm tra và báo lỗi nếu chưa).

---

10. HOTKEYS TOÀN CỤC & UI/UX
    • Ctrl + S – Lưu draft hiện tại (content hoặc note).
    • Ctrl + K – Mở modal tìm kiếm toàn cục (tìm trong tiêu đề, nội dung, tag).
    • Ctrl + E – Xuất bài hiện tại ra WordPress code (hiển thị popup code + copy).
    • Ctrl + Shift + F – Bật/tắt Zen Mode (ẩn sidebar + right panel).
    • F11 – Fullscreen trình duyệt (nếu chạy web) hoặc toàn màn hình Tauri.
    UI tối thiểu:
    • Dark mode mặc định (có toggle ở header).
    • Sidebar trái có thể thu gọn (icon hamburger).
    • Right panel có thể thu gọn thành thanh dọc nhỏ.
    • Tất cả modal đều có thể đóng bằng phím ESC.
    Responsive: Chỉ hỗ trợ desktop (tối thiểu 1366x768).

---

11. YÊU CẦU MÔI TRƯỜNG & CÀI ĐẶT
    • Hệ điều hành: Windows 10/11, macOS 11+, Linux (bản phân phối có hỗ trợ Tauri).
    • Trình duyệt (khi chạy web mode): Chrome, Edge, Firefox mới nhất.
    • VSCode: Phải có command line (code --version hoạt động).
    • Rust & Tauri CLI (nếu build từ mã nguồn).
    • Quyền truy cập: Đọc/ghi file trong thư mục ứng dụng (Tauri tự xin quyền).
    Cách chạy (development):
    bash
    git clone ...
    npm install
    npm run tauri dev
    Build production:
    bash
    npm run tauri build
    Sẽ tạo ra file cài đặt (.msi, .dmg, .AppImage) trong src-tauri/target/release/.
