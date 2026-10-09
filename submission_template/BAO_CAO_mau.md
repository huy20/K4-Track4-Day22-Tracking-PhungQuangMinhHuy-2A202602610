# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** 67 **Thành viên:** Phùng Quang Minh Huy 2A202602610

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | bytetrack | 0.50 | 0.7 | Giữ ID ổn định phần lớn thời gian; thỉnh thoảng hộp của người này nhảy sang người đi ngang rồi nhảy ngược lại (đổi ID khi hai người cắt nhau). | botsort — đổi ID nhiều hơn khi hai người giao nhau, không cải thiện so với bytetrack. |
| video_2 (phố đêm, tĩnh, rất đông) | ocsort | 0.30 | 0.4 | Đêm tối, mật độ rất đông: hộp bám được người rõ nhưng người nhỏ/ngược sáng bị bỏ sót; chênh lệch giữa các mức `conf`/`iou` nhỏ. | strongsort — chậm hơn rõ rệt trên cảnh đông mà ID không rõ hơn. |
| video_3 (camera di động, ảnh nhỏ) | botsort | 0.50 | 0.5 | Camera dịch chuyển + ảnh nhỏ: số ID mới sinh ra ít hơn hẳn so với tracker chỉ theo chuyển động; ID bám qua được lúc khung hình trôi. | ocsort — sinh nhiều ID mới mỗi khi camera dịch. |
| video_4 (trong nhà, camera di chuyển) | botsort | 0.50 | 0.7 | Trong nhà, camera tiến tới, có phản chiếu kính: track dính và theo người tốt; `conf` cao giúp bỏ bớt hộp giả trên phản chiếu. | bytetrack — mất track khi người rẽ hướng, hộp bám kém hơn. |
| video_5 (trên xe bus, giao lộ đông) | botsort | 0.25 | 0.6 | Xe bus rung lắc mạnh: botsort giữ ID tốt hơn; cần `conf` thấp (0.25) để không bỏ sót người nhỏ ở giao lộ. | ocsort — ID nhảy nhiều khi khung hình rung. |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

```
HOTA: video_1   HOTA 24.484 | DetA 13.192 | AssA 45.472 | DetRe 13.346 | DetPr 83.607 | AssRe 48.927 | AssPr 80.059 | LocA 85.184
CLEAR: video_1  MOTA 15.118 | MOTP 83.410 | IDSW 17 | CLR_Re 15.586 | CLR_Pr 97.640 | CLR_TP 2896 | CLR_FN 15685 | CLR_FP 70 | MT 6 | PT 8 | ML 48 | Frag 117
Identity: video_1  IDF1 23.270 | IDR 13.492 | IDP 84.525 | IDTP 2507 | IDFN 16074 | IDFP 459
```

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

**video_1 (quảng trường, camera tĩnh, ban ngày).** Tracker chuyển động `bytetrack` cho ID ổn định nhất, nhưng số liệu cho thấy `CLR_Pr` rất cao (97.6%) trong khi `CLR_Re` chỉ 15.6%: gần như không sinh hộp giả nhưng **bỏ sót rất nhiều người**. Nguyên nhân là `conf=0.5` quá cao cho cảnh đông và nhiều người nhỏ/che khuất ở 640 px; YOLO đã lọc mất phần lớn người trước khi tracker kịp nối. Vì cảnh camera đứng yên, các tracker chủ yếu theo chuyển động là đủ, nên `botsort` (Re-ID) không giúp gì thêm mà còn đổi ID nhiều hơn khi hai người cắt nhau. Đây là ví dụ cho thấy mắt thường ưu tiên "ít hộp giả" còn metric lại phạt nặng việc bỏ sót.

**video_4 (trong nhà, camera tiến tới, phản chiếu kính).** Đây là cảnh chỉ đánh giá bằng mắt. `botsort` dùng ngoại hình nên track "dính" và đi theo người qua các lần người rẽ hướng, trong khi `bytetrack` hay mất track và gán lại ID mới. `conf=0.5` giúp loại các hộp giả sinh ra từ phản chiếu trên kính — một loại nhiễu khiến tracker theo chuyển động dễ bám nhầm. Cảnh trong nhà chiếu sáng tốt nên đặc trưng ngoại hình của Re-ID đủ tin cậy để phát huy tác dụng.

**video_2 (phố đêm, rất đông) và video_5 (trên xe bus, rung lắc).** Ở video_2, trời tối làm đặc trưng ngoại hình yếu nên tracker theo chuyển động (`ocsort`) là lựa chọn cân bằng giữa tốc độ và số lần bỏ sót; chênh lệch `conf`/`iou` nhỏ cho thấy trần chính nằm ở **detector** (người nhỏ, ngược sáng) chứ không phải bộ tham số tracker. Ở video_5, rung lắc làm hộp dao động mạnh; `botsort` với Re-ID giữ danh tính tốt hơn khi cả người và nền cùng dịch chuyển, còn `ocsort` sinh ID mới liên tục.

## 4. Nếu có thêm thời gian

Sẽ hạ `conf` cho video_1 (ví dụ 0.2–0.3) để đổi lấy độ thu hồi phát hiện vì `CLR_Re` mới 15.6%, rồi chấm lại và so HOTA/IDF1; đồng thời xem các frame bỏ sót để kiểm chứng nguyên nhân là người nhỏ chứ không phải lỗi tracker.
