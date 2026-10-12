# Nhóm thực hiện K4 Track 3A

Tên nhóm: HoangVanSon-2A202602375

Repo chung: https://github.com/HoangVanSon252/K4-Track-3A-HoangVanSon-2A202602375-Day24-Eval-Guard

Trưởng nhóm/người nộp: Hoàng Văn Sơn - 2A202602375

Ngày lab/deadline áp dụng: 2026-10-12 23:59 — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: b9f8d5b53bc4f94899aa30fdc52c30be0e96eb6d

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; nhóm thực hiện đơn thành viên (1 người).

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Hoàng Văn Sơn | 2A202602375 | Trưởng nhóm đơn, phụ trách Setup, Phase A, B, C, Runner và hoàn thiện báo cáo | Git commits, artifacts JSON, 3 reports Markdown |

Người tích hợp/chạy batch/nộp link: Hoàng Văn Sơn, thực thi toàn bộ pipeline setup_answers, run_experiments và check_lab.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: Windows 11, Python 3.11.9, MSVC v143.
Virtualenv/dependencies/imports: Đã cài đặt virtualenv .venv, requirements.txt và constraints-core.txt; tải spaCy model en_core_web_lg thành công.
Models/Qdrant/50 answers đã chuẩn bị: Qdrant v1.9.7 Docker container (cổng 6333); mô hình embedding BAAI/bge-m3; 50 answers tạo bởi GPT-4o-mini và lưu tại artifacts/answers_50q.json.
NeMo Guardrails 0.10.0 và Presidio Analyzer/Anonymizer hoạt động đầy đủ trên môi trường runtime.

## INDIVIDUAL

### HoangVanSon-2A202602375

- Phần được giao và phần thực tế đã làm: Do làm nhóm đơn (1 người), phụ trách toàn bộ đề bài: thiết lập môi trường, chạy setup pipeline sinh 50 answers, triển khai Tasks 1-4 (Phase A RAGAS), Tasks 5-8 (Phase B LLM Judge), Tasks 9a-12 (Phase C Guardrails) và Task 13 (Blueprint).
- Bằng chứng: Các commit trên repo GitHub cá nhân, các file artifacts/answers_50q.json, artifacts/ragas_50q.json, artifacts/judge_results.json, artifacts/guard_results.json và 3 báo cáo trong doc/reports/.
- Cách kiểm tra: Đã kiểm tra qua bộ pytest offline (86 passed), chạy live batch runners trên toàn bộ dữ liệu 50 câu RAG, 5 cặp/10 nhãn judge và 28 guard inputs; vượt qua bộ script kiểm tra check_lab.py.
- Điều đã học: Nắm vững quy trình đánh giá hệ thống RAG đa chiều (Faithfulness, Relevancy, Precision, Recall), cách đo lường độ tin cậy và thiên lệch (Position/Verbosity bias, Cohen's Kappa) của LLM-as-a-Judge, kiến trúc Guardrails đa lớp (Presidio PII + NeMo Guardrails) và cách đo độ trễ chuẩn percentile P95 trong môi trường production.
- Giới hạn/cách xử lý: Qdrant-client bản 1.9.2 không hỗ trợ query_points nên đã điều chỉnh adapter để tương thích ngược qua client.search; thêm caching cho quá trình enrichment 109 chunks để tối ưu chi phí và thời gian chạy lặp lại.
- Nguồn/AI hỗ trợ nếu có: Tham khảo tài liệu đề bài, RAGAS docs, NeMo Guardrails docs và có sự hỗ trợ của Antigravity AI Assistant trong việc rà soát cú pháp và tối ưu luồng chạy.

## Cập nhật sau khi nộp

Bản nộp đầu tiên đúng hạn trước 23:59 ngày làm lab, không có cập nhật phát sinh sau deadline.
