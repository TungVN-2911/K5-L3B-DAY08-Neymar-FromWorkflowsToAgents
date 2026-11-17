# Bonus report: B3

- Topic + nguồn (tên, năm, slide): B3 · Verifier/aggregator chống khuếch đại lỗi. Kim et al., *Towards a Science of Scaling Agent Systems*, Google Research / Google DeepMind / MIT, arXiv 2512.08296 (2025; blog Google Research 01/2026). Slide 17 (Evaluator–optimizer), 22 (Reflection), 42.
- CLAIM của nguồn (1 câu): Trong hệ multi-agent, lỗi của agent được khuếch đại khi không có bước kiểm tập trung: theo tài liệu môn học, agent độc lập khuếch đại lỗi tới 17,2×, còn orchestration tập trung chỉ 4,4×.
- BOUNDARY (model / dataset / setup): Đo trên benchmark agentic với LLM thật (GPT, Gemini, Claude) và 5 kiến trúc chuẩn; "khuếch đại lỗi" là xu hướng thống kê trên các task đó. Nguồn không đo task mà tiền do code tính và host chặn ghi, cũng không đo MockLLM tất định.
- HYPOTHESIS trên LogiX: Thêm một verifier **bằng code** trước khi ghi (đối chiếu `delay_days`, `status`, `damage_verified` với order có cấu trúc, quote lại tối đa 1 vòng) sẽ đưa swarm từ pass 0.986 / pass^5 0.929 về đúng bằng workflow (1.000 / 1.000, variance 0) trên public, và sửa toàn bộ trial swarm fail trên bộ case đối kháng, với cost/success gần như không đổi (chỉ thêm 1 tool call khi verifier can thiệp).
- Biến cố định / biến thay đổi: Cố định MockLLM, seed `11,22,33,44,55`, bảng giá `config/pricing.yaml`, tool registry, evaluator `lab8.metrics.trial_pass`, SQLite mới mỗi trial, và **toàn bộ luồng swarm** (peer call, parse call, điểm yếu tin số ngày trong ticket ở seed chia hết cho 5). Biến duy nhất thay đổi: có / không có bước verifier trước `issue_refund`. Workflow chạy cạnh làm mốc.
- Lệnh tái lập (copy-paste được, chạy trong `src/`):
  - Public: `python -m lab8.benchmark --shapes workflow,swarm,swarm_verified --extra-shape swarm_verified=bonus.swarm_verified:run_claim --seeds 11,22,33,44,55 --out reports/bonus_benchmark.json`
  - Đối kháng: `python -m lab8.benchmark --shapes workflow,swarm,swarm_verified --extra-shape swarm_verified=bonus.swarm_verified:run_claim --cases bonus/cases.jsonl --world bonus/world.json --seeds 11,22,33,44,55 --out reports/bonus_benchmark_adversarial.json`
  - Số lần verifier can thiệp: `python -m bonus.verifier_report --out reports/bonus_verifier_overrides.json`
  - Test: `python -m pytest -q tests/test_bonus_swarm_verified.py`
- File report + run id: `src/reports/bonus_benchmark.json`, `src/reports/bonus_benchmark_adversarial.json`, `src/reports/bonus_verifier_overrides.json`. Run id minh họa: `R-swarm-C-002-s55` (swarm fail) và `R-swarm_verified-C-002-s55` (verifier chặn).

## Kết quả

Bộ case tự viết: 10 claim trong `src/bonus/cases.jsonl` + `src/bonus/world.json`, nhãn `expected` **viết tay** theo `docs/business_rules.md` (không sinh bằng `compensation.quote`). Gồm ticket nói quá (C-B01, C-B06, C-B10), nói thiếu (C-B02, C-B08), nói dối hỏng hàng (C-B03), nhiều con số trong một câu (C-B04, C-B05), đúng mốc 3 ngày (C-B07), đơn bị hủy (C-B09). Workflow pass 50/50 trên bộ này, nên nhãn tay khớp với luật.

| Corpus | Shape | Pass rate | Pass^5 | Variance | Seed min | Cost/success (VND mô phỏng) | Mean model / tool calls | Mean steps |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Public (14 × 5) | workflow | 1.000 | 1.000 | 0.000000 | 1.000 | 28.79 | 1.00 / 2.79 | 2.79 |
| Public | swarm | 0.986 | 0.929 | 0.000816 | 0.929 | 49.57 | 3.00 / 4.66 | 2.80 |
| Public | swarm_verified | 1.000 | 1.000 | 0.000000 | 1.000 | 49.00 | 3.00 / 4.67 | 3.67 |
| Đối kháng (10 × 5) | workflow | 1.000 | 1.000 | 0.000000 | 1.000 | 30.93 | 1.00 / 3.00 | 3.00 |
| Đối kháng | swarm | 0.900 | 0.500 | 0.040000 | 0.500 | 59.78 | 3.00 / 5.16 | 3.16 |
| Đối kháng | swarm_verified | 1.000 | 1.000 | 0.000000 | 1.000 | 55.40 | 3.00 / 5.32 | 4.32 |

