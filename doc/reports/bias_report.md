# Đánh giá LLM Judge Phase B

Nhóm: K4-Track-3A-HoangVanSon-2A202602375

Người phụ trách: Hoàng Văn Sơn - 2A202602375

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-12T09:18:08.705561+00:00.
Models generation/judge: generation `gpt-4o-mini-2024-07-18` và judge `gpt-4o-mini`.

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers_50q.json:answer; B từ
fixtures/human_labels_10q.json:model_answer; reference từ fixtures/test_set_50q.json:ground_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Câu hỏi | Candidate A | Candidate B | Pass 1 | Pass 2 | Final | Nhất quán vị trí | Điểm Pass 1 (A/B) | Điểm Pass 2 (A/B) | Nhận xét tiêu biểu |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ bao nhiêu ngày khi kết hôn? | Nhân viên được nghỉ 3 ngày làm việc khi kết hôn. | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | B | B | B | True | 0.8 / 1.0 | 0.8 / 1.0 | Pass 1 & 2 đều chọn B vì nêu rõ nghỉ có lương và không trừ vào phép năm. |
| 5 | Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt? | Cần phê duyệt của Tổng Giám đốc (CEO). | Cần Giám đốc phòng ban phê duyệt. | A | A | A | True | 1.0 / 0.0 | 1.0 / 0.0 | A đúng theo mốc trên 50 triệu VNĐ cần CEO; B nêu sai người phê duyệt. |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là bao nhiêu? | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là **1 tháng lương**. | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | A | A | A | True | 1.0 / 0.8 | 1.0 / 0.8 | A nêu đầy đủ điều kiện từ 6 tháng trở lên và trình bày định dạng rõ ràng hơn. |
| 21 | Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào? | Không tìm thấy. | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | B | B | B | True | 0.0 / 1.0 | 0.0 / 1.0 | B trả lời đầy đủ multi-hop (phép và lương); A bỏ cuộc ("Không tìm thấy"). |
| 41 | Nhân viên được nghỉ bao nhiêu ngày phép năm? | Nhân viên được nghỉ 15 ngày phép năm có lương. | Nhân viên được nghỉ 12 ngày phép năm. | A | A | A | True | 1.0 / 0.0 | 1.0 / 0.3 | A chính xác theo chính sách v2024 (15 ngày có lương); B dùng tài liệu lỗi thời v2023. |

Position bias count/rate: 0 cặp không nhất quán (tỷ lệ 0.0, 0.0%).
Verbosity win rate: 1.0 (100.0%); decisive cases: 5/5 cặp có kết quả quyết định.

### Phân tích tính nhất quán vị trí và xu hướng độ dài

- **Position Consistency**: Cả 5/5 cặp (100%) đều đạt tính nhất quán hoàn hảo qua hai lượt tráo đổi vị trí (`pass1` A vs B và `pass2` B vs A). Judge không bị ảnh hưởng bởi việc câu trả lời xuất hiện ở vị trí Candidate 1 hay Candidate 2 trong prompt.
- **Verbosity Bias**: Tỷ lệ câu trả lời dài hơn chiến thắng là 1.0 (5/5 case quyết định, gồm 3 case A dài hơn thắng và 2 case B dài hơn thắng).
- **Phân biệt tương quan thống kê và bản chất ngữ nghĩa**: Mặc dù trên phương diện thống kê có vẻ như judge thiên vị câu dài (Verbosity win rate = 1.0), việc phân tích sâu nội dung cho thấy câu thắng dài hơn là do cung cấp đúng và đủ thông tin bắt buộc:
  - Ở câu 1: B dài hơn vì bổ sung thuộc tính pháp lý quan trọng "có lương".
  - Ở câu 5: A dài hơn một chút nhưng mấu chốt là chỉ định đúng cấp thẩm quyền CEO thay vì Giám đốc phòng ban.
  - Ở câu 12: A dài hơn do nhắc lại mệnh đề điều kiện "từ 6 tháng trở lên" theo reference.
  - Ở câu 21: B dài hơn vượt trội vì giải quyết trọn vẹn câu hỏi multi-hop, trong khi A chỉ có 3 từ ("Không tìm thấy.").
  - Ở câu 41: A nêu 15 ngày và kèm "có lương" theo đúng tài liệu v2024 mới nhất.

