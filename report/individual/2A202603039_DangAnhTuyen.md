# Báo cáo thực hành Day 3 — Dang Anh Tuyen

- MSSV theo tên thư mục dự án: **2A202603039**.
- Ngày thực hành: **11/11/2026**; sử dụng CVAT local tại `http://localhost:8080`.
- Công cụ đã dùng: CVAT; SAM 2.1 Small để gợi ý mask bằng box và điểm; YOLO26n để phát hiện vật; SegFormer B0 Cityscapes cho semantic; Python để tạo/chỉnh mask, đóng ZIP và chạy bộ tự chấm của repo. Có trợ lý AI hỗ trợ viết pipeline, đặt prompt theo ảnh, kiểm tra và xuất kết quả.

Báo cáo tổng hợp theo các ZIP và kết quả kiểm tra hiện có. Phần quyết định tự vẽ ở mục 2 cần người học bổ sung theo thao tác thực tế.

## 1. Bài đã nộp

Các tệp dưới đây đã có trong `submissions/` và vượt qua kiểm tra cấu trúc bằng `scripts/inspect_submissions.py`. Số ảnh là số ảnh có dữ liệu trong ZIP; cần đối chiếu lại bản lưu trong CVAT trước khi chốt bài. Cột điểm là mức tối đa của task.

| Task | File ZIP đúng tên | Ảnh có dữ liệu trong ZIP | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | **14 / 14 ảnh có dữ liệu** | **100** |

Easy, CP3, CP4 và CP6 dùng **Segmentation mask 1.1**. Medium, Hard, CP1, CP2 và CP5 dùng **COCO 1.0**. Các mask được tạo hoặc sửa bằng Python ở lượt sau đã được ghi vào ZIP; annotation tương ứng đang lưu trong CVAT chưa được cập nhật bằng lượt xuất này.

## 2. Một quyết định trước khi dùng gợi ý

**Chưa có thông tin xác nhận về object Medium đầu tiên được người học tự vẽ trước khi xem gợi ý. Cần điền theo việc đã làm thực tế; các đề xuất do pipeline tạo sau đó không thay thế phần này.**

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: **[BỔ SUNG tên ảnh, vị trí và object]**.
- Class và quy tắc dùng để chọn biên: **[BỔ SUNG quyết định thực tế]**.
- Nếu dùng gợi ý sau đó: **[BỔ SUNG vùng đã giữ/sửa và lý do; ghi “không dùng” nếu đúng thực tế]**.

## 3. Một lỗi tôi tìm thấy và sửa

- **Task/ảnh/vùng:** `cp1_holes`, ảnh `000000144300.jpg`, chiếc mô tô đỏ–trắng lớn ở tiền cảnh.
- **Loại lỗi:** gộp–tách instance và giữ các vùng kín bên trong vật.
- **Bằng chứng quan sát trong lượt kiểm tra có trợ lý hỗ trợ:** bản đề xuất ban đầu có hai mask `motorcycle`, một cho phần trước và một cho phần sau của cùng chiếc xe. ZIP CP1 trong `submissions/` trước lượt bổ sung cũng chưa có annotation.
- **Quy tắc và hành động sửa:** xác định đây là một chiếc mô tô; đặt box bao toàn bộ xe cùng các điểm dương ở thân và hai bánh để SAM tạo lại mask. Giữ một instance `motorcycle`, xử lý các vùng kín bên trong mask xe theo yêu cầu CP1. Kết quả ZIP có 6 mask: 1 motorcycle, 2 person, 1 car và 2 truck. Nhãn của xe van phía sau còn cần kiểm lại như mục 4.
- **Trạng thái lưu/xuất:** đã ghi kết quả vào `submissions/cp1_holes.zip` và kiểm tra RLE, kích thước ảnh, số instance mô tô. Lượt sửa này thực hiện bằng Python; chưa xác nhận bản annotation CP1 mới đã được nhập và Save trong CVAT.

Đã dùng ground truth được cung cấp để tự đánh giá ba tier. Sau các lượt sửa có hỗ trợ mô hình, tổng điểm tự kiểm lần lượt là **43.1 → 50.9 → 54.5 / 82**. Điểm mới nhất nằm trong [SCORECARD.md](../../reports/tiers/SCORECARD.md):

| Task | Metric của bộ tự chấm | Giá trị | Điểm tự kiểm |
| --- | --- | ---: | ---: |
| easy_semantic | mIoU | 0.807 | 18.1 / 20 |
| medium_instance | Mean matched IoU × recall | 0.677 | 19.7 / 32 |
| hard_panoptic | PQ | 0.450 | 16.7 / 30 |
| **Tổng ba tier** | | | **54.5 / 82** |

Lượt sửa Hard gần nhất tăng từ **13.1 lên 16.7/30**, với việc xử lý mask trong vùng cắt gần, kết hợp dự đoán semantic và kiểm các đề xuất rất nhỏ. Các bản thử mới của Easy và Medium có điểm thấp hơn nên giữ ZIP của lượt trước.

Các kết quả trên là **tự kiểm sau khi có reference và dùng reference để đánh giá/chọn bản sửa**, không đại diện cho chất lượng bản làm độc lập trước khi xem đáp án. Bộ sinh mask đọc ảnh, nhãn và dự đoán trước đó; không sao chép mask GT vào ZIP, không sửa scorer hoặc GT. Chưa có reference riêng của sáu checkpoint nên chưa tự tính điểm các trạm; **54.5/82 chưa phải điểm cuối trên 100**. GT chỉ giữ trên máy.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| CP1, `000000144300.jpg`, phần trước và sau của mô tô đỏ–trắng | Hai vùng đề xuất là hai xe, hoặc hai phần của cùng một xe | Quan sát thấy thân và yên liên tục nối hai bánh; model đã chia vật thành hai mask | Đã giữ một instance `motorcycle`; kiểm lại các vùng kín theo quy tắc CP1 |
| CP1, `000000144300.jpg`, xe van xanh phía sau mô tô | Gán `truck` hoặc `car` cho xe van | Xe bị che một phần; schema không có class `van`, nên hình dáng nhìn thấy chưa đủ để khẳng định mapping | Bản hiện tại gán `truck`; cần coach xác nhận cách ánh xạ loại xe này |
| Hard, `000000350023.jpg`, vùng sát lề đường hai bên | Vỉa hè cho người đi bộ, hoặc làn xe/đỗ xe cùng màu mặt đường | Cần đối chiếu bó vỉa, vị trí hàng cây và chức năng bề mặt; màu xám đơn thuần không quyết định class | Đã thử prompt riêng cho sidewalk và kiểm mask chồng lấn. Bộ chấm vẫn báo sidewalk chưa có segment khớp đạt ngưỡng; cần kiểm lại ranh giới trước khi chốt |

Giới hạn còn lại: Hard vẫn bỏ sót hoặc phân đoạn chưa đạt với người, xe đạp và vỉa hè. Medium còn nhiều mask không khớp reference; việc có đủ ảnh và ZIP đọc được không đồng nghĩa mọi vật đã gán đúng. Cần rà soát các đề xuất và ghi nhận thao tác tự làm thực tế trước khi nộp.
