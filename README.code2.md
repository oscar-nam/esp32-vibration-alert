#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <MPU6050.h>

//--- OLED Display Configuration ---
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
#define OLED_ADDRESS 0x3C

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

//--- Pin Configuration ---
#define LED_PIN 25
#define BUZZER_PIN 26
#define SDA_PIN 21
#define SCL_PIN 22

MPU6050 mpu;

//--- Vibration Threshold ---
const float threshold = 2.0;  // Vibration threshold (in g) to trigger alert

void setup() {
  Serial.begin(115200);
  delay(100);

  Serial.println("ESP32 Vibration Alert System");
  Serial.println("Initializing...");

  // Initialize I2C with explicit pins
  Wire.begin(SDA_PIN, SCL_PIN);
  delay(100);

  // Configure output pins
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);
  digitalWrite(BUZZER_PIN, LOW);

  // Initialize OLED display
  Serial.println("Initializing OLED display...");
  if (!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDRESS)) {
    Serial.println("ERROR: OLED display not found at address 0x3C!");
    Serial.println("Check wiring and I2C address.");
    while (true) {
      delay(1000);
    }
  }

  Serial.println("OLED display initialized successfully");

  // Clear display and show initial message
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.println("ESP32 Vibration");
  display.println("Alert System");
  display.println("");
  display.println("Initializing");
  display.println("MPU6050...");
  display.display();
  delay(1000);

  // Initialize MPU6050
  Serial.println("Initializing MPU6050...");
  mpu.initialize();

  if (!mpu.testConnection()) {
    Serial.println("ERROR: MPU6050 connection failed!");
    display.clearDisplay();
    display.setCursor(0, 0);
    display.println("ERROR:");
    display.println("MPU6050 not found!");
    display.println("");
    display.println("Check wiring:");
    display.println("SDA->GPIO21");
    display.println("SCL->GPIO22");
    display.display();
    while (true) {
      delay(1000);
    }
  }

  Serial.println("MPU6050 initialized successfully");

  // Show ready message
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("System Ready!");
  display.println("");
  display.println("Monitoring");
  display.println("vibrations...");
  display.display();
  delay(2000);

  Serial.println("System ready - monitoring vibrations");
}

void loop() {
  int16_t ax, ay, az;
  mpu.getAcceleration(&ax, &ay, &az);

  // Convert to g units (MPU6050 default sensitivity: 1g = 16384 LSB)
  float X = ax / 16384.0;
  float Y = ay / 16384.0;
  float Z = az / 16384.0;

  // Calculate total vibration magnitude
  float magnitude = sqrt(X * X + Y * Y + Z * Z);

  // Calculate deviation from 1g (Earth's gravity baseline)
  float deviation = abs(magnitude - 1.00);

  // Display on OLED
  display.clearDisplay();
  display.setTextSize(1);
  display.setCursor(0, 0);
  display.println("Vibration Monitor");
  display.println("----------------");

  display.print("Mag: ");
  display.print(magnitude, 2);
  display.println(" g");

  display.print("Dev: ");
  display.print(deviation, 2);
  display.println(" g");

  display.println("");

  // Check if vibration exceeds threshold
  if (deviation > threshold) {
    display.println("*** ALERT! ***");
    digitalWrite(LED_PIN, HIGH);
    tone(BUZZER_PIN, 1000);

    Serial.print("ALERT! Vibration detected: ");
    Serial.print(magnitude, 2);
    Serial.print(" g (deviation: ");
    Serial.print(deviation, 2);
    Serial.println(" g)");
  } else {
    display.println("Status: Normal");
    digitalWrite(LED_PIN, LOW);
    noTone(BUZZER_PIN);
  }

  display.display();

  // Also print to serial for debugging
  Serial.print("X: ");
  Serial.print(X, 2);
  Serial.print(" Y: ");
  Serial.print(Y, 2);
  Serial.print(" Z: ");
  Serial.print(Z, 2);
  Serial.print(" Mag: ");
  Serial.print(magnitude, 2);
  Serial.print(" Dev: ");
  Serial.println(deviation, 2);

  delay(200);
}