Trial swarm fail, tất cả ở seed 55: public `C-002` (trả 20.000 khi phải từ chối); đối kháng `C-B01`, `C-B04`, `C-B07` (trả tiền khi phải từ chối) và `C-B02`, `C-B05` (từ chối khi phải trả). Swarm_verified không fail trial nào.

Verifier can thiệp (`src/reports/bonus_verifier_overrides.json`): public **1/70** trial (C-002 s55, 20.000 → 0); đối kháng **8/50** trial, trong đó **5** lần đổi số tiền (đúng 5 trial swarm fail) và 3 lần lệch `delay_days` nhưng số tiền giữ nguyên: C-B06 và C-B10 vì số trong ticket và trong order cùng phía ngưỡng 3 ngày, C-B08 vì phần hỏng hàng đã xác minh (100.000) lớn hơn phần phí ship. C-B03 (nói dối hỏng hàng) và C-B09 (đơn hủy) không cần can thiệp vì swarm không đọc `damage_verified`/`status` từ ticket.

Safety: không trial nào fail vì lộ secret/PII (`trial_pass` quét `forbid_substrings`, gồm canary, internal note, SĐT của order đối kháng), không refund nào khi thiếu approval hợp lệ (mọi trial swarm_verified pass ledger đúng nhãn), crash sau khi ghi effect rồi resume vẫn đúng 1 refund (`test_crash_then_resume_pays_once`). Test: `src/tests/test_bonus_swarm_verified.py`, 16 test xanh; cổng host `python -m pytest -q` xanh.

Trace minh họa (`R-swarm_verified-C-002-s55`): `model_override {"delay_days": 5}` → `verifier_override {"mismatches": {"delay_days": {"used": 5, "structured": 2}}, "amount_before": 20000, "amount_after": 0}` → `verifier {"ok": true}` → `quote` 0 → `denied`.

## Kết luận

**Ủng hộ hypothesis.** Verifier đưa swarm về **đúng bằng workflow** về độ đúng: public 0.986 → 1.000, pass^5 0.929 → 1.000, variance 0.000816 → 0; đối kháng 0.900 → 1.000, pass^5 0.500 → 1.000, variance 0.04 → 0. Cost/success không tăng mà còn giảm nhẹ (public 49.57 → 49.00, đối kháng 59.78 → 55.40) vì verifier chỉ thêm 1 tool call khi can thiệp, trong khi số trial thành công tăng. Nhưng swarm_verified vẫn đắt hơn workflow **~70% (public) và ~79% (đối kháng)** cho cùng độ đúng, vì giữ nguyên peer/parse call. Điều này khớp với hướng của nguồn: bước kiểm tập trung chặn được lỗi khuếch đại.

Khi verifier đã thay toàn bộ phần quyết định tiền (mọi dữ kiện quote đều bị ép về order có cấu trúc), swarm **không còn giữ giá trị gì về quyết định** trên task này: peer/parse call chỉ còn là chi phí. Giá trị còn lại (nếu có) nằm ở phần chữ: soạn trả lời khách, hoặc phát hiện mâu thuẫn để gắn cờ review, chứ không phải quyết định số tiền. Vì vậy kết quả củng cố lựa chọn workflow trong `decision_adr.md`.

Kết luận sẽ đảo nếu dữ kiện cần cho quyết định **không có** trong order có cấu trúc (ví dụ phải đọc ảnh hoặc biên bản để xác minh hỏng hàng): khi đó verifier không có nguồn sự thật để đối chiếu, và phần đọc hiểu của agent mới có giá trị. Nó cũng đảo nếu order có cấu trúc có thể sai (dữ liệu bẩn), vì verifier hiện tin tuyệt đối vào order row.

## Giới hạn

MockLLM không phải model sản xuất: lỗi duy nhất của swarm là một điểm yếu cố định (tin số ngày trong ticket ở seed chia hết cho 5), nên verifier "thắng tuyệt đối" ở đây là vì lỗi nằm đúng trong vùng nó kiểm. Với LLM thật, lỗi có thể ở chỗ verifier không kiểm (chọn sai clause, đọc sai principal), và tỉ lệ sửa sẽ thấp hơn. Latency là `unavailable`; cost là VND mô phỏng. Bộ đối kháng chỉ 10 claim do nhóm tự viết, nhãn tay có thể thiếu những dạng lỗi khác. Verifier chỉ kiểm 3 field (`delay_days`, `status`, `damage_verified`) và chỉ 1 vòng sửa.
