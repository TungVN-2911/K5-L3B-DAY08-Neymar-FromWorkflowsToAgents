# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: Neymar
- Thành viên: xem `TEAM.md` (Vũ Ngọc Tùng, Nguyễn Ngọc Bách)
- Commit chốt (trùng `report/team/decision_adr.md`): eee64a9

## 2. Mục tiêu và cách tiếp cận

Bài toán: xử lý claim hoàn phí vận chuyển bằng bốn kiến trúc cùng chữ ký `run_claim`, rồi chọn kiến trúc ít tự chủ nhất mà vẫn đạt. Nhóm hiểu ranh giới cốt lõi là **tiền, approval, idempotency do code và host quyết; model chỉ đọc và soạn chữ**, và chữ trong ticket là dữ liệu người dùng, không phải lệnh.

Nhóm 2 người, chia theo file để tránh conflict:

- Người 1 (Nguyễn Ngọc Bách): workflow + integration; single.
- Người 2 (Vũ Ngọc Tùng): orchestrator + handoff; ledger + evaluation (crash, approval, benchmark).

Thứ tự: CP1 cài đặt và xác nhận baseline (smoke pass, cổng host xanh, ma trận student 186 failed / 62 passed) → hai người viết shape song song trên file riêng → merge → mở rộng test ledger sang cả 3 shape → crash suite, benchmark, ablation → báo cáo chung.

## 3. Giải pháp

| Shape | Đọc gì | Quyết định gì | Gọi host |
| --- | --- | --- | --- |
| A `workflow` | `get_claim_view`, `lookup_order` | Các bước Python cố định: thiếu `order_id` → `needs_info`; không thấy/không sở hữu đơn → `escalated`; quote bằng `compute_compensation` từ dòng order; quote 0 → `denied` | Quote > 0 → `durability.issue_refund(conn, claim_id, <quote đầy đủ>, crash_at=crash_at)` |
| B `single` | Vòng `run_react` (tối đa 8 bước), `decide` chỉ chọn tool đọc `lookup_order` → `compute_compensation` | Model (MockLLM) chỉ chọn bước đọc; không chọn số tiền, không có tool ghi | Sau vòng lặp, code của shape gọi `issue_refund` với quote đầy đủ |
| C `orchestrator` | `get_claim_view`; handoff → `run_readonly_investigators` | Orchestrator đọc order view và quote mà investigator trả về | Orchestrator gọi `issue_refund`; investigator không có write registry |

Ranh giới code / model: không shape nào để MockLLM quyết tiền; tiền luôn từ `compute_compensation`. Shape luôn truyền **quote đầy đủ** để host tự trừ phần đã hoàn của đơn (T13, T14), và luôn gọi `issue_refund` khi quote > 0 để host ghi audit `policy_denied` khi từ chối. `crash_at` được chuyển thẳng xuống host và `InjectedCrash` không bị bắt (trace được flush trước khi ném lại).

Handoff (orchestrator): dựng bằng `lab8.handoff.make_handoff`, chỉ field trong allowlist, `facts` rỗng (không ticket, không principal, không claim row), `allowed_tools` chỉ gồm tool đọc; `sanitize` từ chối nếu còn `CANARY_SECRET_`/`parent_secret`/`raw_headers`.

Trace: mọi nhánh return đi qua `common.finish()`, ghi `traces/<run_id>.json` đúng contract v1. Event gồm `order_lookup`/`investigator`/`tool_result` (chỉ view đã allowlist), `quote`, `decision`, `handoff` (đã sanitize), `crash`. Không ghi ticket, claim row, order row thô.

## 4. Kết quả

Benchmark public (`src/reports/benchmark.json`): 14 claim × 5 seed `11,22,33,44,55`, MockLLM, `as_of` 2026-11-15, SQLite mới mỗi trial.

| Shape | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success (VND mô phỏng) | Mean / p95 steps |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| workflow | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 28.79 | 2.79 / 3 |
| single | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 29.65 | 3.64 / 4 |
| orchestrator | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 46.43 | 2.79 / 3 |
| swarm | 0.986 | 0.929 | 0.929–1.000 | 0.000816 | 49.57 | 2.80 / 3 |

