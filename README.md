#include <WiFi.h>
#include <Adafruit_GFX.h>
#include <Adafruit_ST7789.h>

#define TFT_CS 15
#define TFT_DC 2
#define TFT_RST 4
#define TFT_BL 32

Adafruit_ST7789 tft = Adafruit_ST7789(TFT_CS, TFT_DC, TFT_RST);

void startScreen() {
  pinMode(TFT_BL, OUTPUT);
  digitalWrite(TFT_BL, HIGH);
  pinMode(TFT_DC, OUTPUT);
  digitalWrite(TFT_DC, HIGH);
  tft.init(135, 240);
  tft.setRotation(1);
  tft.setTextWrap(false);
}

String encName(wifi_auth_mode_t t) {
  if (t == WIFI_AUTH_OPEN) return "OPEN";
  if (t == WIFI_AUTH_WEP) return "WEP";
  if (t == WIFI_AUTH_WPA_PSK) return "WPA";
  if (t == WIFI_AUTH_WPA2_PSK) return "WPA2";
  if (t == WIFI_AUTH_WPA_WPA2_PSK) return "WPA*";
  if (t == WIFI_AUTH_WPA3_PSK) return "WPA3";
  return "SEC";
}

void drawBars(int x, int y, int rssi) {
  int bars = 1;
  if (rssi > -90) bars = 2;
  if (rssi > -67) bars = 3;
  if (rssi > -55) bars = 4;

  for (int i = 0; i < 4; i++) {
    int h = 3 + i * 2;
    int bx = x + i * 4;
    int by = y + 8 - h;
    if (i < bars) tft.fillRect(bx, by, 3, h, ST77XX_GREEN);
    else tft.drawRect(bx, by, 3, h, ST77XX_WHITE);
  }
}

void setup() {
  startScreen();
  tft.fillScreen(ST77XX_BLACK);
  tft.setTextColor(ST77XX_CYAN);
  tft.setTextSize(2);
  tft.setCursor(6, 20);
  tft.print("ANALYZER");

  WiFi.mode(WIFI_STA);
  WiFi.disconnect(true);
  delay(200);
  startScreen();
}

void loop() {
  int n = WiFi.scanNetworks(false, true);

  startScreen();
  tft.fillScreen(ST77XX_BLACK);
  tft.setTextColor(ST77XX_CYAN);
  tft.setTextSize(2);
  tft.setCursor(4, 2);
  tft.print("WiFi");

  tft.setTextSize(1);
  tft.setTextColor(ST77XX_YELLOW);
  tft.setCursor(80, 8);
  tft.print(n);
  tft.print(" nets");

  int y = 24;
  int shown = min(n, 7);
  for (int i = 0; i < shown; i++) {
    String name = WiFi.SSID(i);
    if (name.length() == 0) name = "(hidden)";
    if (name.length() > 14) name = name.substring(0, 14);

    tft.setCursor(4, y);
    tft.setTextColor(ST77XX_WHITE);
    tft.print(name);

    tft.setCursor(100, y);
    tft.setTextColor(ST77XX_ORANGE);
    tft.print("ch");
    tft.print(WiFi.channel(i));

    drawBars(150, y, WiFi.RSSI(i));

    tft.setCursor(172, y);
    tft.setTextColor(ST77XX_GREEN);
    tft.print(WiFi.RSSI(i));

    tft.setCursor(206, y);
    tft.setTextColor(ST77XX_YELLOW);
    tft.print(encName(WiFi.encryptionType(i)));

    y += 14;
  }

  delay(2000);
}
