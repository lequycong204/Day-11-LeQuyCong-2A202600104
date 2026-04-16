# Tóm tắt công việc Day 11

## Chủ đề ngày 11

Day 11 tập trung vào `Guardrails`, `HITL (Human-in-the-Loop)` và `Responsible AI`.
Mục tiêu chính không phải chỉ làm cho agent chạy được, mà là biến một agent "dễ bị tấn công" thành một hệ thống có lớp phòng vệ, có cách kiểm thử an toàn, và có cơ chế chuyển sang con người khi rủi ro cao.

## Mục tiêu cuối cùng cần đạt

Bạn cần hoàn thành 2 đầu ra chính:

1. `Security Report`
So sánh trước và sau khi thêm guardrails với ít nhất `5 cuộc tấn công`.
Report phải thể hiện rõ agent ban đầu bị khai thác như nào, guardrails nào đã chặn được gì, và điểm nào vẫn còn rủi ro.

2. `HITL Flowchart`
Thiết kế luồng ra quyết định có ít nhất `3 decision points`, nêu rõ khi nào hệ thống tự xử lý, khi nào cần chặn, khi nào cần escalate sang con người.

## Toàn bộ công việc phải hoàn thành

Ngày 11 có `13 TODO`, chia thành 4 phần lớn:

1. Tấn công agent chưa bảo vệ
2. Xây guardrails đầu vào và đầu ra
3. Kiểm thử lại và tạo pipeline đánh giá an toàn
4. Thiết kế HITL workflow

---

## Phần 1: Tấn công agent chưa được bảo vệ

### TODO 1: Viết 5 adversarial prompts

#### Cần làm gì

Tạo ít nhất `5 prompt tấn công` để thử phá agent chưa có guardrails.
Các prompt này nên đa dạng, không nên chỉ cùng một kiểu prompt injection.

#### Nên có những nhóm tấn công nào

- `Prompt injection`: yêu cầu model bỏ qua chỉ dẫn trước đó
- `Role override`: giả làm system/admin/developer
- `Data exfiltration`: cố ép agent lộ secrets, API key, cấu hình nội bộ
- `Policy bypass`: yêu cầu trả lời nội dung đáng lẽ phải bị chặn
- `Indirect attack / hidden instruction`: chèn chỉ dẫn đánh lừa trong ngữ cảnh

#### Cách hoàn thành tốt

Mỗi prompt nên có:

- Tên case
- Mục tiêu tấn công
- Nội dung prompt
- Kỳ vọng rủi ro nếu agent không được bảo vệ

#### Tiêu chí hoàn thành

- Có tối thiểu 5 prompt
- Prompt đủ khác nhau về kiểu tấn công
- Có thể dùng lại để test trước/sau guardrails

### TODO 2: Dùng AI để sinh thêm attack test cases

#### Cần làm gì

Sử dụng `Gemini` để hỗ trợ tạo thêm test cases hoặc biến thể của các prompt tấn công.
Mục đích là tăng độ phủ kiểm thử, không phụ thuộc vào 5 prompt viết tay ban đầu.

#### Cách làm đúng

- Yêu cầu AI sinh thêm biến thể theo từng nhóm tấn công
- Giữ lại những case thực sự có giá trị kiểm thử
- Loại bỏ case trùng ý hoặc quá yếu

#### Đầu ra nên có

- Danh sách attack cases được mở rộng
- Có phân nhóm theo loại tấn công
- Có thể dùng trực tiếp trong phần testing pipeline

#### Tiêu chí hoàn thành

- Có thêm các test case ngoài 5 prompt gốc
- Các test case mới phục vụ được kiểm thử bảo mật
- Có cơ sở để đưa vào đánh giá tự động

---

## Phần 2A: Input Guardrails

### TODO 3: Injection detection bằng regex

#### Cần làm gì

Xây một bộ lọc đầu vào để phát hiện các dấu hiệu `prompt injection` hoặc chỉ dẫn phá luật.

#### Nên kiểm tra những gì

- Cụm như `ignore previous instructions`
- Cụm như `reveal system prompt`
- Cụm như `act as`
- Cụm như `developer mode`, `bypass`, `override`
- Mẫu lệnh cố ép agent tiết lộ thông tin nội bộ

