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
- Thử sửa `lab8/trace.py` (cập nhật mtime khi ghi trace) để chống stale trace trên Linux/WSL; sửa này đã được gỡ (commit ffc23fc) vì grader chỉ lấy 3 file shape, và workflow/single vẫn pass với `trace.py` gốc.
- `7d8eb4c` thêm `src/tests/test_student_cases.py`: 4 test cho workflow (thiếu order_id → needs_info, đơn không thuộc principal → escalated) và single (trễ < 3 ngày → denied, C-001 resolved 35.000 với một dòng ledger).
- Vượt qua toàn bộ 62/62 test case student cho `workflow` và 62/62 test case student cho `single`, bao gồm các kịch bản quan trọng: T01–T10, T11 (idempotency retry), T12 (crash before checkpoint does not double pay), T13 (order already refunded cap), T14 (pay only difference).

## 4. Bằng chứng

- File test tự viết: `src/tests/test_student_cases.py` (chạy lệnh: `pytest tests/test_student_cases.py` -> 4 passed):
  - `test_workflow_missing_order_id_triggers_needs_info`: Xác thực Workflow phát hiện thiếu `order_id` và trả về `needs_info` an toàn, không ghi ledger bậy.
  - `test_workflow_escalation_for_unauthorized_order`: Xác thực Workflow chuyển cấp `escalated` khi tài khoản không sở hữu đơn hàng, bảo mật PII.
  - `test_single_react_agent_denied_below_threshold`: Xác thực Single ReAct Agent thẩm định đúng điều kiện trễ < 3 ngày và trả về `denied`.
  - `test_single_react_agent_successful_refund`: Xác thực vòng lặp ReAct gọi các tool tuần tự, tính toán bồi thường và hoàn tiền thành công 35.000 VND.
- Kiểm thử ma trận sinh viên:
  - `python -m pytest -q -m student -o addopts= -k workflow` -> 62 passed.
  - `python -m pytest -q -m student -o addopts= -k "single and not orchestrator and not swarm"` -> 62 passed.
  - `test_t11_retry_keeps_one_refund[workflow]` -> PASSED.
  - `test_t11_retry_keeps_one_refund[single]` -> PASSED.
  - `test_t12_crash_before_checkpoint_does_not_double_pay[workflow]` -> PASSED.
  - `test_t12_crash_before_checkpoint_does_not_double_pay[single]` -> PASSED.
- Test riêng: `python -m pytest -q tests/test_student_cases.py` -> 4 passed.
- Kết quả kiểm tra từ bộ chấm `lab8.check_submission`:
  - `[ OK ] shape workflow: format đúng, 12/12 claim public pass ở seed 11`
  - `[ OK ] shape single: format đúng, 12/12 claim public pass ở seed 11`

## 5. Công cụ AI đã dùng

- Google Antigravity (Gemini): Hỗ trợ phân tích test fail, đối soát nghiệp vụ LogiX-VN, sinh mã nguồn ban đầu cho Shape A và Shape B theo đúng spec và contracts.
- Tự kiểm chứng: Chạy toàn bộ ma trận pytest (`pytest -m student`), kiểm tra log trace json sinh ra và xác thực với `lab8.check_submission`.

## 6. Tự đánh giá

- Hoàn thành đầy đủ chỉ tiêu của Shape A (Workflow) và Shape B (Single Agent).
- Đảm bảo tính nhất quán của hệ thống sổ cái (ledger), không để lộ PII và tuân thủ các quy định bảo mật bồi thường.
