# HỆ THỐNG ĐIỀU KHIỂN ĐÈN GIAO THÔNG (Traffic Light Control System)

Đây là Bài tập lớn cho môn học **Vi xử lý - Vi điều khiển (CO3009)** thuộc Khoa Khoa học và Kỹ thuật Máy tính, Trường Đại học Bách khoa - ĐHQG TP.HCM.

Dự án mô phỏng một hệ thống điều khiển đèn giao thông tại một ngã tư đơn giản (2 làn đường) bằng vi điều khiển STM32. Hệ thống được thiết kế để áp dụng các kiến thức về lập trình vi điều khiển, xử lý ngắt, và quản lý phần cứng.

## ⚙️ Công nghệ & Kiến trúc

Hệ thống được xây dựng trên nền tảng phần cứng và các kỹ thuật phần mềm sau:

* **Vi điều khiển (MCU):** **STM32F103RB**.
* **Môi trường phát triển:** **STM32CubeIDE** (dựa trên cấu hình pin, clock và I2C trong báo cáo).
* **Thư viện:** **STM32 HAL (Hardware Abstraction Layer)**.
* **Giao tiếp:** **I2C** để giao tiếp với màn hình LCD.
* **Phần cứng:**
    * Màn hình **LCD 16x2** để hiển thị thông tin.
    * 2 bộ đèn LED giao thông (cho 2 làn đường X và Y).
    * 2 nút nhấn để điều khiển.

### Kiến trúc phần mềm

Dự án được thiết kế theo kiến trúc module hóa và điều khiển bằng máy trạng thái (FSM):

1.  **Máy trạng thái (Finite State Machine - FSM):**
    * Toàn bộ logic của hệ thống được quản lý bởi các máy trạng thái.
    * Một FSM chính được dùng để chuyển đổi giữa 3 chế độ (Switch FSM).
    * Mỗi chế độ (Automatic, Manual, Modify) có một FSM con để xử lý logic nội bộ.

2.  **Bộ lập lịch tác vụ (Scheduler):**
    * Thay vì dùng RTOS, dự án sử dụng một **bộ lập lịch hợp tác (Cooperative Scheduler)** đơn giản được xây dựng tùy chỉnh.
    * Scheduler này quản lý các tác vụ (tasks) bằng một **danh sách liên kết đôi (Doubly Linked List)**.
    * Nó chịu trách nhiệm quyết định tác vụ nào được thực thi (như đọc nút nhấn, cập nhật FSM, hiển thị LCD) dựa trên thời gian định trước.
    * Thiết kế này sử dụng một danh sách `freeList` để tái sử dụng các node tác vụ đã xóa, giảm chi phí cấp phát động (`malloc`).

3.  **Xử lý Nút nhấn:**
    * Module `button.c` triển khai kỹ thuật **chống dội (debouncing)** bằng cách đọc tín hiệu nhiều lần.
    * Hỗ trợ phát hiện tín hiệu **nhấn giữ (long press)** và **nhấn thả (rising edge)**.

4.  **Tổ chức mã nguồn:**
    * Mã nguồn được chia thành các module chức năng rõ ràng:
        * `global.c/h`: Quản lý các biến toàn cục, trạng thái hệ thống.
        * `button.c/h`: Xử lý logic nút nhấn.
        * `I2C_LCD.c/h` & `displayLCD.c/h`: Trừu tượng hóa việc giao tiếp và hiển thị LCD.
        * `trafficLight.c/h`: Điều khiển bật/tắt các đèn LED.
        * `scheduler.c/h`: Bộ lập lịch tác vụ.
        * `autoModeFSM.c/h`: FSM cho chế độ Tự động.
        * `manualModeFSM.c/h`: FSM cho chế độ Thủ công.
        * `modifyModeFSM.c/h`: FSM cho chế độ Tùy chỉnh.

## 🚀 Tính năng

Hệ thống có 3 chế độ hoạt động chính, có thể chuyển đổi bằng cách nhấn giữ nút:

### 1. Chế độ Tự động (Automatic Mode)
* Đây là chế độ hoạt động mặc định.
* Đèn giao thông tự động chuyển trạng thái (Đỏ, Xanh, Vàng) dựa trên thời gian được cài đặt trước.
* Màn hình LCD hiển thị thời gian đếm ngược của đèn trên cả 2 làn.
* Hệ thống có logic tự động kiểm tra và điều chỉnh thời gian (ví dụ: `Thời gian Đỏ = Thời gian Xanh + Thời gian Vàng`) khi khởi tạo.

### 2. Chế độ Thủ công (Manual Mode)
* Cho phép người dùng "can thiệp" vào hệ thống.
* Trong chế độ này, người dùng có thể nhấn nút để chuyển qua lại giữa các trạng thái (ví dụ: X-Đỏ/Y-Xanh và X-Xanh/Y-Đỏ).
* Nếu không có thao tác trong một khoảng thời gian (20 giây), hệ thống sẽ tự động quay về Chế độ Tự động.

### 3. Chế độ Tùy chỉnh (Modify Mode)
* Cho phép người dùng thay đổi thời gian của đèn.
* Người dùng có thể tùy chỉnh thời gian cho **đèn Đỏ** và **đèn Xanh**.
* Thời gian đèn Vàng sẽ được hệ thống tự động tính toán dựa trên 2 giá trị này.
* Giá trị được tăng/giảm bằng nút nhấn và lưu lại vào bộ đệm hệ thống.

## 🧑‍💻 Tác giả

* **Sinh viên:** Phạm Gia Lương - 2211960
* **Github:** [PhamGiaLuong/MCU_Assignment](https://github.com/PhamGiaLuong/CO3009-TrafficLightControlSystem)
