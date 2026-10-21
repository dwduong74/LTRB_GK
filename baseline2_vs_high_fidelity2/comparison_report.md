# Báo cáo so sánh bản đồ SLAM

- Map A: `/home/dwduong74/tb3_maps/slam_baseline2.yaml`
- Map B: `/home/dwduong74/tb3_maps/slam_high_fidelity2.yaml`
- Kích thước A/B: `238 x 198` / `243 x 259` ô
- Resolution A/B: `0.050` / `0.050` m/pixel
- Origin A: `[-1.160, -1.150]` m
- Origin B: `[-1.220, -4.160]` m
- Ô có thể so sánh: `47124`
- Ô occupied/free sai khác: `3255` (6.91%)
- Ô unknown hoặc ngoài phạm vi chung: `0`
- Vùng sai khác từ 4 ô trở lên: `48`

## Diễn giải ảnh difference.ppm

- Đỏ: một map xem là vật cản, map còn lại xem là free.
- Trắng nhạt: cả hai map đều free.
- Đen: cả hai map đều occupied.
- Xám: unknown hoặc vị trí nằm ngoài map còn lại.

## Lưu ý về tính công bằng của phép so sánh

Công cụ căn từng ô theo `origin` và `resolution` trong YAML. Nếu hai lượt chạy không có cùng vị trí bắt đầu và cùng tuyến lái, sai khác có thể đồng thời phản ánh độ bao phủ, lệch pose graph và cấu hình SLAM. Để đánh giá riêng hai profile, hãy chạy cùng world, cùng pose bắt đầu và cùng lộ trình.

## Vùng sai khác lớn nhất

| ID | Ô | Diện tích (m²) | Tâm X (m) | Tâm Y (m) |
| --- | ---: | ---: | ---: | ---: |
| 1 | 831 | 2.078 | 7.062 | 5.799 |
| 2 | 417 | 1.043 | 0.815 | 5.866 |
| 3 | 405 | 1.013 | 8.305 | 2.493 |
| 4 | 259 | 0.648 | 6.784 | -0.695 |
| 5 | 184 | 0.460 | 8.338 | 6.279 |
| 6 | 183 | 0.458 | 3.104 | 8.076 |
| 7 | 148 | 0.370 | 6.033 | 5.838 |
| 8 | 138 | 0.345 | 2.668 | 6.996 |
| 9 | 75 | 0.188 | 3.118 | -1.030 |
| 10 | 40 | 0.100 | -0.160 | 5.224 |
