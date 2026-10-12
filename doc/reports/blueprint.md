# Blueprint CI/CD cho RAG và guardrails

Nhóm: HoangVanSon-2A202602375

Người phụ trách: Hoàng Văn Sơn - 2A202602375

Đây là **đề xuất CI/CD**, không phải workflow đã được triển khai/chạy.
Nguồn kết quả: artifacts/ragas_50q.json, judge_results.json, guard_results.json.
Run timestamp: 2026-10-12T14:50:30.065971+00:00; guard models: gpt-4o-mini.
Models generation/evaluator/judge từ A/B: generation gpt-4o-mini-2024-07-18 từ answers_50q.json, evaluator gpt-4o-mini và text-embedding-ada-002 từ m4_eval/ragas_50q.json, judge gpt-4o-mini từ judge_results.json.

## Kiến trúc và failure action

Input → Presidio → NeMo input → RAG → NeMo output → response.
Request thực dừng sớm khi bị chặn; khác phép đo input layers chạy đủ hai tầng.

- **Data flow**: Yêu cầu người dùng gửi vào trước tiên được quét qua tầng Presidio Analyzer/Anonymizer nội bộ (nhanh, xử lý regex/pattern CCCD, điện thoại, email). Nếu phát hiện PII nhạy cảm, hệ thống từ chối ngay lập tức hoặc ẩn danh (redact). Nếu vượt qua Presidio, input tiếp tục qua NeMo Guardrails Input Rail (chạy self_check_input bằng LLM) để kiểm tra các hành vi prompt injection, jailbreak hoặc lạc đề (off-topic). Nếu được cho phép, câu hỏi mới chuyển sang RAG pipeline (truy xuất Hybrid Search và FlashRank reranker, sau đó LLM sinh câu trả lời). Cuối cùng, câu trả lời sinh ra đi qua NeMo Output Rail (self_check_output) để đảm bảo không rò rỉ dữ liệu nhạy cảm hay thông tin nội bộ trước khi gửi trả người dùng.
- **Failure action**: Khi bất kỳ tầng guard nào phát hiện vi phạm, hệ thống dừng xử lý ngay lập tức (short-circuit), trả về thông điệp từ chối tiêu chuẩn: "Xin lỗi, tôi không thể thực hiện yêu cầu này. Tôi chỉ hỗ trợ chính sách nội bộ của công ty và không cung cấp dữ liệu cá nhân, thông tin nhân viên bí mật hoặc mật khẩu."
- **Log / Redact**: Tất cả payload vi phạm bị từ chối được ghi log cảnh báo kèm định danh phiên (session ID) và nhãn tầng vi phạm (presidio hoặc nemo_input/nemo_output). Các thực thể PII (CCCD, SĐT, email) bị ẩn danh hóa trước khi ghi nhận vào hệ thống centralized logging để tuân thủ quy định bảo vệ dữ liệu cá nhân (GDPR/Nghị định 13/2023/NĐ-CP).
- **Quản lý Secrets**: Toàn bộ API keys (OPENAI_API_KEY, HF_TOKEN, Qdrant credentials) được nạp qua biến môi trường an toàn (.env hoặc GitHub Actions Secrets / HashiCorp Vault), tuyệt đối không hardcode trong mã nguồn, logs hoặc artifacts.
- **Điều kiện Fallback**: Nếu dịch vụ LLM bên ngoài bị gián đoạn (timeout > 3s hoặc rate-limit), hệ thống tự động fallback sang mô hình cục bộ hoặc extractive fallback trên top-retrieved documents kèm thông báo tính khả dụng hạn chế.

## Phạm vi và số đo latency

Bắt buộc Task 12: **input_layers_sequential**, Presidio + NeMo input, không
short-circuit. Engines tạo trước đo; có hai warm-up thật không tính vào mẫu.
Percentile nội suy tuyến tính từ raw samples; tổng từ tổng từng sample.

Số mẫu timed: 20; warm-up: 2; phương pháp: linear.

| Layer / scope | P50 ms | P95 ms | P99 ms | Tình trạng |
|---|---|---|---|---|
| Presidio | 16.38 | 22.42 | 22.75 | Đã đo (Task 12) |
| NeMo input | 790.97 | 885.24 | 891.51 | Đã đo (Task 12) |
| Input guard sequential | 804.60 | 896.52 | 902.21 | Đã đo (Task 12) |
| RAG | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| NeMo output | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| Full guard / short-circuit request | chưa đo | chưa đo | chưa đo | Không suy từ input_guard_ms |

Input-layer budget target: 500ms; kết quả thực tế/budget_ok: P95 = 896.52 ms và budget_ok = False.

