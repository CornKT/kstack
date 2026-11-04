# kstack

A curated suite of agent skills, workflows, and global instructions designed for **Claude Code** and **Codex**, optimized for use with **[T3 Code](https://t3.codes)** and modern multi-agent workflows.

---

## 🚀 Danh sách Skills (`skills/`)

`kstack` cung cấp các bộ kỹ năng chuyên biệt giúp tự động hóa và nâng cao năng lực cho coding agent:

### 1. Quy trình Git & Tự động hóa PR (Workflows & PR Automation)

| Skill | Mô tả & Tác dụng |
| :--- | :--- |
| **`file-pr`** | Mở Pull Request ngắn gọn, chuẩn chỉ qua GitHub CLI (`gh`), tự động so sánh diff với nhánh base và sinh tiêu đề theo Conventional Commits. |
| **`babysit-pr`** | Tự động theo dõi PR qua các vòng review (đặc biệt là bot review AI) và kiểm tra CI; tự động sửa lỗi và push code cập nhật cho đến khi sẵn sàng merge. |
| **`full-send`** | Chế độ tự động hóa hoàn toàn end-to-end: agent tự code, chạy test, mở PR, chạy kiểm tra chất lượng và tự động merge khi hoàn tất. |
| **`full-stop`** | Thực hiện toàn bộ quy trình như `full-send` nhưng dừng lại ở bước PR sẵn sàng để người dùng tự tay review và bấm merge. |
| **`modernize`** | Rebase và đồng bộ các PR cũ lên phiên bản mới nhất của nhánh chính, tự động giải quyết xung đột (conflict) và chạy lại test. |
| **`takeover`** | Tiếp quản một PR hoặc nhánh dở dang từ người khác / agent khác để hoàn thiện mục tiêu mà vẫn giữ nguyên credit của người đóng góp ban đầu. |
| **`leaving-pr-comments`** | Chuẩn hóa định dạng nhận xét do AI viết trên GitHub PR, luôn gắn callout `> [!NOTE]` minh bạch. |

### 2. Đánh giá chất lượng & Phản biện đa mô hình (Review & Auditing)

| Skill | Mô tả & Tác dụng |
| :--- | :--- |
| **`council`** | Triệu tập "Hội đồng AI" gồm nhiều frontier models để audit, rà soát chéo code và tìm kiếm các lỗ hổng kiến trúc trước khi merge. |
| **`pstack-interrogate`** | Đánh giá đối nghịch (adversarial review) bằng sự đa dạng của nhiều model khác nhau để tìm ra các điểm mù và stress test code. |
| **`pstack-blast-radius`** | Đánh giá "bán kính ảnh hưởng" của một thay đổi code xem có nguy cơ gây lỗi ngầm cho các module khác trong hệ thống hay không. |
| **`sol-loop`** | Vòng lặp cải thiện độ chính xác của code thông qua việc đối chiếu và sửa lỗi lặp lại trước khi tạo PR. |

### 3. Viết tài liệu & Chuẩn hóa nội dung (Technical Writing)

| Skill | Mô tả & Tác dụng |
| :--- | :--- |
| **`pstack-technical-writing`** | Hướng dẫn viết tài liệu kỹ thuật (RFC, README, docs, runbook) theo cấu trúc 4 tầng, dễ hiểu ngay từ lần đọc đầu tiên. |
| **`pstack-unslop`** | Quét và lược bỏ các từ ngữ sáo rỗng, cấu trúc văn phong máy móc của AI để bài viết tự nhiên và chuẩn văn phong con người. |

### 4. Kỹ thuật chuyên sâu & Tiện ích (Specialized Engineering & Tools)

| Skill | Mô tả & Tác dụng |
| :--- | :--- |
| **`effect`** | Hướng dẫn toàn diện và chuẩn mực khi xây dựng ứng dụng TypeScript với kiến trúc Effect v4 (Services, Layers, Schemas, Streams, Caches, Tests). |

---

## 🛠️ Cài đặt & Sử dụng

Để agent có thể nhận diện và sử dụng các skill này, tạo symbolic link (symlink) vào thư mục cấu hình tương ứng của từng công cụ:

```bash
# Dành cho Claude Code
ln -s "$PWD/skills/council" ~/.claude/skills/council

# Dành cho Codex
ln -s "$PWD/skills/council" ~/.agents/skills/council
```

Sau khi liên kết, bạn có thể gọi trực tiếp tên skill (ví dụ: `council`, `full-send`) hoặc phím tắt `$` trong T3 Code và các công cụ agent.

---

## 📜 Cấu hình toàn cục (`AGENTS.md`)

File [`AGENTS.md`](AGENTS.md) chứa các nguyên tắc toàn cục về:
- Phong cách code (TypeScript type safety, Svelte 5 runes, Effect v4).
- Quy chuẩn commit theo Conventional Commits.
- Chiến lược điều phối Subagents qua `delegate_task` của T3 Code.
