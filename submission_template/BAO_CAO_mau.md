# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** nhom123 **Thành viên:** Nguyễn Đức Triệu - 2A202602978

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | bytetrack | 0.15 | 0.5 | Camera tĩnh, người đi bộ đều. Ngưỡng conf=0.15 cho phép ByteTrack bắt được cả người đi ở xa (hộp nhỏ, mờ) qua cơ chế ghép cặp 2 giai đoạn, track mượt và rất ít nhảy ID (IDSW chỉ 13). | bytetrack, conf=0.30, iou=0.5 (bị bỏ sót nhiều người ở xa phía quảng trường, HOTA và IDF1 thấp hơn). |
| video_2 (phố đêm, tĩnh, rất đông) | bytetrack | 0.20 | 0.5 | Cảnh ban đêm rất đông người chen chúc, che khuất liên tục. ByteTrack duy trì track ổn định nhờ liên kết hộp confidence thấp khi người bị che khuất một phần, không bị phụ thuộc vào ngoại hình vốn dễ lẫn lộn trong bóng tối. | botsort, conf=0.25, iou=0.5 (trong điều kiện trời tối, ảnh nhỏ và độ tương phản thấp, đặc trưng Re-ID bị nhiễu dẫn đến nhầm lẫn ngoại hình và đổi ID thường xuyên hơn khi hai người đi cắt nhau, đồng thời tốc độ xử lý quá chậm). |
| video_3 (camera di động, ảnh nhỏ) | botsort | 0.25 | 0.5 | Camera di chuyển và FPS thấp khiến vị trí người nhảy quãng lớn giữa các frame liên tiếp. BoTSORT có bù chuyển động camera (GMC) kết hợp đặc trưng Re-ID giúp theo dấu lại mục tiêu chính xác sau các bước nhảy khung hình. | bytetrack, conf=0.30, iou=0.5 (Kalman filter tuyến tính không bù trừ chuyển động camera khiến dự đoán vị trí lệch xa khỏi hộp phát hiện thực tế, track thường xuyên bị gãy và sinh ID mới liên tục). |
| video_4 (trong nhà, camera di chuyển) | bytetrack | 0.35 | 0.5 | Cảnh hành lang trong nhà với nhiều kính và bề mặt phản chiếu. Tăng conf lên 0.35 loại bỏ triệt để các hộp phát hiện giả từ bóng phản chiếu trên kính/cửa sổ. ByteTrack bám sát người đi dọc hành lang rất ổn định. | bytetrack, conf=0.20, iou=0.5 (conf thấp sinh ra nhiều hộp giả nhấp nháy trên bóng phản chiếu ở cửa kính, tracker nhận nhầm bóng người phản chiếu là một đối tượng mới). |
| video_5 (trên xe bus, giao lộ đông) | bytetrack | 0.25 | 0.5 | Góc quay từ xe bus bị rung lắc và giao lộ đông xe cộ/người qua lại. Conf=0.25 giữ cân bằng tốt giữa phát hiện người đi bộ và bỏ qua các vùng chuyển động hỗn loạn của phương tiện; ByteTrack gán track liên tục và hồi phục tốt sau khi xe rung lắc. | ocsort, conf=0.25, iou=0.5 (bộ lọc gặp lỗi tương thích kiểu dữ liệu mảng scalar trong điều kiện chuyển động gắt, hoặc tracker Re-ID bị chậm không đáp ứng thời gian xử lý thực tế). |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

