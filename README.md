#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <SoftwareSerial.h>
#include <TinyGPS++.h>

// LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Pins
#define LIMIT_SWITCH 2
#define MQ3_PIN 3

// GPS & GSM
SoftwareSerial gpsSerial(4, 5);   // RX, TX
SoftwareSerial gsmSerial(7, 6);   // RX, TX

TinyGPSPlus gps;

// Mobile numbers
String number1 = "+91628*****";
String number2 = "+91798*****";

bool accidentSent = false;
bool alcoholSent = false;

void setup() {
  Serial.begin(9600);
  gpsSerial.begin(9600);
  gsmSerial.begin(9600);

  pinMode(LIMIT_SWITCH, INPUT_PULLUP);
  pinMode(MQ3_PIN, INPUT);

  lcd.begin();
  lcd.backlight();

  lcd.print("System Ready");
  delay(2000);
  lcd.clear();
}

void loop() {

  // ---- GPS Reading ----
  while (gpsSerial.available()) {
    gps.encode(gpsSerial.read());
  }

  float lat = gps.location.lat();
  float lng = gps.location.lng();

  // ---- Accident Detection ----
  if (digitalRead(LIMIT_SWITCH) == LOW && !accidentSent) {

    lcd.clear();
    lcd.print("Accident!");
    
    String msg = "Accident Alert!\nLocation:\n";
    msg += "http://maps.google.com/?q=";
    msg += String(lat, 6) + "," + String(lng, 6);

    sendSMS(number1, msg);
    sendSMS(number2, msg);

    accidentSent = true;
  }

  // ---- Alcohol Detection ----
  if (digitalRead(MQ3_PIN) == LOW && !alcoholSent) {

    lcd.clear();
    lcd.print("Alcohol Detected");

    String msg = "Alcohol Detected!\nLocation:\n";
    msg += "http://maps.google.com/?q=";
    msg += String(lat, 6) + "," + String(lng, 6);

    sendSMS(number1, msg);
    sendSMS(number2, msg);

    alcoholSent = true;
  }

  // ---- Normal Display ----
  lcd.setCursor(0, 0);
  lcd.print("Lat:");
  lcd.print(lat, 2);

  lcd.setCursor(0, 1);
  lcd.print("Lng:");
  lcd.print(lng, 2);

  delay(1000);
}

// 📩 SMS Function
void sendSMS(String number, String message) {
  gsmSerial.println("AT+CMGF=1");
  delay(1000);

  gsmSerial.print("AT+CMGS=\"");
  gsmSerial.print(number);
  gsmSerial.println("\"");
  delay(1000);

  gsmSerial.print(message);
  delay(1000);

  gsmSerial.write(26); // CTRL+Z
  delay(3000);
}
