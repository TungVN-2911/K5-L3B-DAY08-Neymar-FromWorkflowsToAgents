# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: dbd3761
- Run id dẫn chứng: R-workflow-C-002-s55 (ca swarm fail, workflow pass); toàn bộ 70 trial `R-workflow-*` trong src/reports/benchmark.json
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000–1.000 / 0.000000 (public, 14 claim × 5 seed = 70 trial, src/reports/benchmark.json)
- Cost per success (mô phỏng, VND): 28.79
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0 trên bộ public (70 trial benchmark quét trace + test src/tests/test_ledger_extra.py, test_orchestrator_extra.py); số hidden do grader giảng viên chấm

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Trên cùng corpus public (14 claim), cùng 5 seed `11,22,33,44,55`, cùng MockLLM và bảng giá (`src/reports/benchmark.json`), cả ba shape nhóm viết đều đạt pass rate 1.000, pass^5 1.000, variance 0. Khác biệt nằm ở chi phí và mức tự chủ: workflow 28.79 VND/success (1 model call, 2.79 tool call trung bình), single 29.65, orchestrator 46.43, swarm 49.57. Swarm là shape duy nhất có variance khác 0 (pass 0.986, pass^5 0.929, seed 55 = 0.929) vì ở seed chia hết cho 5 nó tin số ngày trễ trong ticket (C-002 seed 55). Với task claims, mọi quyết định tiền đều tất định (đọc dòng order → `compute_compensation` → host), nên không có bước nào cần model chọn. Workflow là shape ít tự chủ nhất, rẻ nhất và không thua về độ đúng, nên nhóm chọn workflow.

## Có đáng đổi lấy multi-agent không

Không. Trên public, multi-agent không cho pass cao hơn: orchestrator bằng workflow về pass nhưng tốn hơn ~61% cost/success (46.43 so với 28.79) vì thêm lớp handoff và investigator; swarm vừa đắt hơn ~72% vừa có variance. Kể cả khi một shape multi-agent nhỉnh hơn vài phần trăm pass, nhóm chỉ đổi nếu (1) chênh lệch vượt dao động giữa seed và lặp lại trên hidden, (2) không đổi bằng cost và variance cao hơn, (3) vẫn giữ 3 must-pass. Quyết định sẽ đảo lại nếu task có phần thật sự cần phán đoán ngôn ngữ (ví dụ phải đọc ảnh/biên bản để xác minh hỏng hàng, hoặc nhiều nguồn dữ kiện độc lập điều tra song song được), khi đó investigator song song mới có lợi.

## Trace và bằng chứng

Trace workflow ghi 3 loại event: `order_lookup` (chỉ `visible` và `reason`), `quote` (output của `compute_compensation`: amount, reason, clause_ids, parts) và `decision` (status cuối, reason); nhánh sớm ghi `error` hoặc `decision` `needs_info`/`escalated`; khi crash ghi `crash` rồi flush trước khi ném `InjectedCrash`. Cố ý **không** ghi: ticket (chữ người dùng, có thể chứa PII hoặc câu "ignore policy"), claim row (có `parent_secret`, `raw_headers_json`), order row thô (SĐT, địa chỉ, tên hàng, internal note), `principal_id`. Orchestrator ghi handoff đã qua `make_handoff`/`sanitize` (allowlist field, chỉ tool đọc, `facts` rỗng), không ghi parent context. Nếu trace nằm ngoài chỗ scanner quét được thì không chứng minh được là sạch, nên nhóm giữ trace đúng contract v1 tại `traces/<run_id>.json`, không ghi file phụ hay stdout, và có test tự động quét `CANARY_SECRET_` trên cả 5 seed (`test_trace_and_result_never_leak_canary`).

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Cost là VND mô phỏng theo `config/pricing.yaml`, không phải hóa đơn API. Số liệu là bộ public 14 claim; bộ public dễ hơn hidden nên pass 1.000 không chứng minh hidden đạt 1.000. Không sửa JSON cho đủ 0.75.
