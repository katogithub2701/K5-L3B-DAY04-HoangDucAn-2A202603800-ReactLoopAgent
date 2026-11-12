# Ghi chú phân tích · Day 04

Họ và tên: Hoàng Đức An
MSSV: 2A202603800

## 1. Case được chọn

- ID case: A5
- Câu trả lời của agent: 
```text
## Không thể cung cấp danh sách đầy đủ incident GitHub Actions năm 2025 bằng dữ liệu hiện có

Lý do (dựa trên tool, không suy đoán):

- Tool get_status chỉ trả về 50 incident gần nhất, và trả về kèm ghi chú:
  "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố."
- Thời điểm snapshot: 2026-11-09T03:50:47Z.
- Toàn bộ incident trong cửa sổ 50 sự cố này có updated_at nằm trong khoảng
  2026-10 đến 2026-11-08 — tức là năm 2025 nằm hoàn toàn ngoài cửa sổ dữ liệu
  mà tool truy xuất được.
- Tôi đã thử hai cách: tìm theo component "GitHub Actions"
  (11 incident khớp trong 50 incident gần nhất) và không lọc component
  (đủ 50 incident). Cả hai đều không chứa dữ liệu 2025.

Kết luận về độ đầy đủ: 0/12 tháng của năm 2025.
```
- Điểm ở lần chạy 1: 92/100

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
        [
          {
            "event": "step",
            "step": 1,
            "finish": "tool_calls"
          },
          {
            "event": "tool",
            "step": 1,
            "name": "get_status",
            "arguments": {
              "provider": "github",
              "component": "GitHub Actions"
            },
            "result": {
              "provider": "github",
              "captured_at": "2026-11-09T03:50:47.065020+00:00",
              "overall": {
                "indicator": "none",
                "description": "All Systems Operational"
              },
              "components": [],
              "incidents": [
                {
                  "id": "t89g2q1800fr",
                  "name": "Incident with Actions",
                  "status": "resolved",
                  "updated_at": "2026-11-05T17:42:35.455Z"
                },
                {
                  "id": "xr3myzj7ms51",
                  "name": "Incident with Actions and Pages",
                  "status": "resolved",
                  "updated_at": "2026-11-07T01:04:38.647Z"
                },
                {
                  "id": "ntrfd2jqldn6",
                  "name": "Delayed Actions status check results",
                  "status": "resolved",
                  "updated_at": "2026-10-28T22:29:49.032Z"
                },
                {
                  "id": "q29p76qtnkyk",
                  "name": "Incident with Actions",
                  "status": "resolved",
                  "updated_at": "2026-11-05T00:04:42.627Z"
                },
                {
                  "id": "8tzbpcnl75jp",
                  "name": "Degraded availability for GitHub Actions, GitHub Pages and Webhooks",
                  "status": "resolved",
                  "updated_at": "2026-10-22T13:12:51.444Z"
                }
              ],
              "matching_incidents_in_last_50": 11,
              "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.",
              "source": "https://www.githubstatus.com"
            }
          },
          {
            "event": "step",
            "step": 2,
            "finish": "tool_calls"
          },
          {
            "event": "tool",
            "step": 2,
            "name": "get_status",
            "arguments": {
              "provider": "github"
            },
            "result": {
              "provider": "github",
              "captured_at": "2026-11-09T03:50:47.065020+00:00",
              "overall": {
                "indicator": "none",
                "description": "All Systems Operational"
              },
              "components": [],
              "incidents": [
                {
                  "id": "pyynvrm1vskw",
                  "name": "Increase in timeouts for some unauthenticated users",
                  "status": "resolved",
                  "updated_at": "2026-11-08T12:32:35.646Z"
                },
                {
                  "id": "pfbmpxxfq1cy",
                  "name": "Degraded performance for multiple GitHub services",
                  "status": "resolved",
                  "updated_at": "2026-11-06T00:20:49.369Z"
                },
                {
                  "id": "t89g2q1800fr",
                  "name": "Incident with Actions",
                  "status": "resolved",
                  "updated_at": "2026-11-05T17:42:35.455Z"
                },
                {
                  "id": "h02n9572mx9y",
                  "name": "Incident with Actions",
                  "status": "resolved",
                  "updated_at": "2026-11-07T01:10:18.506Z"
                },
                {
                  "id": "xr3myzj7ms51",
                  "name": "Incident with Actions and Pages",
                  "status": "resolved",
                  "updated_at": "2026-11-07T01:04:38.647Z"
                }
              ],
              "matching_incidents_in_last_50": 50,
              "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.",
              "source": "https://www.githubstatus.com"
            }
          },
          {
            "event": "step",
            "step": 3,
            "finish": "stop"
          }
        ]
