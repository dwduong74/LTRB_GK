# Ghi chú vận hành: SLAM trong `tb3_complex_maze`

**Ngày chạy:** 21-10-2026  
**Cấu hình đang chạy:** `slam_high_fidelity.yaml` với TurtleBot3 Burger,
Gazebo Harmonic và SLAM Toolbox.

## Quan sát từ log

1. RViz xuất các dòng như `Trying to create a map of size 117 x 126` và sau đó
   tăng dần tới các kích thước lớn hơn.
2. Ceres in cảnh báo `options.num_threads: 50 exceeds maximum ... 8`, sau đó
   tự giới hạn số luồng đang dùng xuống 8.
3. RViz từng báo GLSL: `active samplers with a different type refer to the same
   texture image unit`.
4. RViz cũng báo `Publisher already registered for provided node name` khi có
   nhiều publisher logger cùng tên.

## Diễn giải

- Các dòng tạo map là **bình thường** trong phiên SLAM online. Khi robot khám
  phá tới biên mới hoặc SLAM tối ưu pose graph, bản tin `/map` có thể đổi kích
  thước. RViz sẽ tạo lại texture để hiển thị map; world lớn làm thông báo này
  xuất hiện nhiều hơn.
- Cảnh báo Ceres **không phải do map lớn**. Solver yêu cầu 50 luồng nhưng bản
  Ceres của máy chỉ hỗ trợ tối đa 8 và tự giới hạn xuống 8. Cảnh báo này không
  trực tiếp làm sai map.
- Cảnh báo GLSL thuộc đường render của RViz/driver OpenGL. RViz vẫn nhận và
  hiển thị `/map`, nên chưa có bằng chứng cảnh báo này làm hỏng dữ liệu SLAM.
- Cảnh báo publisher logger trùng tên là cảnh báo ROS logging, không đồng nghĩa
  có hai node SLAM cùng publish `/map`. Mỗi lượt đo vẫn phải bảo đảm chỉ một
  `async_slam_toolbox_node` chạy trong `ROS_DOMAIN_ID=43`.

## Liên hệ với kết quả map

Map high-fidelity đã xuất hiện chồng map nội bộ (ghosting), biến dạng tường và
mảng occupancy không nhất quán. Các log trên không chứng minh LiDAR lỗi. Cần
ưu tiên kiểm tra local scan matching, loop closure, odometry, TF và timestamp
scan trước khi kết luận nguyên nhân từ cảm biến.
