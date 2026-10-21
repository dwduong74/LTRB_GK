# Báo cáo so sánh hai cấu hình SLAM Toolbox

## Mục tiêu

Đánh giá khác biệt giữa hai cấu hình `slam_toolbox` cho TurtleBot3 Burger trong
Gazebo Harmonic. Mỗi lượt dùng lidar 2D `/scan`, odometry Gazebo và cùng độ
phân giải map 0,05 m/ô. Map được lưu bằng `nav2_map_server/map_saver_cli` và
so sánh bằng `tb3_slam_map_analysis/compare_maps`.

Hai profile được dùng:

| Profile | File cấu hình |
| --- | --- |
| Baseline | `tb3_slam_map_analysis/config/slam_baseline.yaml` |
| High-fidelity | `tb3_slam_map_analysis/config/slam_high_fidelity.yaml` |

`difference.ppm` quy ước: **đỏ** là ô một map xem là vật cản và map kia xem
là free; **trắng nhạt** là free giống nhau; **đen** là vật cản giống nhau;
**xám** là unknown hoặc nằm ngoài vùng map còn lại.

## Cấu hình và ý nghĩa tham số

Các tham số frame, sensor và solver được giữ giống nhau để tập trung vào mật
độ scan và liên kết pose graph.

| Nhóm | Baseline | High-fidelity | Ý nghĩa và hệ quả |
| --- | ---: | ---: | --- |
| `resolution` | 0,05 m | 0,05 m | Giữ cùng lưới map để các ô có thể so sánh trực tiếp. Giảm resolution sẽ chi tiết hơn nhưng tốn CPU/RAM hơn. |
| `min_laser_range`, `max_laser_range` | 0,12; 5,5 m | 0,12; 5,5 m | Khớp giới hạn lidar Burger; không đưa số đo ngoài khả năng sensor vào scan matching. |
| `minimum_time_interval` | 0,50 s | 0,15 s | High-fidelity dùng scan thường xuyên hơn, giúp mô tả lúc robot quay hoặc đi gần vật cản, đổi lại tăng tải xử lý. |
| `minimum_travel_distance` | 0,50 m | 0,15 m | High-fidelity thêm node pose graph dày hơn. Đường biên có thể mịn hơn, nhưng pose graph lớn hơn. |
| `minimum_travel_heading` | 0,50 rad | 0,15 rad | High-fidelity ghi nhận nhiều scan khi quay; hữu ích ở góc, cửa và ngã rẽ. |
| `map_update_interval` | 5,0 s | 1,0 s | High-fidelity cập nhật `/map` trên RViz nhanh hơn. Tham số này chủ yếu ảnh hưởng tần suất xuất bản/hiển thị, không tự làm tăng độ phân giải map. |
| `scan_buffer_size` | 10 | 30 | Giữ nhiều scan gần để ghép cục bộ ổn định hơn, nhưng tăng bộ nhớ và tính toán. |
| `link_scan_maximum_distance` | 1,5 m | 3,0 m | Mở rộng vùng tìm liên kết cục bộ; chịu sai số odometry tốt hơn nhưng có nhiều phương án ghép hơn. |
| `loop_search_maximum_distance` | 3,0 m | 5,0 m | Tìm đóng vòng trong vùng rộng hơn khi robot quay lại nơi cũ; có thể giảm drift ở world lớn. |
| `loop_match_minimum_chain_size` | 10 | 8 | High-fidelity xét loop closure sớm hơn. |
| Ngưỡng match fine/coarse | 0,10 / 0,35 / 0,45 | 0,08 / 0,30 / 0,40 | High-fidelity dễ chấp nhận liên kết hơn. Điều này hữu ích khi scan thưa hoặc world lớn, nhưng nếu môi trường lặp lại/hành lang giống nhau thì tăng nguy cơ liên kết sai. |

Các tham số `solver_plugin`, Ceres solver, scan matching, correlation search,
`do_loop_closing`, `transform_publish_period`, TF buffer và các penalty được
giữ nguyên. Vì vậy chúng không phải nguyên nhân trực tiếp tạo khác biệt giữa
hai profile trong thí nghiệm này.

## Lượt chạy 1: world tiêu chuẩn, phạm vi nhỏ

| Chỉ số | Baseline | High-fidelity |
| --- | ---: | ---: |
| Kích thước map | 112 × 103 ô | 112 × 103 ô |
| Origin map | [-0,959; -2,070] m | [-0,942; -2,090] m |
| Độ phân giải | 0,05 m/ô | 0,05 m/ô |

Kết quả từ [`baseline_vs_high_fidelity/comparison_report.md`](baseline_vs_high_fidelity/comparison_report.md):

| Chỉ số | Giá trị |
| --- | ---: |
| Ô có thể so sánh | 11.536 |
| Ô occupied/free sai khác | 127 |
| Tỷ lệ sai khác | **1,10%** |
| Vùng sai khác từ 4 ô trở lên | 2 |
| Vùng lớn nhất | 7 ô = 0,018 m² |

Hai map có cùng kích thước và origin chỉ lệch khoảng 1,7 cm theo X và 2 cm
theo Y. Các vùng đỏ lớn nhất chỉ 6–7 ô, phù hợp với sai khác ở mép tường,
góc hoặc việc lượng tử hóa scan vào lưới 5 cm. Trong world nhỏ, hai profile
cho kết quả nhất quán; không có bằng chứng high-fidelity làm map xấu đi.

### Ảnh map và vùng sai khác — lượt 1

**Baseline**

![Map baseline của lượt 1](figures/run1_baseline_map.png)

