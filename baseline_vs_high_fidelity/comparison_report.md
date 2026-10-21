# Báo cáo so sánh bản đồ SLAM

- Map A: `/home/dwduong74/tb3_maps/slam_baseline.yaml`
- Map B: `/home/dwduong74/tb3_maps/slam_high_fidelity.yaml`
- Resolution A/B: `0.050` / `0.050` m/pixel
- Ô có thể so sánh: `11536`
- Ô occupied/free sai khác: `127` (1.10%)
- Ô unknown hoặc ngoài phạm vi chung: `0`
- Vùng sai khác từ 4 ô trở lên: `2`

## Diễn giải ảnh difference.ppm

- Đỏ: một map xem là vật cản, map còn lại xem là free.
- Trắng nhạt: cả hai map đều free.
- Đen: cả hai map đều occupied.
- Xám: unknown hoặc vị trí nằm ngoài map còn lại.

## Vùng sai khác lớn nhất

| ID | Ô | Diện tích (m²) | Tâm X (m) | Tâm Y (m) |
| --- | ---: | ---: | ---: | ---: |
| 1 | 7 | 0.018 | 3.566 | -1.445 |
| 2 | 6 | 0.015 | 0.441 | 2.505 |
