# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R1 · Scaling |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Kim et al., *Towards a Science of Scaling Agent Systems*, Google Research / Google DeepMind / MIT, arXiv 2512.08296 (12/2025), https://arxiv.org/abs/2512.08296; blog Google Research 28/01/2026. Loại bằng chứng: benchmark có kiểm soát (nhiều cấu hình agent × 6 benchmark × 3 họ LLM) |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Thêm agent không giúp một cách đáng tin cậy: khi topology phối hợp không khớp với cấu trúc task thì hiệu năng giảm, và giao tiếp giữa agent có "coordination tax" (token và nhiễu) bù trừ lợi ích của suy luận song song |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Đo trên 6 benchmark agentic và các LLM thật (GPT, Gemini, Claude) với 5 kiến trúc chuẩn; không đo task có tiền/approval do host quyết, không đo MockLLM tất định. Kết luận là xu hướng thống kê, không phải định luật cho mọi task |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Task claims chủ yếu tuần tự (claim → order → quote → host), không có nhánh song song có giá trị. Đổi workflow → orchestrator (thêm lớp handoff + investigator) sẽ giữ nguyên pass rate (±0) và tăng cost/success khoảng 40–80% do tool call phối hợp |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Cố định: MockLLM, tool registry manifest v1.1, seed 11,22,33,44,55, corpus data/public_claims.jsonl (14 claim), config/pricing.yaml, evaluator lab8.metrics.trial_pass, SQLite mới mỗi trial. Biến duy nhất được thay đổi: topology — workflow (một luồng) so với orchestrator (orchestrator + investigator chỉ đọc qua handoff) |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến: coordination tax làm tăng chi phí mà không tăng độ đúng. Đo bằng cost/success và mean tool calls; kiểm kèm pass rate, pass^5, variance để chắc không đổi độ đúng; leak/bypass phải giữ 0 |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,orchestrator --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/ablation_r1_workflow_vs_orchestrator.json`
- Run id / file report: `src/reports/ablation_r1_workflow_vs_orchestrator.json` (run id `R-workflow-*` và `R-orchestrator-*`, ví dụ `R-orchestrator-C-002-s55`); khớp số trong `src/reports/benchmark.json`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: Đúng. Pass rate 1.000 → 1.000, pass^5 1.000 → 1.000, variance 0 → 0; cost/success 28.79 → 46.43 VND (+61%); mean tool calls 2.79 → 4.64. Mean steps trong trace bằng nhau (2.79), nên chênh lệch đến từ lớp phối hợp chứ không từ số bước quyết định
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`. Corpus public chỉ 14 claim và MockLLM tất định nên không đo được lợi ích tiềm năng của investigator khi model thật mắc lỗi; orchestrator của nhóm không gọi model nên chênh lệch cost ở đây là tool call, không phải token.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