#### Cách hoàn thành tốt

- Dùng `regex` hoặc danh sách pattern rõ ràng
- Trả về kết quả dễ hiểu: `safe / unsafe`, lý do, pattern nào match
- Không chỉ trả `True/False`, vì phần report cần giải thích tại sao bị chặn

#### Tiêu chí hoàn thành

- Phát hiện được các prompt injection phổ biến
- Có logic rõ ràng, dễ kiểm thử
- Có thể dùng như một bước trước khi gửi prompt vào agent

### TODO 4: Topic filter

#### Cần làm gì

Xây bộ lọc chủ đề để chặn hoặc giới hạn các chủ đề không được phép.

#### Ví dụ chủ đề cần xử lý

- Nội dung nguy hiểm
- Dữ liệu nhạy cảm
- Chủ đề ngoài phạm vi ứng dụng
- Chủ đề bị policy cấm

#### Cách hoàn thành tốt

- Có danh sách `allowed topics` và `blocked topics`
- Nếu input thuộc chủ đề bị chặn thì trả cảnh báo hoặc từ chối sớm
- Nếu input ngoài phạm vi thì định nghĩa hành vi rõ ràng: từ chối, chuyển hướng, hoặc escalate

#### Tiêu chí hoàn thành

- Phân biệt được nội dung hợp lệ và nội dung ngoài chính sách
- Có rule nhất quán
- Dễ tích hợp với input guardrail plugin

### TODO 5: Input Guardrail Plugin cho Google ADK

#### Cần làm gì

Đóng gói các kiểm tra ở TODO 3 và TODO 4 thành `plugin` hoặc hook trong `Google ADK`.

#### Mục tiêu thực tế

Trước khi agent xử lý câu hỏi, hệ thống phải kiểm tra:

1. Có dấu hiệu injection không
2. Có thuộc topic bị chặn không
3. Nếu có vấn đề thì chặn hoặc trả phản hồi an toàn ngay

#### Cách hoàn thành tốt

- Tích hợp guardrails thành luồng chạy thực tế, không chỉ là hàm rời rạc
- Có đầu ra thống nhất: cho phép, chặn, hoặc gắn cờ
- Có log hoặc thông tin lý do để phục vụ report

#### Tiêu chí hoàn thành

- Agent thật sự gọi input guardrails trước khi xử lý
- Prompt nguy hiểm bị chặn trước khi vào model
- Có thể chứng minh hiệu quả bằng test trước/sau

---

## Phần 2B: Output Guardrails

### TODO 6: Content filter cho output

#### Cần làm gì

Kiểm tra phản hồi đầu ra để phát hiện nội dung không an toàn, ví dụ:

- `PII`
- `secrets`
- Nội dung vi phạm chính sách
- Thông tin mà agent không nên tiết lộ

#### Cách hoàn thành tốt

- Quét output bằng rule-based checks
- Có danh mục những thứ cần phát hiện
- Nếu phát hiện vi phạm thì thay phản hồi bằng câu từ chối hoặc sanitize output

#### Tiêu chí hoàn thành

- Chặn được output chứa dữ liệu nhạy cảm hoặc nội dung không phù hợp
- Có lý do cụ thể tại sao output bị đánh dấu
- Có thể tái sử dụng trong pipeline test

### TODO 7: LLM-as-Judge safety check

#### Cần làm gì

Dùng `Gemini` như một bộ chấm an toàn cho output.
Thay vì chỉ dựa vào regex, cho model đánh giá xem câu trả lời có an toàn, có vi phạm policy, có nên trả ra cho user hay không.

#### Cách hoàn thành tốt

- Viết prompt đánh giá rõ tiêu chí
- Yêu cầu model trả về kết quả có cấu trúc
- Ví dụ: `safe/unsafe`, `risk_score`, `reason`

#### Lưu ý quan trọng

- Không nên dùng judge mơ hồ
- Prompt judge phải nhất quán, dễ parse
- Nên có fallback nếu judge trả lời lỗi hoặc không đúng format

#### Tiêu chí hoàn thành

