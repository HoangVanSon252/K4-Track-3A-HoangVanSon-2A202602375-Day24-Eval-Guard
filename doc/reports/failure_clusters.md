# Phân tích lỗi Phase A

Nhóm: HoangVanSon-2A202602375

Người phụ trách: Hoàng Văn Sơn - 2A202602375

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-12T09:17:12.514850+00:00.
Generation mode/fallback và điều kiện chạy: Chạy trực tiếp qua pipeline RAG kết hợp Hybrid Search (BM25 + Dense BAAI/bge-m3) và FlashRank reranker; câu trả lời được sinh trực tiếp bằng LLM OpenAI gpt-4o-mini-2024-07-18 với 0 câu fallback.
Models generation/evaluator: Generation model gồm gpt-4o-mini-2024-07-18; Evaluator models từ m4_eval gồm gpt-4o-mini và text-embedding-ada-002.

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0.8333 | 0.2683 | 0.8000 |
| answer_relevancy | 0.7092 | 0.4197 | 0.5903 |
| context_precision | 0.9125 | 0.8125 | 0.9667 |
| context_recall | 0.8250 | 0.5083 | 0.6500 |
| avg_score | 0.8200 | 0.5022 | 0.7517 |

## Bottom 10 và nguyên nhân

| Rank | ID | Phân phối | Điểm TB | Metric yếu nhất | Chẩn đoán | Đề xuất khắc phục |
|---|---|---|---|---|---|---|
| 1 | 8 | factual | 0.0000 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 2 | 35 | multi_hop | 0.0000 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 3 | 40 | multi_hop | 0.0000 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 4 | 6 | factual | 0.1458 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 5 | 23 | multi_hop | 0.2083 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 6 | 32 | multi_hop | 0.2500 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 7 | 34 | multi_hop | 0.2500 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 8 | 39 | multi_hop | 0.2500 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 9 | 50 | adversarial | 0.3333 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 10 | 21 | multi_hop | 0.3750 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |

### Đối chiếu chi tiết các case đại diện
- **Case ID 39 (multi_hop)**: Câu hỏi yêu cầu so sánh yêu cầu mật khẩu giữa policy v1.0 và v2.0 (độ dài, chu kỳ đổi, MFA). Context chỉ lấy được đoạn trích v1 và v2 về độ dài mà thiếu phần chu kỳ đổi và MFA, dẫn đến câu trả lời bị thiếu dữ kiện ("Không tìm thấy"). Điều này kéo tụt faithfulness và answer_relevancy về 0.0.
- **Case ID 40 (multi_hop)**: Câu hỏi hỏi về nhân viên Manager thử việc phát hiện sự cố bảo mật (nên và không nên làm gì). Đoạn retrieved context chỉ trả về quy định thử việc 60-90 ngày và chính sách nghỉ không lương, hoàn toàn không truy xuất được quy trình báo cáo bảo mật trong 1 giờ qua helpdesk@cty.vn. Đây là lỗi truy xuất multi-hop khi hai thực thể (thử việc và bảo mật) nằm ở hai văn bản tách biệt.
- **Case ID 50 (adversarial)**: Hỏi về việc nhân viên Manager có thể dùng VPN cá nhân như NordVPN khi WFH không. Do câu hỏi có tính gợi ý khẳng định, mô hình hoặc retrieval bị nhiễu giữa tính năng bảo mật chung và quy định nghiêm cấm của công ty đối với phần mềm VPN bên thứ ba.
- **Case ID 8 (factual)**: Câu hỏi về số ngày nghỉ khi cha mẹ mất. Ngữ cảnh có nhắc đến nghỉ phép tang lễ nhưng do định dạng văn bản khiến mô hình suy diễn lệch hoặc không trích xuất đúng ý, dẫn tới điểm faithfulness bị đánh giá 0.0.

## Matrix và nhóm yếu nhất

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 4 | 17 | 2 |
| context_recall | 2 | 2 | 6 |
| context_precision | 2 | 0 | 0 |
| answer_relevancy | 12 | 1 | 2 |

Nhóm yếu theo mean thấp nhất: multi_hop; mean: 0.5022.
Worst metric xuất hiện nhiều nhất: faithfulness; số câu: 23.

### Nguyên nhân và giải pháp cải thiện
Nhóm `multi_hop` đạt điểm trung bình thấp nhất (0.5022) do đặc thù câu hỏi đòi hỏi kết hợp thông tin từ nhiều điều khoản khác nhau (ví dụ: cấp bậc lương kết hợp với phụ cấp, hoặc thâm niên kết hợp với quy chế nghỉ ốm). Khi một trong các chunk tài liệu bị thiếu trong Top-K của retrieval, mô hình LLM không có đủ căn cứ và trả lời "Không tìm thấy" hoặc tự suy luận dẫn đến vi phạm faithfulness (17/20 câu multi_hop có worst metric là faithfulness).
Để khắc phục:
1. Nâng cao chất lượng retrieval bằng cách tăng kích thước `top_k` của Dense/BM25 hoặc áp dụng Query Expansion / Sub-query Decomposition trước khi tìm kiếm.
2. Thiết lập chặt chẽ System Prompt với temperature = 0 để hạn chế hallucination khi context không đầy đủ.
3. Chạy lại benchmark với bộ 50 câu hỏi để đo lường sự cải thiện của faithfulness trên nhóm multi_hop.

## Bẫy adversarial và giới hạn

Nhóm `adversarial` đạt điểm trung bình 0.7517, thấp hơn nhóm `factual` (0.8200). Điểm yếu tập trung chủ yếu ở `context_recall` (6/10 câu worst metric là context_recall).
Nguyên nhân cốt lõi là bẫy xung đột phiên bản (Version Conflict) và bẫy phủ định (Negation Trap):
- Bẫy phiên bản (như chính sách nghỉ phép v2023 12 ngày so với v2024 15 ngày, chính sách mật khẩu v1.0 8 ký tự so với v2.0 12 ký tự): Pipeline thường truy xuất cả 2 tài liệu cũ và mới vào context. Nếu prompt không có chỉ dẫn ưu tiên phiên bản hiệu lực mới nhất, câu trả lời sẽ dễ chọn nhầm văn bản cũ.
- Bẫy phủ định (như quyền lợi thử việc không được hưởng phép năm và gói PVI): Tài liệu không đề cập trực tiếp từ phủ định mạnh nên mô hình có thể suy diễn sai.
Giới hạn thực nghiệm: Bộ kiểm thử 10 câu adversarial chỉ phản ánh một phần các tình huống bẫy thực tế. Khi triển khai production, cần bổ sung metadata filter theo ngày hiệu lực và trường cấm rõ ràng để triệt tiêu hoàn toàn lỗi phiên bản.