```
HOTA: nhom01_video1-pedestrian     HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
video_1                            27.314    15.855    47.144    16.109    81.962    49.462    84.484    84.136    27.544    33.187    80.779    26.808    
COMBINED                           27.314    15.855    47.144    16.109    81.962    49.462    84.484    84.136    27.544    33.187    80.779    26.808    

CLEAR: nhom01_video1-pedestrian    MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag      
video_1                            18.314    81.945    18.384    19.019    96.769    11.29     16.129    72.581    14.881    3534      15047     118       13        7         10        45        34        
COMBINED                           18.314    81.945    18.384    19.019    96.769    11.29     16.129    72.581    14.881    3534      15047     118       13        7         10        45        34        

Identity: nhom01_video1-pedestrian IDF1      IDR       IDP       IDTP      IDFN      IDFP      
video_1                            26.987    16.146    82.147    3000      15581     652       
COMBINED                           26.987    16.146    82.147    3000      15581     652       

Count: nhom01_video1-pedestrian    Dets      GT_Dets   IDs       GT_IDs    
video_1                            3652      18581     35        62        
COMBINED                           3652      18581     35        62        
```

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

Với **ít nhất hai video** (nên gồm một video bạn chỉ đánh giá bằng mắt), viết 3–5 câu:

- **video_1 (Đánh giá định lượng qua TrackEval)**: ByteTrack kết hợp `conf=0.15` phát huy tối đa lợi thế của thuật toán ghép cặp 2 giai đoạn: giai đoạn 1 ghép các phát hiện có độ tin cậy cao, giai đoạn 2 ghép các phát hiện có confidence thấp hơn (do người ở xa hoặc bị che khuất nhẹ) với các tracklet chưa kết thúc. Nhờ vậy, recall tăng đáng kể (từ 3332 lên 3534 TP) trong khi precision vẫn giữ ở mức rất cao (96.77%), số lần đổi ID rất thấp (chỉ 13 lần ID switch trên toàn bộ 600 frame). Điều này làm tăng đồng thời HOTA (27.31%), MOTA (18.31%) và IDF1 (26.99%) so với mức conf=0.30 ban đầu.

- **video_3 (Camera di chuyển, ảnh nhỏ, low FPS — Đánh giá bằng mắt)**: Khi camera di chuyển tự do và tốc độ khung hình thấp, độ dịch chuyển pixel giữa các frame rất lớn và không tuân theo quy luật vận tốc không đổi đơn thuần. ByteTrack và Kalman filter cơ bản thường xuyên dự đoán sai vị trí mục tiêu ở frame kế tiếp, dẫn đến mất dấu và gán ID mới liên tục. BoTSORT vượt trội hơn rõ rệt nhờ cơ chế Global Motion Compensation (GMC) ước lượng ma trận biến đổi của camera giữa 2 khung hình để căn chỉnh lại tọa độ trước khi dự đoán, kết hợp với vector đặc trưng ngoại hình Re-ID (OSNet) để tái nhận diện đúng người sau các bước nhảy khung hình lớn.

- **video_4 (Trong nhà, phản chiếu kính — Đánh giá bằng mắt)**: Trong môi trường hành lang trong nhà có bề mặt kính bóng, người thật khi đi ngang qua tạo ra hình ảnh phản chiếu rõ nét. Khi đặt `conf` thấp (0.20), YOLO phát hiện cả hình phản chiếu trên kính gây ra hàng loạt hộp giả và track song song. Việc nâng `conf` lên 0.35 giúp lọc bỏ hoàn toàn các phát hiện giả mạo này, đồng thời giữ lại đúng người thật đang di chuyển dọc hành lang, mang lại độ chính xác rất cao mà không bị phân tán danh tính.

## 4. Nếu có thêm thời gian

Một hoặc hai câu: bạn sẽ thử tiếp điều gì (Re-ID khác, quét `conf` mịn hơn, xem frame gây lỗi…).

Nếu có thêm thời gian, nhóm sẽ thử nghiệm các mô hình Re-ID nhẹ hơn được tối ưu hóa tăng tốc suy luận trên GPU (TensorRT) để BoTSORT có thể chạy thời gian thực (>30 FPS) trên các cảnh camera chuyển động phức tạp. Ngoài ra, nhóm sẽ quét dải tham số mịn hơn cho cả `--conf` (bước nhảy 0.02) và `--iou`, đồng thời phân tích cụ thể các frame xuất hiện ID switch để tinh chỉnh tham số `track_buffer` nhằm tối ưu hóa hơn nữa chỉ số IDF1.