- Có thể đánh giá output bằng LLM
- Kết quả judge dùng được trong quyết định chặn/cho qua
- Có lý do giải thích cho mỗi case

### TODO 8: Output Guardrail Plugin cho Google ADK

#### Cần làm gì

Tích hợp `content filter` và `LLM-as-Judge` vào luồng trả kết quả của agent.

#### Luồng mong muốn

1. Agent sinh câu trả lời
2. Output đi qua rule-based filter
3. Output đi qua LLM judge nếu cần
4. Nếu an toàn thì trả user
5. Nếu không an toàn thì chặn, sửa, hoặc trả lời an toàn hơn

#### Tiêu chí hoàn thành

- Có lớp kiểm tra sau khi model sinh output
- Có cơ chế chặn output nguy hiểm
- Có thể chứng minh hiệu quả bằng test case cụ thể

---

## Phần 2C: NeMo Guardrails

### TODO 9: Cấu hình NeMo Guardrails với Colang

#### Cần làm gì

Dùng `NeMo Guardrails` của NVIDIA để định nghĩa rule an toàn theo kiểu khai báo (`Colang`).

#### Mục tiêu

Không chỉ có guardrails viết tay bằng Python, mà còn phải thử một framework chuyên dụng cho safety.

#### Bạn cần thể hiện được

- Cách định nghĩa rule/hành vi trong Colang
- Rule nào dùng để chặn hoặc điều hướng hội thoại
- Sự khác nhau giữa guardrails custom và NeMo Guardrails

#### Tiêu chí hoàn thành

- Có cấu hình hoặc demo dùng NeMo Guardrails
- Có ít nhất một rule an toàn chạy được
- Có thể dùng để so sánh với cách làm bằng ADK/Python

---

## Phần 3: Testing và defense pipeline

### TODO 10: Chạy lại 5 attacks sau khi có guardrails

#### Cần làm gì

Dùng chính 5 prompt tấn công ban đầu để test lại hệ thống đã thêm guardrails.

#### Mục tiêu

Chứng minh guardrails có tác dụng thực tế, không chỉ tồn tại trên code.

#### Kết quả nên ghi lại

- Attack case
- Kết quả trước khi phòng vệ
- Kết quả sau khi phòng vệ
- Guardrail nào đã xử lý
- Còn lọt rủi ro hay không

#### Tiêu chí hoàn thành

- Có bảng trước/sau rõ ràng
- Có ít nhất 5 case được rerun
- Từ bảng này có thể viết `Security Report`

### TODO 11: Automated security testing pipeline

#### Cần làm gì

Tạo pipeline tự động để chạy nhiều attack cases qua agent và ghi nhận kết quả.

#### Pipeline nên làm được gì

- Nhận danh sách test cases
- Chạy lần lượt qua agent
- Ghi trạng thái `blocked`, `allowed`, `flagged`, `unsafe`
- Lưu lại lý do hoặc output mẫu
- Tổng hợp tỉ lệ chặn thành công

#### Cách hoàn thành tốt

- Có cấu trúc dữ liệu rõ ràng cho test cases
- Có format kết quả nhất quán
- Có thể chạy lại nhiều lần

#### Tiêu chí hoàn thành

- Pipeline chạy được mà không phải test tay từng case
- Có đầu ra đủ để so sánh và báo cáo
- Dùng được cho cả ADK guardrails và NeMo guardrails nếu có thể

---

## Phần 4: HITL (Human-in-the-Loop)

### TODO 12: Confidence Router

#### Cần làm gì

Thiết kế một `router` quyết định khi nào agent được tự trả lời, khi nào cần chuyển sang người kiểm duyệt.

#### Các tín hiệu có thể dùng

- Điểm tự tin thấp
- Nội dung nhạy cảm
- Phát hiện mâu thuẫn
- Kết quả judge không chắc chắn
- User yêu cầu có rủi ro cao

#### Hành vi mong muốn

- `high confidence + low risk`: cho agent tự trả lời
- `medium confidence hoặc có dấu hiệu nghi ngờ`: gắn cờ hoặc cần review
- `low confidence hoặc high risk`: escalate sang human

#### Tiêu chí hoàn thành

- Có rule routing rõ ràng
- Có thể giải thích vì sao một case bị escalate
- Kết nối logic này với phần safety phía trên

