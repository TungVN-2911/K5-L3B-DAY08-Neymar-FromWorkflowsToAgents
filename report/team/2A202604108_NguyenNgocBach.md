# Báo cáo cá nhân — 2A202604108_NguyenNgocBach

## 1. Thông tin thành viên

- Họ tên: Nguyễn Ngọc Bách
- MSSV: 2A202604108
- Email: bachnguyenfptu@gmail.com
- GitHub: https://github.com/bachnguyen2001

## 2. Phần việc phụ trách

- Triển khai Shape A (Deterministic Workflow) trong `src/lab8/shapes/workflow.py`.
- Tích hợp và triển khai Shape B (Single ReAct Agent) trong `src/lab8/shapes/single.py`.
- Tích hợp kiểm thử, cấu hình môi trường thực thi và xử lý hợp đồng trace/idempotency/crash safety cho workflow và single agent.

## 3. Đóng góp cụ thể

- Triển khai hoàn chỉnh `src/lab8/shapes/workflow.py`: đọc claim qua `get_claim_view`, thẩm định điều kiện `order_id`, bảo vệ privacy ranh giới thông tin với `lookup_order`, tính bồi thường bằng `compute_compensation`, hoàn tiền chuẩn với `durability.issue_refund`, hỗ trợ xử lý crash window (`crash_at`).
- Triển khai hoàn chỉnh `src/lab8/shapes/single.py`: dựng vòng lặp ReAct agent (`run_react`) có giới hạn 8 bước (`max_steps=8`), gọi các read-only tools độc lập, tính toán tiền bồi thường chính xác bằng code, ủy quyền write cho host an toàn.
- Cập nhật cơ chế cập nhật thời gian ghi trace (`lab8/trace.py`) nhằm đảm bảo tính toàn vẹn và chống stale trace trên môi trường Linux/WSL.
- Vượt qua toàn bộ 62/62 test case student cho `workflow` và 62/62 test case student cho `single`, bao gồm các kịch bản quan trọng: T01–T10, T11 (idempotency retry), T12 (crash before checkpoint does not double pay), T13 (order already refunded cap), T14 (pay only difference).

## 4. Bằng chứng

- Kiểm thử ma trận sinh viên:
  - `python -m pytest -q -m student -o addopts= -k workflow` -> 62 passed.
  - `python -m pytest -q -m student -o addopts= -k "single and not orchestrator and not swarm"` -> 62 passed.
  - `test_t11_retry_keeps_one_refund[workflow]` -> PASSED.
  - `test_t11_retry_keeps_one_refund[single]` -> PASSED.
  - `test_t12_crash_before_checkpoint_does_not_double_pay[workflow]` -> PASSED.
  - `test_t12_crash_before_checkpoint_does_not_double_pay[single]` -> PASSED.
- Kết quả kiểm tra từ bộ chấm `lab8.check_submission`:
  - `[ OK ] shape workflow: format đúng, 12/12 claim public pass ở seed 11`
  - `[ OK ] shape single: format đúng, 12/12 claim public pass ở seed 11`

## 5. Công cụ AI đã dùng

- Google Antigravity (Gemini): Hỗ trợ phân tích test fail, đối soát nghiệp vụ LogiX-VN, sinh mã nguồn ban đầu cho Shape A và Shape B theo đúng spec và contracts.
- Tự kiểm chứng: Chạy toàn bộ ma trận pytest (`pytest -m student`), kiểm tra log trace json sinh ra và xác thực với `lab8.check_submission`.

## 6. Tự đánh giá

- Hoàn thành đầy đủ chỉ tiêu của Shape A (Workflow) và Shape B (Single Agent).
- Đảm bảo tính nhất quán của hệ thống sổ cái (ledger), không để lộ PII và tuân thủ các quy định bảo mật bồi thường.
