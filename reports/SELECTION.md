# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, chọn năm frame bạn sẽ ưu tiên nếu chỉ có ngân sách rà năm ảnh. Ghi tên, điểm, thời điểm, thứ tự và lý do; tối thiểu một quyết định phải xét ảnh gần trùng hoặc trường hợp model không dự đoán được box:
Nếu chỉ được sửa 5 ảnh, tôi chọn frame_0182.jpg (hạng 1, điểm 0.959), frame_0369.jpg (hạng 2, điểm 0.932), frame_0380.jpg (hạng 3, điểm 0.917), frame_0326.jpg (hạng 4, điểm 0.915) và frame_0331.jpg (hạng 5, điểm 0.915). Đây là các ảnh có điểm bất định cao nhất đại diện cho các thời điểm khác nhau. Tôi bỏ qua frame_0372.jpg (hạng 6) vì nó cách frame_0369.jpg chỉ khoảng 1 giây, cảnh rất giống nhau nên không cần lãng phí công rà.

Ba frame thuộc lô 12 ảnh model chọn và bằng chứng trong CSV/ảnh contact sheet:
Trong 12 ảnh AI đã chọn, tôi nhận thấy frame_0182.jpg (điểm 0.959), frame_0369.jpg (điểm 0.932) và frame_0099.jpg (điểm 0.906) đều có điểm cao trên 0.9. Bằng chứng trong file CSV cho thấy cột `selected` là `True` và cột `n_ambiguous` của chúng rất cao (chứng tỏ AI đang phân vân với nhiều box).

Một frame có điểm cao nhưng không chọn hoặc một frame có điểm thấp vẫn nên xem, và lý do:
Frame_0372.jpg (hạng 6, điểm 0.910) có điểm cao hơn một số ảnh được chọn (ví dụ frame_0099.jpg hạng 8), nhưng AI bỏ qua vì nó quá gần thời gian với frame_0369.jpg. Hai ảnh gần như chung một cảnh, sửa cả hai sẽ tốn công vô ích mà model không học thêm được gì nhiều.

Điều phép chọn này chưa chứng minh về chất lượng mô hình:
Điểm cao chỉ phản ánh việc mô hình đang phân vân và độ bất định lớn. Nó chưa chứng minh rằng việc gán nhãn lại ảnh đó chắc chắn sẽ giúp mô hình cải thiện chất lượng nhận diện xe tốt hơn trên toàn tập dữ liệu.