Do đó, độ dài ở đây là hệ quả của độ đầy đủ và chính xác về mặt thông tin chứ không phải do judge bị đánh lừa bởi văn phong dài dòng rỗng tuếch.

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong fixtures/human_labels_10q.json;
không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference.
Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| ID | Câu hỏi | Model Answer Gốc | Nhãn Judge (0/1) | Human Label (0/1) | Đồng thuận (Agree) | Lý do phán quyết (Reasoning) |
|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ bao nhiêu ngày khi kết hôn? | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | 1 | 1 | Đồng thuận | Câu trả lời chính xác theo thông tin trong reference, nêu rõ số ngày nghỉ và điều kiện không trừ vào phép năm. |
| 5 | Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt? | Cần Giám đốc phòng ban phê duyệt. | 0 | 0 | Đồng thuận | Câu trả lời sai vì cần Tổng Giám đốc phê duyệt chứ không phải Giám đốc phòng ban. |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là bao nhiêu? | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | 1 | 1 | Đồng thuận | Câu trả lời chính xác theo thông tin trong reference, nêu rõ điều kiện và mức thưởng cho nhân viên chính thức có từ 6 tháng trở lên. |
| 21 | Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào? | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | 1 | 1 | Đồng thuận | Câu trả lời chính xác theo reference, nêu rõ số ngày phép và khoảng lương cho nhân viên Senior với 9 năm thâm niên. |
| 23 | Nhân viên được tài trợ khóa học 25 triệu, nghỉ việc sau 8 tháng. Phải hoàn trả bao nhiêu? | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | 1 | 1 | Đồng thuận | Câu trả lời chính xác theo reference, nêu rõ nhân viên phải hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. |
| 29 | Nhân viên tạm ứng 8 triệu, chưa thanh toán sau 30 ngày. Ai phê duyệt và phí phạt là bao nhiêu? | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 triệu. | 0 | 0 | Đồng thuận | Câu trả lời sai vì không đề cập đến sự cần thiết có sự phê duyệt của Kế toán trưởng bên cạnh Trưởng phòng. Ngoài ra, phí phạt được tính cho 15 ngày quá hạn, không phải là 2% trên 8 triệu mà là 80.000 VNĐ. |
| 33 | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm? | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng. | 1 | 1 | Đồng thuận | Câu trả lời cung cấp đúng số ngày phép và tổng phụ cấp hàng tháng theo thông tin trong tài liệu tham khảo. |
| 41 | Nhân viên được nghỉ bao nhiêu ngày phép năm? | Nhân viên được nghỉ 12 ngày phép năm. | 0 | 0 | Đồng thuận | Câu trả lời sai vì nó dựa trên chính sách cũ (v2023) thay vì chính sách hiện hành (v2024) là 15 ngày. |
| 46 | Nhân viên thử việc có được nghỉ phép năm không? | Nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần. | 1 | 1 | Đồng thuận | Câu trả lời chính xác theo reference, nêu rõ rằng nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần, đồng thời không bỏ qua điều kiện phê duyệt từ trưởng phòng. |
| 50 | Nhân viên Manager có thể dùng VPN cá nhân (NordVPN) khi WFH không? | Được, miễn là đảm bảo kết nối an toàn. | 0 | 0 | Đồng thuận | Câu trả lời sai vì khẳng định rằng nhân viên Manager có thể dùng VPN cá nhân khi WFH, trong khi chính sách rõ ràng cấm việc này. |

Cohen’s κ: 1.0 (Đồng thuận tuyệt đối).
Observed/chance agreement và diễn giải:
- Tỷ lệ đồng thuận quan sát được: $P_o = \frac{10}{10} = 1.0$ (100%).
- Số nhãn 1: 6 câu; Số nhãn 0: 4 câu ở cả Judge và Human.
- Tỷ lệ đồng thuận ngẫu nhiên kỳ vọng: $P_e = \frac{6 \times 6 + 4 \times 4}{10 \times 10} = \frac{36 + 16}{100} = 0.52$.
- Hệ số Cohen's Kappa: $\kappa = \frac{P_o - P_e}{1 - P_e} = \frac{1.0 - 0.52}{1.0 - 0.52} = 1.0$.
- Diễn giải: Giá trị $\kappa = 1.0$ thể hiện sự tương thích hoàn hảo giữa LLM Judge (`gpt-4o-mini`) và nhãn của chuyên gia con người trên tập mẫu thử nghiệm hiệu chỉnh. Mô hình nắm bắt chính xác cả các trường hợp đúng hoàn toàn lẫn các lỗi sai tinh vi (ví dụ: nhầm thẩm quyền phê duyệt, tính sai tiền phạt quá hạn, hoặc dùng tài liệu phiên bản cũ).

## Giới hạn và cách dùng judge

- **Quy mô tập mẫu thử nghiệm**: Tập calibration chỉ gồm 10 mẫu nhị phân và 5 cặp so sánh pairwise. Mặc dù hệ số $\kappa = 1.0$ là tín hiệu rất tích cực, kết quả này chưa phản ánh toàn diện độ tin cậy của judge trên các câu hỏi phức tạp đa miền ngoài phân phối hẹp của bài lab.
- **Giới hạn của LLM Judge**:
  - LLM Judge dùng prompt zero-shot hoặc few-shot có thể gặp hiện tượng hallucination khi reference quá dài hoặc phức tạp.
  - Nguy cơ bias tiềm ẩn khi gặp các câu trả lời sinh bởi mô hình cùng họ (self-enhancement bias) hoặc các câu trả lời cố tình viết dài, dùng từ hoa mỹ để ngụy tạo độ chính xác.
- **Chiến lược ứng dụng trong thực tế**:
  - Không sử dụng LLM Judge như nguồn sự thật tuyệt đối (ground truth) thay thế con người hoàn toàn.
  - Sử dụng LLM Judge trong CI/CD pipeline như một công cụ đánh giá tự động hồi quy (automated regression testing gate) để phát hiện nhanh các lỗi suy giảm chất lượng lớn.
  - Áp dụng kỹ thuật swap-and-average và ensemble nhiều model judge (ví dụ gpt-4o, claude-3-5-sonnet) cho các quyết định nhạy cảm.
  - Thiết lập cơ chế cảnh báo Human-in-the-loop: Bất kỳ câu nào có độ bất đồng giữa 2 pass (position inconsistent), điểm số ở ngưỡng ranh giới (borderline 0.5), hoặc câu hỏi thuộc nhóm rủi ro cao (chính sách lương thưởng, bảo mật) đều phải được chuyển sang cho chuyên gia con người kiểm định.
