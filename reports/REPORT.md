# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Hồ Minh Hậu

Công cụ gán nhãn đã dùng: CVAT

## 1. Dữ liệu và cách chia tập

Tại sao tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng đệm ở giữa, thay vì chia ngẫu nhiên? Nếu chia ngẫu nhiên, số đo trên tập kiểm thử sẽ bị lệch theo hướng nào, và vì sao?

Vì camera đứng cố định, một chiếc xe sẽ xuất hiện trong nhiều khung hình liên tiếp vài giây. Việc chia theo trục thời gian với vùng đệm nhằm đảm bảo một chiếc xe ở tập train không xuất hiện lại trong tập test. Nếu chia ngẫu nhiên, mô hình sẽ được chấm điểm trên chính những chiếc xe nó đã học, dẫn đến hiện tượng rò rỉ dữ liệu (data leakage) và số đo AP50 sẽ cao hơn thực tế rất nhiều.

## 2. Mô hình khởi đầu lạnh (cold start)

Chép dòng vòng 0 từ `rounds_table.md`. Dựa vào `outputs/compare_round0.jpg`, cho biết mô hình khởi đầu lạnh không khớp nhãn tham chiếu ở những loại xe nào. Độ phủ (recall) theo kích thước xe cho thấy điều gì? Một trường hợp nào cần người rà lại nhãn tham chiếu trước khi kết luận mô hình sai?

| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |

Độ phủ xe nhỏ (R small = 0.182) thấp hơn nhiều so với xe lớn (R large = 0.561), cho thấy mô hình bỏ sót rất nhiều xe ở xa. Dựa vào ảnh compare, mô hình thường bỏ sót các xe quá mờ, ở rất xa chỉ thấy chấm đèn. Tuy nhiên, do nhãn tham chiếu test do một model khác tự tạo (chưa có người rà thủ công), nếu model của ta dự đoán đúng các xe nhỏ nhưng nhãn tham chiếu bị thiếu, ta cần phải rà lại test label trước khi kết luận là model của ta sai.

## 3. Chiến lược chọn mẫu

Giải thích bằng lời công thức `score = W_U·U + W_A·A + W_D·D` và vai trò của `MIN_GAP_S`. Dẫn ba frame trong `reports/SELECTION.md` và một frame khác để chứng minh cách bạn cân nhắc độ bất định, ảnh gần trùng và công gán nhãn. Điểm bất định có chứng minh ảnh đó sẽ cải thiện mô hình không? Vì sao?

Điểm số chọn mẫu bằng tổng của độ bất định của box (U), số lượng box bị nhập nhằng (A) và sự khác biệt về cảnh/thời gian so với các ảnh đã chọn (D). `MIN_GAP_S` đảm bảo 2 ảnh được chọn phải cách nhau một khoảng thời gian nhất định để tránh chọn ảnh trùng cảnh. Ví dụ, trong `SELECTION.md`, frame_0182.jpg, frame_0369.jpg được chọn vì có độ bất định cao (trên 0.93) và cách xa nhau, trong khi frame_0372.jpg dù điểm cao (0.910) lại bị loại bỏ vì quá gần frame_0369.jpg. Điểm bất định cao chỉ phản ánh độ phân vân của AI tại thời điểm đó, chứ không đảm bảo việc gán nhãn ảnh đó sẽ mang lại đa dạng kiến thức để cải thiện mô hình tổng thể.

## 4. Các vòng học chủ động (active learning)

Chép bảng từ `rounds_table.md`. Với mỗi vòng, trình bày: mức độ bạn đã sửa nhãn gợi ý, AP50 thay đổi bao nhiêu, nhóm xe nào tốt lên hoặc xấu đi theo số đo. Phân biệt quan sát độc lập, lỗi pre-label và kết quả mô hình.

| 1 | yolov8n fine-tune vong 1..1 | 12 | 321 | 0.480 | -0.291 | 1.000 | 0.144 | 0.252 | 0.000 | 0.128 | 0.488 |

Ở vòng 1, tôi đã thêm 160 box, xóa 8 box sai và chỉnh 2 box. Điểm AP50 sụt giảm mạnh so với vòng 0 (từ 0.771 xuống 0.480). Các nhóm xe nhỏ (R small = 0) và vừa (R medium = 0.128) đều giảm trầm trọng. Điều này cho thấy hiện tượng "catastrophic forgetting", model overfit vào 12 ảnh tối này, trở nên cực kỳ dè dặt (P=1.0 nhưng R=0.144). Khi nhìn vào ảnh `compare_round1.jpg`, model bỏ sót rất nhiều xe rõ rệt mà trước đó vòng 0 từng phát hiện được. Bản thân trong `BLIND_SCAN.md` và `REVIEW_LOG.csv` xác nhận tôi đã cố gắng gán rất nhiều xe nhỏ, nhưng model sau khi học lại với lượng dữ liệu quá bé lại đánh mất khả năng tổng quát hóa ban đầu.

## 5. Kết luận và giới hạn

Kết quả vòng này so với cold start ra sao? Vì sao bạn dừng hoặc tiếp tục? Đề xuất hai ca còn yếu hoặc bất định cho vòng sau, kèm chi phí rà nhãn và nguy cơ ảnh gần trùng. Tập kiểm thử chỉ 20 ảnh, có luật bỏ qua xe quá nhỏ và nhãn tham chiếu do mô hình tạo chưa được rà thủ công; các giới hạn đó ảnh hưởng thế nào đến kết luận? Nếu AP50 giảm, bạn sẽ kiểm tra điều gì trước khi train thêm?

Kết quả sụt giảm mạnh so với cold start do thiếu đa dạng dữ liệu. Tôi quyết định dừng vì với lượng ảnh gán hạn hẹp, việc lặp thêm vòng 2 có thể càng làm model quên đi khả năng tổng quát mà không khắc phục được tận gốc. Hai ca yếu cần xử lý tiếp là các xe nhỏ ở cực kỳ xa và xe bị đèn lóa che khuất một phần. Việc tập kiểm thử quá nhỏ (20 ảnh) và nhãn máy tự động sinh chưa chuẩn (thiếu sót nhiều xe) khiến cho điểm AP50 bị biến động ảo, không đánh giá đúng được thực trạng. Khi AP50 giảm mạnh, tôi cần xem xét lại mức độ chi tiết của nhãn (có gán quá nhiều xe bé li ti mà tập test không có hay không) và cân nhắc lấy mẫu lớn hơn trước khi quyết định train thêm.