Trial fail duy nhất: swarm `R-swarm-C-002-s55` (`status resolved != denied`), đúng điểm yếu tham chiếu (tin số ngày trễ trong ticket ở seed chia hết cho 5).

- Crash safety: **20/20** (`src/reports/crash.json`).
- Ma trận student (`pytest -m student`): 248 passed trên 4 shape. Cổng host `pytest -q` xanh, gồm 33 test nhóm thêm: 29 của orchestrator/ledger (crash 3 điểm, approval hỏng, secret, ticket override, trần theo đơn, chạy trên cả 3 shape) và 4 của workflow/single (`test_student_cases.py`: needs_info, escalated, denied, resolved).
- Tự kiểm `student_drop/` (`src/reports/hidden_selfcheck.json`, 40 claim × 5 seed × 3 shape = 600 trial): 0 trial lỗi, 0 `leak_suspect`, cả 3 shape cho cùng `status` và `proposal_amount` trên mọi seed. Bộ này không có nhãn nên không ra pass rate; nhóm đối chiếu tay 40 claim của workflow với `docs/business_rules.md` và không thấy claim nào lệch luật. Điểm hidden chính thức do grader giảng viên chấm.

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md` (chọn **workflow**). Phiếu nghiên cứu và ablation: xem `report/team/research_card.md` (R1: workflow → orchestrator giữ pass, tăng cost/success +61%). Bonus B3 (verifier cho swarm): xem `report/team/bonus_report.md` (swarm_verified về bằng workflow: public 0.986 → 1.000, bộ đối kháng tự viết 0.900 → 1.000, nhưng vẫn đắt hơn workflow ~70%).

## 5. Khó khăn và cách xử lý

- **Orchestrator chép từ swarm ghi sai tên shape.** Bản đầu dùng `make_run_id("swarm", ...)` và `shape="swarm"` nên `run_id`/`shape` của kết quả sai shape. Sửa bằng một hằng `SHAPE = "orchestrator"`, đồng thời bỏ khối `REFERENCE_WEAKNESS` và đoạn tính lại delay.
- **Sửa file ngoài phạm vi grader.** Một commit có sửa `src/lab8/trace.py` (thêm `os.utime`). Grader chỉ lấy 3 file shape từ repo nhóm, nên sửa này bị bỏ qua và bị ghi tampered. Nhóm khôi phục `trace.py` về bản gốc và chạy lại: ma trận student vẫn 248 passed, cổng host vẫn xanh, nên shape không phụ thuộc sửa đó.
- **Chạy test sai cách.** `python tests/test_orchestrator_extra.py` từ `src/tests/` báo `ModuleNotFoundError: No module named 'lab8'`. Nguyên nhân: file test phải chạy qua pytest và từ `src/` (`pytest.ini` có `pythonpath = .`). Chạy `python -m pytest -q tests/...` trong `src/` thì pass.
- **Phiên bản Python.** Máy dùng Python 3.14.3, đề yêu cầu 3.11–3.13; mọi lệnh chạy được nhưng CI/máy chấm dùng bản trong khoảng yêu cầu.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`; cost là VND mô phỏng. Bộ public chỉ 14 claim và dễ hơn hidden, nên pass 1.000 trên public không chứng minh hidden đạt 1.000. Vì MockLLM tất định, benchmark không đo được trường hợp model thật đọc sai tool output; với model thật, single/orchestrator có thể có variance mà workflow không có. Kết luận "chọn workflow" sẽ đổi nếu task cần phán đoán ngôn ngữ thật (xác minh hỏng hàng từ mô tả/ảnh) hoặc có nhiều nguồn dữ kiện độc lập điều tra song song được. Hướng cải thiện: thêm claim adversarial (ticket trích policy cũ, mã đơn trong chữ ticket) vào test.

## 8. Tài liệu tham khảo

- Kim et al., *Towards a Science of Scaling Agent Systems*, arXiv 2512.08296, 2025. https://arxiv.org/abs/2512.08296
- Google Research blog, *Towards a science of scaling agent systems: When and why agent systems work*, 2026. https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/
- Tài liệu lab: `docs/business_rules.md`, `docs/contracts.md`, `docs/metrics.md`, `docs/architecture.md`.
