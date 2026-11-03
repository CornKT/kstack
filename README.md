# kstack

Agent skills and global instructions for Claude Code, Codex, and OpenCode (with [T3 Code](https://t3.codes) support).

Migrated and adapted from [danielphan2003/dstack](https://github.com/danielphan2003/dstack).

---

## Danh sách Skills (`skills/`)

Dưới đây là 16 skills đã được chuyển giao vào repo:

| Skill | Nguồn gốc | Tác dụng |
| :--- | :--- | :--- |
| **`babysit-pr`** | tstack (Theo Browne) | Tự động theo dõi PR qua các vòng review (bot AI) và CI; tự sửa lỗi và push cho đến khi sẵn sàng merge. |
| **`council`** | tstack (Theo Browne) | Triệu tập hội đồng đa mô hình (frontier models) để audit và phản biện code chéo trước khi merge. |
| **`file-pr`** | tstack (Theo Browne) | Mở PR ngắn gọn, chuẩn chỉnh bằng GitHub CLI (`gh`), tự so sánh diff với nhánh base. |
| **`full-send`** | tstack (Theo Browne) | Chế độ tự động hóa hoàn toàn từ đầu đến cuối (tự code, test, mở PR, audit và tự động merge). |
| **`full-stop`** | tstack (Theo Browne) | Tương tự `full-send` nhưng dừng lại ở bước tạo PR xong và chờ người dùng tự bấm merge. |
| **`leaving-pr-comments`** | tstack (Theo Browne) | Tiêu chuẩn để agent viết nhận xét review trên GitHub PR (gắn kèm callout note minh bạch). |
| **`modernize`** | tstack (Theo Browne) | Rebase PR cũ lên commit mới nhất của nhánh chính, giải quyết conflict và chạy lại test. |
| **`sol-loop`** | tstack (Theo Browne) | Vòng lặp cải thiện độ chính xác code bằng cách nhờ model GPT Sol review và fix lỗi lặp lại. |
| **`takeover`** | tstack (Theo Browne) | Tiếp quản PR hoặc nhánh dở dang từ người khác / agent khác để hoàn thiện mục tiêu. |
| **`pstack-blast-radius`** | pstack (Lauren Tan) | Phân tích và đo lường rủi ro, dự đoán các module có thể bị ảnh hưởng/hỏng ngầm bởi thay đổi code. |
| **`pstack-interrogate`** | pstack (Lauren Tan) | Review phản biện đối nghịch (adversarial review) bằng nhiều model để tìm điểm mù và stress test code. |
| **`pstack-technical-writing`**| pstack (Lauren Tan) | Hướng dẫn viết tài liệu kỹ thuật (RFC, docs, README) súc tích, dễ hiểu. |
| **`pstack-unslop`** | pstack (Lauren Tan) | Lược bỏ các từ ngữ sáo rỗng, cấu trúc văn phong máy móc của AI để bài viết tự nhiên. |
| **`effect`** | Local (Daniel Phan) | Cẩm nang hướng dẫn chuyên sâu phát triển ứng dụng TypeScript với Effect v4. |
| **`lakebed-share`** | Local (Daniel Phan) | Chia sẻ trang HTML/mockup ra ngoài thread an toàn với mã hóa đầu-cuối (E2E) qua Lakebed. |
| **`wattpad-archiver`** | Local (Daniel Phan) | Công cụ sao lưu thư viện truyện từ Wattpad về máy dưới định dạng EPUB. |

---

## Hướng dẫn sử dụng

Tạo symbolic link (symlink) từ thư mục `skills/` vào thư mục cấu hình của agent:

```bash
# Cho Claude Code
ln -s "$PWD/skills/council" ~/.claude/skills/council

# Cho Codex hoặc OpenCode
ln -s "$PWD/skills/council" ~/.agents/skills/council
```