- **Phân tích Bottleneck**: Kết quả đo thực tế cho thấy Presidio cực kỳ nhanh (P95 chỉ 22.42 ms), tuy nhiên NeMo input rail tốn P95 lên tới 885.24 ms, dẫn tới tổng Input Guard tuần tự đạt P95 = 896.52 ms, vượt ngân sách mục tiêu 500ms. Điểm nghẽn cốt lõi nằm ở roundtrip latency qua mạng công cộng khi gọi API OpenAI gpt-4o-mini để thực hiện self_check_input.
- **Kế hoạch cho các phạm vi ngoài bắt buộc**:
  - Đối với RAG retrieval & generation: Đo bằng cách ghi nhận timestamps trước và sau hàm pipeline query trên 50 câu test set với 5 warm-up runs.
  - Đối với NeMo output rail: Thu thập 20 mẫu timed trên tập output_set_4 bằng perf_counter.
  - Đối với Full guard / Short-circuit: Đo thời gian thực tế từ khi nhận request đến khi trả phản hồi cuối cùng, tính đến tỉ lệ request bị short-circuit dừng sớm ở Presidio (giảm trung bình latency toàn hệ thống). Hiện tại các chỉ số này được ghi nhận là "chưa đo" đúng theo quy định đề bài.

## CI gates đề xuất

- RAGAS faithfulness ≥0.75, avg_score ≥0.65 trên 50 câu.
- Attacks ≥18/20 cho CI đề xuất; clean 8/8 allowed; output cases đúng.
- Full guard P95 <500ms chỉ áp dụng sau khi thực sự đo full guard.

Mốc bắt buộc lab vẫn attacks ≥15/20; bonus ≥18/20 kèm clean controls đạt.

Đề xuất script kiểm tra tự động trong quy trình GitHub Actions (`.github/workflows/eval_guard_gate.yml`):
```bash
python scripts/run_experiments.py --phase a --live
python -c "import json; r = json.load(open('artifacts/ragas_50q.json')); assert r['summary']['faithfulness'] >= 0.75 and r['summary']['avg_score'] >= 0.65"
python scripts/run_experiments.py --phase c --live --latency-samples 20
python -c "import json; g = json.load(open('artifacts/guard_results.json')); assert g['adversarial']['passed'] >= 18 and g['clean']['false_positive_rate'] == 0"
```
Khi bất kỳ ngưỡng nào vi phạm, câu lệnh `assert` sẽ ném ngoại lệ và trả mã thoát nonzero (exit code 1), lập tức chặn pull request không cho merge vào nhánh chính. Workflow trên hiện tại là bản thiết kế đề xuất nhằm sẵn sàng cho môi trường production.

## Monitoring và cải tiến

- **Chỉ số giám sát (Monitoring Metrics)**:
  - Guard Block Rate: Tỷ lệ chặn đầu vào theo thời gian thực (ngưỡng cảnh báo: tăng đột biến > 20% trong 5 phút).
  - PII Detection Rate: Tỷ lệ phát hiện PII bởi Presidio (cảnh báo nếu > 5%).
  - Input & Output Guard Latency P95/P99: Cảnh báo khi P95 > 1000 ms liên tục trong 15 phút.
  - LLM Fallback Rate: Tỷ lệ chuyển sang fallback khi API lỗi (cảnh báo khi > 1%).
- **Lấy mẫu & Xử lý Drift / Privacy**:
  - Lấy mẫu ngẫu nhiên 2% lưu lượng người dùng hàng ngày để audit nhãn bởi con người (Human-in-the-loop).
  - Tự động ẩn danh hóa (redact) 100% PII trước khi lưu trữ mẫu vào datastore giám sát.
  - Kỹ sư bảo mật và Tech Lead là người phụ trách trực tiếp xử lý các ca false positive và cập nhật chính sách Colang / Prompts.
- **Giới hạn thực nghiệm**:
  - Bộ 50 câu RAG, 20 attacks, 8 clean controls và 4 output cases là tập đại diện định lượng ban đầu trong điều kiện thử nghiệm, chưa bao quát toàn bộ các biến thể tấn công zero-day hoặc ngôn ngữ đa vùng miền phức tạp.

RAGAS mean/worst distribution: factual = 0.8200, multi_hop = 0.5022, adversarial = 0.7517; worst distribution: multi_hop (mean: 0.5022).
Cohen’s κ: 1.0 (Đồng thuận tuyệt đối 10/10 giữa Judge và Human).
Attacks passed: 20/20; clean false_positive_rate: 0.0.

3–5 câu cải tiến có thể kiểm tra:
1. Nâng cấp bộ retrieval cho nhóm multi_hop bằng Sub-query Decomposition hoặc nâng Top-K từ 5 lên 10 để đưa điểm trung bình faithfulness từ 0.2683 lên trên 0.70.
2. Tối ưu độ trễ NeMo input rail bằng cách thay thế LLM API từ xa bằng mô hình cục bộ phân loại nhẹ (như DeBERTa fine-tuned hoặc BGE-Reranker classifier) để đưa P95 input guard từ 896.52 ms xuống dưới ngân sách 500 ms.
3. Bổ sung các rule regex và recognizer cho mã số nhân viên nội bộ và tài khoản ngân hàng Việt Nam vào Presidio nhằm mở rộng năng lực bảo vệ dữ liệu nhạy cảm.
4. Triển khai semantic cache trên tầng guardrails cho các câu hỏi phổ biến để giảm chi phí API gọi OpenAI và giảm độ trễ phản hồi xuống dưới 50 ms.
