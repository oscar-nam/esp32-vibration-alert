#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <MPU6050.h>

//--- Cấu hình màn hình OLED ---
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

//--- Cấu hình chân ---
#define LED_PIN 25
#define BUZZER_PIN 26

MPU6050 mpu;

//--- Biến ngưỡng rung ---
const float threshold = 2.0;  // giá trị rung (theo g) để bật cảnh báo

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22);
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);

  // Khởi động màn hình
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("Không tìm thấy màn hình OLED!");
    while (true);
  }

  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.println("Khoi dong MPU6050...");
  display.display();

  // Khởi động MPU6050
  mpu.initialize();
  if (!mpu.testConnection()) {
    display.println("MPU6050 loi ket noi!");
    display.display();
    while (true);
  }

  delay(1000);
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("Cam bien san sang!");
  display.display();
}

void loop() {
  int16_t ax, ay, az;
  mpu.getAcceleration(&ax, &ay, &az);

  // Chuyển đổi sang đơn vị g (1g = 16384)
  float X = ax / 16384.0;
  float Y = ay / 16384.0;
  float Z = az / 16384.0;

  // Tính độ rung tổng hợp
  float magnitude = sqrt(X * X + Y * Y + Z * Z);

  // Hiển thị lên màn hình
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("Do rung (g):");
  display.setCursor(0, 20);
  display.print(magnitude, 2);
  display.display();

  // Kiểm tra ngưỡng cảnh báo
  if (abs(magnitude - 1.00) > threshold) { // 1.00 ~ trọng lực trái đất
    digitalWrite(LED_PIN, HIGH);
    tone(BUZZER_PIN, 1000);
  } else {
    digitalWrite(LED_PIN, LOW);
    noTone(BUZZER_PIN);
  }

  delay(200);
}
