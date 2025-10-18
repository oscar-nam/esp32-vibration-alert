# Hệ thống cảnh báo rung ESP32

Dự án này sử dụng vi điều khiển ESP32 với cảm biến gia tốc MPU6050 và màn hình OLED để phát hiện rung động và kích hoạt cảnh báo bằng đèn LED và còi.

## Tính năng
- Giám sát rung động theo thời gian thực
- Hiển thị dữ liệu trên màn hình OLED
- Cảnh báo bằng đèn LED và còi khi vượt ngưỡng

## Phần cứng
- ESP32
- Cảm biến MPU6050
- Màn hình OLED (SSD1306)
- Đèn LED (kết nối chân 25)
- Còi (kết nối chân 26)

## Nguyên lý hoạt động
- MPU6050 đo gia tốc theo 3 trục.
- Tính toán độ lớn rung động.
- Nếu vượt ngưỡng (2g), đèn LED sáng và còi kêu.

## Kết nối
- MPU6050: SDA đến GPIO21, SCL đến GPIO22
- OLED: địa chỉ I2C 0x3C
- LED: GPIO25
- Còi: GPIO26

## Cài đặt
1. Cài đặt các thư viện:
   - `Adafruit_GFX`
   - `Adafruit_SSD1306`
   - `MPU6050`
2. Nạp mã vào ESP32
3. Mở Serial Monitor ở tốc độ 115200

## Giấy phép
MIT
