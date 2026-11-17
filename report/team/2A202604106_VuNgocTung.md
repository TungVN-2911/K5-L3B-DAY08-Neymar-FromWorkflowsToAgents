# Báo cáo cá nhân — 2A202604106_VuNgocTung

## 1. Thông tin thành viên

- Họ tên: Vũ Ngọc Tùng
- MSSV: 2A202604106
- Email: vungoctung098@gmail.com
- GitHub: TungVN-2911

## 2. Phần việc phụ trách

Người 2 trong nhóm 2 người: shape C `orchestrator` và handoff (`src/lab8/shapes/orchestrator.py`); ledger (crash, idempotency, approval) trên cả 3 shape; evaluation (crash suite, benchmark, ablation R1).

## 3. Đóng góp cụ thể

- `491b19e` implement orchestrator: orchestrator đọc claim qua `get_claim_view`, dựng handoff bằng `make_handoff` (allowlist, `facts` rỗng, chỉ tool đọc), chạy `run_readonly_investigators`, quote từ `compute_compensation` và gọi `durability.issue_refund` với quote đầy đủ, chuyển `crash_at` và không bắt `InjectedCrash`. Bỏ điểm yếu tham chiếu của swarm (tin số ngày trễ trong ticket).
- `9c801da` `src/tests/test_orchestrator_extra.py`: 4 test (8 ca) — handoff trong trace chỉ có field allowlist và tool đọc; trace/kết quả không lộ `CANARY_SECRET_` trên 5 seed; C-002 seed 55 phải `denied` (ca swarm fail); T14 chỉ hoàn phần chênh 120.000.
- `2626f09` `src/tests/test_ledger_extra.py`: crash ở 3 điểm rồi resume vẫn một refund và một audit committed; approval bị thu hồi / của claim khác / thiếu → `denied`, quote đầy đủ, có `policy_denied` (`approval_claim_mismatch`); approval hết hạn C-007. Sau khi merge, mở rộng sang cả workflow, single, orchestrator (21 ca).
- Evaluation: chạy crash suite (20/20), benchmark 4 shape × 5 seed, ablation R1 workflow vs orchestrator.
- Quyết định kỹ thuật: phát hiện commit có sửa `src/lab8/trace.py` (ngoài 3 file grader lấy) và khôi phục về bản gốc sau khi chạy lại xác nhận các shape không phụ thuộc sửa đó.

## 4. Bằng chứng

- Ma trận student: `python -m pytest -q -m student -o addopts=` → 248 passed; orchestrator 62/62.
- Cổng host `python -m pytest -q` xanh (87 passed), trong đó 29 test do mình thêm.
- `src/reports/crash.json`: `crash_safety` 20/20.
- `src/reports/benchmark.json`: orchestrator pass 1.000, pass^5 1.000, variance 0, cost/success 46.43 VND; run id ví dụ `R-orchestrator-C-002-s55`.
- `src/reports/ablation_r1_workflow_vs_orchestrator.json`: cost/success 28.79 → 46.43 (+61%), pass không đổi.

## 5. Công cụ AI đã dùng

Claude Code (Claude): đọc và tóm tắt đề, gợi ý cách chia vai tránh conflict, hướng dẫn từng bước và viết bản code orchestrator cùng hai file test, soạn nháp báo cáo. Cách tự kiểm tra: chạy lại `pytest` (cổng host và ma trận student), chạy thử "phá" handoff để thấy test bắt lỗi, đối chiếu mọi số trong báo cáo với `src/reports/*.json` do công cụ lab sinh ra; không sửa tay JSON và không lấy số từ AI.

## 6. Tự đánh giá

Hoàn thành phần orchestrator, handoff và các test ledger/approval/secret; must-pass được kiểm bằng test tự động trên cả 3 shape. Điểm cần cải thiện: chưa có bộ hidden `student_drop/` để tự kiểm, và nên viết thêm test claim adversarial (ticket trích policy cũ, mã đơn trong chữ ticket).