**High-fidelity**

![Map high-fidelity của lượt 1](figures/run1_high_fidelity_map.png)

**Vùng sai khác**

![Ô sai khác của lượt 1; màu đỏ là occupied/free không trùng nhau](figures/run1_difference.png)

## Lượt chạy 2: `tb3_complex_maze`, phạm vi lớn hơn

| Chỉ số | Baseline | High-fidelity |
| --- | ---: | ---: |
| Kích thước map | 238 × 198 ô | 243 × 259 ô |
| Origin map | [-1,160; -1,150] m | [-1,220; -4,160] m |
| Độ phân giải | 0,05 m/ô | 0,05 m/ô |

Kết quả từ [`baseline2_vs_high_fidelity2/comparison_report.md`](baseline2_vs_high_fidelity2/comparison_report.md):

| Chỉ số | Giá trị |
| --- | ---: |
| Ô có thể so sánh | 47.124 |
| Ô occupied/free sai khác | 3.255 |
| Tỷ lệ sai khác | **6,91%** |
| Vùng sai khác từ 4 ô trở lên | 48 |
| Vùng lớn nhất | 831 ô = 2,078 m² |

High-fidelity có biên map thấp hơn baseline **3,01 m theo Y** và cao hơn 61
hàng lưới. Điều này chứng tỏ lượt high-fidelity đã bao phủ thêm một vùng đáng
kể của maze, hoặc pose graph của hai lượt đã đặt map frame khác nhau. Vì map
high-fidelity chứa toàn bộ phạm vi của baseline, công cụ không đếm ô ngoài
phạm vi chung; tuy nhiên khác biệt về độ bao phủ vẫn ảnh hưởng mạnh tới số ô
đỏ trong phần diện tích chung.

Các cụm sai khác lớn trải dài theo hàng mét, không còn là vài pixel mép tường
như lượt 1. Nguyên nhân hợp lý nhất là tổng hợp của ba yếu tố:

1. Robot được lái thủ công qua hai tuyến hoặc quãng đường không hoàn toàn
   giống nhau; world lớn có nhiều nhánh nên chênh lệch này tăng nhanh.
2. High-fidelity thêm node scan dày hơn và thử loop closure rộng/sớm hơn. Khi
   tối ưu pose graph, các tường đã quét trước đó có thể dịch vài ô so với
   baseline, tạo dải đỏ dài.
3. High-fidelity dùng ngưỡng match thấp hơn. Trong maze có nhiều vách song
   song hoặc hình học lặp lại, nó có thể nhận thêm liên kết hữu ích nhưng cũng
   nhạy hơn với liên kết chưa tốt.

Kết quả **không đủ để kết luận high-fidelity kém hơn**, vì đầu vào hai lượt
chạy chưa được kiểm soát giống nhau. Nó cho thấy world lớn làm phép so sánh
nhạy với độ bao phủ, tuyến di chuyển và các quyết định loop closure.

### Ảnh map và vùng sai khác — lượt 2

**Baseline**

![Map baseline của lượt 2 trong maze](figures/run2_baseline_map.png)

**High-fidelity**

![Map high-fidelity của lượt 2 trong maze](figures/run2_high_fidelity_map.png)

**Vùng sai khác**

![Ô sai khác của lượt 2; màu đỏ là occupied/free không trùng nhau](figures/run2_difference.png)

## Log vận hành đã quan sát

- RViz báo `Trying to create a map of size ...`: bình thường khi SLAM mở rộng
  biên map hoặc tối ưu lại pose graph; world lớn làm thông báo xuất hiện nhiều
  hơn.
- Ceres báo `options.num_threads: 50 ... maximum ... 8`: không phải do map
  lớn. Solver tự giới hạn còn 8 luồng nên phiên SLAM vẫn hoạt động; chỉ tạo
  cảnh báo log.
- Cảnh báo GLSL của RViz là vấn đề render/OpenGL. Trong các lượt này RViz vẫn
  hiển thị map nên chưa có dấu hiệu dữ liệu SLAM bị hỏng.

Ghi chú chi tiết: [`complex_world_runtime_report.md`](../Lap_trinh_robot/bcr_bot_ws/src/tb3_slam_map_analysis/complex_world_runtime_report.md).

## Kết luận

1. Với world nhỏ và cùng phạm vi quét, hai profile rất gần nhau: sai khác
   1,10% và chỉ có hai cụm rất nhỏ.
2. Profile high-fidelity phù hợp khi cần bản đồ chi tiết hơn ở góc, cửa, lối
   hẹp và world lớn, với chi phí CPU/RAM cao hơn và thời gian tối ưu pose graph
   dài hơn.
3. Lượt maze cho sai khác 6,91%, chủ yếu phản ánh thực nghiệm chưa đồng nhất
   về độ bao phủ/tuyến lái cùng với độ nhạy cao hơn của loop closure; đây chưa
   phải bằng chứng định lượng rằng một profile chính xác hơn profile kia.

## Đề xuất lượt đánh giá tiếp theo

Để đo riêng tác động tham số, reset Gazebo trước mỗi profile, dùng cùng pose
spawn và cùng tuyến di chuyển. Tốt nhất ghi một rosbag gồm `/scan`, `/odom`,
`/tf` và `/clock`, sau đó phát lại đúng cùng dữ liệu cho cả hai profile. Khi
đó, so sánh `difference.ppm`, số vùng đỏ lớn và thời gian/CPU sẽ cho kết luận
tin cậy hơn về độ chính xác và chi phí tính toán.
