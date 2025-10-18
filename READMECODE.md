#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <MPU6050.h>

//--- OLED Display Configuration ---
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

//--- Pin Configuration ---
#define LED_PIN 25
#define BUZZER_PIN 26

MPU6050 mpu;

//--- Vibration Threshold ---
const float threshold = 2.0;  // Vibration threshold (in g) to trigger alert

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22);
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);

  // Initialize OLED display
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("OLED display not found!");
    while (true);
  }

  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.println("Initializing MPU6050...");
  display.display();

  // Initialize MPU6050
  mpu.initialize();
  if (!mpu.testConnection()) {
    display.println("MPU6050 connection failed!");
    display.display();
    while (true);
  }

  delay(1000);
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("Sensor ready!");
  display.display();
}

void loop() {
  int16_t ax, ay, az;
  mpu.getAcceleration(&ax, &ay, &az);

  // Convert to g units (1g = 16384)
  float X = ax / 16384.0;
  float Y = ay / 16384.0;
  float Z = az / 16384.0;

  // Calculate total vibration magnitude
  float magnitude = sqrt(X * X + Y * Y + Z * Z);

  // Display on OLED
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("Vibration (g):");
  display.setCursor(0, 20);
  display.print(magnitude, 2);
  display.display();

  // Check if vibration exceeds threshold
  if (abs(magnitude - 1.00) > threshold) { // 1.00 ~ Earth's gravity
    digitalWrite(LED_PIN, HIGH);
    tone(BUZZER_PIN, 1000);
  } else {
    digitalWrite(LED_PIN, LOW);
    noTone(BUZZER_PIN);
  }

  delay(200);
}