```

- Nguyên nhân lỗi: Agent chưa kiểm soát tốt mức độ chắc chắn của kết luận so với phạm vi dữ liệu thực tế mà tool cung cấp.

## 3. Thay đổi đã thực hiện

- **Thay đổi SYSTEM_PROMPT** (`agent.py:3`): bổ sung ràng buộc nêu rõ dữ liệu tool trả về chỉ là mẫu/cửa sổ giới hạn; không khẳng định "toàn bộ" khi tool chỉ trả một phần; không đưa danh sách ngoài phạm vi câu hỏi nếu không cần.

- **Lưu URL nguồn** (`agent.py:134-138`): khi tool `get_incident_details` trả thành công (`ok`) và có `url`, lưu URL vào `observed_sources` (không trùng lặp).

- **Evidence gate** (`agent.py:160-185`): khi `finish == "stop"` mà đã có URL quan sát nhưng answer không chứa URL nào, ghi event `validation_missing_source` vào trace, gửi message yêu cầu model sửa và tiếp tục `max_steps`; nếu hết lượt thì trả `max_steps` thay vì `completed`. Không bắt URL khi không có incident thành công.

- **Vì sao chọn thay đổi này:** (1) Sửa đúng nguyên nhân điểm thấp của A5 (kết luận vượt quá phạm vi bằng chứng); (2) Agent thiếu thông tin về các urls để kiểm chứng;

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 98.95 | 99.0 |
| Điểm case đã chọn | 92 | 96 |
| Số model calls | 62 (22 case) / 3 (A5) | 65 (22 case) / 4 (A5) |
| Token, chi phí nếu có | 127.065 tokens / A5: 5.045 tokens | 153.981 tokens / A5: 12.863 tokens |

- Lỗi đã giải quyết:
  - A5 tăng 92 → 96; các chiều tăng: factuality 35 → 38, evidence 18 → 19 (completeness 24 và boundaries 15 giữ nguyên).
  - Nguyên nhân điểm thấp cũ ("khẳng định toàn bộ 50 incident" khi tool chỉ trả mẫu) đã được xử lý: câu trả lời after chỉ nêu "toàn bộ quan sát được là tháng 11/2026", diễn giải đúng `matching_incidents_in_last_50 = 23` là số khớp chuỗi, không dựng danh sách 2025.
  - Agent dùng thêm `get_incident_details` (2 → 5 tool calls) nên có ID, thời gian, component, URL `stspg.io` và nguồn `githubstatus.com` để kiểm chứng; trung thực thừa nhận incident chưa có URL.

- Lỗi còn lại hoặc mới xuất hiện:
  - Điểm tổng chỉ tăng +0.05 (98.95 → 99.0) trong khi 7 case giảm: E1, E2, E4 (100 → 97), A2, H5, H6 (100 → 99), H8 (100 → 94). Lý do chủ yếu là judge trừ nhẹ vì khẳng định quá mức hoặc dữ liệu bị cắt, không phải lỗi grounding mới.
  - H8 giảm mạnh nhất (100 → 94): câu trả lời after chủ yếu là bản sửa URL, không trình bày lại đầy đủ giới hạn 2020–2026.
  - A5 vẫn còn điểm trừ: chưa đề xuất cụ thể endpoint Statuspage API kèm tham số thời gian để lấy lịch sử 2025.
  - Chi phí tăng: tổng token tăng từ 127.065 → 153.981 (+21%), list cost ~$0.0295 → ~$0.0689 (~2,3×) do evidence gate gọi lại model và tăng tool calls.

## 5. Giới hạn

- **Giới hạn của điểm judge:**
  - Judge là LLM và dùng **cùng model với agent** nên điểm mang tính xấp xỉ và có thể thiên lệch theo văn phong/độ dài câu trả lời.
  - Điểm là tổng hợp 4 chiều chủ quan (factuality 40, completeness 25, evidence 20, boundaries 15) trên 22 case; một số khác biệt nhỏ (ví dụ E1/E2/E4 100 → 97) nằm trong nhiễu đánh giá hơn là thay đổi năng lực thực sự.
  - Case H8 giảm 100 → 94 chủ yếu do judge nhận xét cách trình bày, không hẳn là lỗi dữ kiện; cần đọc `reason`/`improvement` thay vì chỉ nhìn điểm.

- **Giới hạn của snapshot:**
  - Snapshot cố định tại `captured_at = 2026-11-09T03:50:47Z`; câu trả lời chỉ đúng cho thời điểm này, không đại diện trạng thái hiện tại.
  - `get_status` chỉ trả **50 incident gần nhất**; với A5/H8, dữ liệu 2025 hoàn toàn nằm ngoài cửa sổ nên không thể kết luận "không có sự cố" hay lập danh sách đầy đủ cho 2025. "Không tìm thấy" không đồng nghĩa "không tồn tại".
  - Fixture có thể bị cắt: `updates_total` lớn hơn số update hiển thị, `truncated: true`, và `components` đôi khi rỗng — rỗng không chứng minh được là không có dịch vụ bị ảnh hưởng.
  - Trace trong `eval_*.json` chỉ chứa các bước agent thực sự gọi; nó không chứng minh agent đã "kiểm tra toàn bộ" dữ liệu ngoài phạm vi tool.

- **Giới hạn vì chỉ chạy một lần mỗi phiên bản:**
  - Mỗi phiên bản chỉ chạy **một lần** trên cùng bộ 22 case với cùng cấu hình; chênh lệch điểm tổng +0.05 chưa đủ kết luận cải tiến ổn định.
  - Không có nhiều seed/lần lặp nên không đo được phương sai do sampling của model và judge; một số case tăng/giảm có thể do ngẫu nhiên.
  - Thay đổi evidence gate làm tăng số model calls (62 → 65) và token (127.065 → 153.981); chi phí tăng ~2,3× là đánh đổi cần cân nhắc nếu chạy ở quy mô lớn.
  - Để kết luận chắc chắn hơn cần chạy lặp (≥3 lần/phiên bản), giữ nguyên fixture và so sánh phân bố điểm, không chỉ một con số trung bình.