### TODO 13: Thiết kế 3 điểm quyết định HITL

#### Cần làm gì

Vẽ hoặc mô tả một `HITL flowchart` với ít nhất `3 decision points`.

#### 3 điểm quyết định tối thiểu nên có

1. `Input screening`
Nếu input có injection, topic cấm, hoặc dấu hiệu abuse thì chặn hoặc escalate.

2. `Output safety review`
Nếu output bị content filter hoặc judge đánh giá nguy hiểm thì không trả trực tiếp cho user.

3. `Low-confidence / high-impact decision`
Nếu agent không chắc hoặc câu trả lời có tác động lớn thì chuyển sang human reviewer.

#### Flowchart nên thể hiện

- Điểm bắt đầu từ user request
- Các nhánh `allow`, `block`, `review`, `escalate`
- Ai là người xử lý cuối cùng ở mỗi nhánh

#### Tiêu chí hoàn thành

- Có đúng 3 decision points trở lên
- Có escalation path rõ ràng
- Có thể dùng như thiết kế vận hành cho hệ thống thật

---

## Những gì cần có trong bài nộp để được xem là hoàn chỉnh

### 1. Mã nguồn hoặc notebook hoàn thành các TODO chính

Ít nhất phải có:

- Attack prompts và attack cases
- Input guardrails
- Output guardrails
- Demo hoặc cấu hình NeMo Guardrails
- Testing pipeline
- Logic HITL / confidence router

### 2. Security Report

Report nên có các phần sau:

- Mô tả agent ban đầu
- Danh sách 5+ cuộc tấn công
- Kết quả trước khi có guardrails
- Kết quả sau khi có guardrails
- Guardrail nào chặn được gì
- Những điểm còn yếu hoặc false positive / false negative
- Kết luận mức độ cải thiện an toàn

### 3. HITL Flowchart

Flowchart nên thể hiện:

- 3 decision points
- Điều kiện kích hoạt từng nhánh
- Khi nào hệ thống tự động xử lý
- Khi nào cần human review
- Kết quả cuối cùng sau review

---

## Thứ tự làm bài khuyến nghị

Nếu muốn làm nhanh mà vẫn chắc, nên đi theo thứ tự này:

1. Tạo agent chưa bảo vệ và viết 5 attack prompts
2. Chạy thử để thấy agent fail như nào
3. Viết input guardrails
4. Viết output guardrails
5. Tích hợp guardrails vào ADK
6. Thử thêm NeMo Guardrails
7. Chạy lại toàn bộ attack cases
8. Viết testing pipeline tự động
9. Tổng hợp thành Security Report
10. Thiết kế HITL flowchart và confidence router

---

## Checklist hoàn thành nhanh

- [ ] Có ít nhất 5 adversarial prompts
- [ ] Có thêm test cases sinh bằng AI
- [ ] Có injection detector
- [ ] Có topic filter
- [ ] Có input guardrail plugin
- [ ] Có output content filter
- [ ] Có LLM-as-Judge
- [ ] Có output guardrail plugin
- [ ] Có demo hoặc config NeMo Guardrails
- [ ] Có rerun 5 attacks sau phòng vệ
- [ ] Có automated testing pipeline
- [ ] Có confidence router
- [ ] Có HITL flowchart với ít nhất 3 decision points
- [ ] Có Security Report

## Tiêu chuẩn tự kiểm tra trước khi nộp

Bạn có thể tự hỏi 5 câu này:

1. Nếu giảng viên nhìn vào bài, họ có thấy rõ `trước và sau khi phòng vệ` không?
2. Guardrails của bạn có thật sự chặn được một số case, hay chỉ tồn tại trong code?
3. Pipeline test có đủ dữ liệu để chứng minh hiệu quả phòng thủ không?
4. HITL flow có cho thấy khi nào AI phải nhường quyền cho con người không?
5. Bài của bạn có thể hiện tư duy `Responsible AI`, tức là không tin agent tuyệt đối, luôn có cơ chế kiểm soát rủi ro không?

Nếu trả lời chắc được cả 5 câu trên thì bài Day 11 đã tương đối đầy đủ.
