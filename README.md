# Mô phỏng Mạng CAN Bus trên Vi điều khiển STM32

Đây là dự án cuối kỳ môn học, mô phỏng một mạng giao tiếp CAN bus cơ bản theo kiến trúc Master-Slave sử dụng hai vi điều khiển STM32F103. Dự án tập trung vào việc thiết lập, gửi, nhận và xử lý các gói tin CAN một cách đáng tin cậy.

---

## 🎯 Tổng quan dự án

Dự án xây dựng một hệ thống gồm 2 node giao tiếp với nhau qua bus CAN:

1.  **Slave Node (Nút Phụ):** Đóng vai trò là một khối thu thập dữ liệu (Data Acquisition). Nó đọc trạng thái từ các thiết bị ngoại vi như nút nhấn, cảm biến Hall, joystick (thông qua ADC), sau đó đóng gói dữ liệu và gửi lên bus CAN khi có sự kiện xảy ra.
2.  **Master Node (Nút Chủ):** Đóng vai trò là một khối giám sát trung tâm. Nó liên tục lắng nghe các gói tin trên bus CAN, phân loại chúng dựa trên ID, và gửi lại một gói tin phản hồi ("OK") để xác nhận đã nhận được dữ liệu.

Mục tiêu chính là chứng minh khả năng thiết lập và vận hành một mạng CAN ổn định, xử lý ngắt và truyền nhận dữ liệu hiệu quả giữa các node.

---

## 🛠️ Chức năng chi tiết

### Slave Node (ID: `0x222`)
- **Phát hiện sự kiện bằng ngắt (Interrupt-driven):** Sử dụng ngắt ngoài (EXTI) để phát hiện ngay lập tức các sự kiện từ:
  - 3 nút nhấn điều khiển (tương ứng 3 lệnh khác nhau).
  - 1 cảm biến Hall.
- **Đọc dữ liệu Analog:** Sử dụng ADC và DMA để liên tục đọc giá trị từ một joystick/biến trở mà không cần sự can thiệp của CPU.
- **Đóng gói dữ liệu:** Dữ liệu từ các cảm biến được đóng gói vào một gói tin CAN 8-byte theo một cấu trúc xác định.
- **Truyền dữ liệu:** Khi có sự kiện mới (nút nhấn, thay đổi trạng thái cảm biến, giá trị joystick vượt ngưỡng), node sẽ đặt cờ và gửi gói tin lên bus CAN trong vòng lặp chính.

### Master Node (Lắng nghe các ID)
- **Cấu hình bộ lọc (CAN Filter):** Thiết lập bộ lọc CAN để nhận các gói tin từ các slave node (`0x111`, `0x222`).
- **Nhận dữ liệu bằng ngắt:** Sử dụng ngắt `CAN_IT_RX_FIFO0_MSG_PENDING` để xử lý gói tin ngay khi nó đến.
- **Bộ đệm vòng (Circular Buffer):** Triển khai 2 bộ đệm vòng riêng biệt để lưu trữ các gói tin từ 2 slave node khác nhau, đảm bảo không bị mất dữ liệu khi có nhiều gói tin đến gần nhau.
- **Gửi phản hồi (Acknowledgment):** Ngay sau khi nhận thành công một gói tin từ bất kỳ slave nào, Master Node sẽ gửi lại một gói tin "OK" với ID `0x446` để xác nhận.

---

## ⚙️ Công nghệ sử dụng
- **Vi điều khiển:** STM32F103C8Tx
- **Nền tảng:** STM32CubeIDE, HAL Library
- **Giao thức:** CAN Bus
- **Ngôn ngữ:** C

---

## 📈 Sơ đồ hoạt động

`Slave Node (Phát hiện sự kiện từ Nút nhấn/Cảm biến)` -> `Đóng gói & Gửi gói tin CAN (ID: 0x222)` -> `CAN Bus` -> `Master Node (Nhận gói tin)` -> `Lưu vào bộ đệm` -> `Gửi gói tin "OK" (ID: 0x446)` -> `CAN Bus`

---

## 🚀 Hướng phát triển
Từ nền tảng giao tiếp CAN này, dự án có thể được mở rộng để thực hiện đầy đủ các chức năng của một hệ thống trên ô tô như trong báo cáo ban đầu:
- **Body Control Module:** Dùng dữ liệu từ các nút nhấn/joystick của Slave để điều khiển đèn xi-nhan, bật/tắt cửa sổ điện.
- **Engine Control Unit:** Dùng dữ liệu từ biến trở để điều khiển tốc độ động cơ DC.
- **Odometer/Dashboard:** Node Master có thể kết nối với một màn hình LCD/OLED để hiển thị thông tin nhận được (trạng thái cửa, tốc độ, tín hiệu xi-nhan,...).

---

## 📝 Cách sử dụng
1.  **Phần cứng:**
    - 2 board STM32F103 (Blue Pill hoặc tương tự).
    - 2 module CAN transceiver (ví dụ: TJA1050, SN65HVD230).
    - Nút nhấn, biến trở, cảm biến Hall và các linh kiện phụ trợ.
    - Kết nối các board với nhau qua bus CAN (chân CAN_H và CAN_L).
2.  **Phần mềm:**
    - Mở dự án bằng STM32CubeIDE.
    - Nạp code từ thư mục `master_node` vào board Master.
    - Nạp code từ thư mục `slave_node` vào board Slave.
    - Sử dụng một trình terminal (như Hercules) hoặc một bộ chuyển đổi USB-to-UART để theo dõi các gói tin "OK" được gửi từ Master.
