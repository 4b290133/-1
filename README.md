#include <Servo.h>

// 腳位定義
const int DRYER_RELAY_PIN = 7; // 接繼電器控制廚餘乾燥機
const int SERVO_PIN = 9;       // 一般垃圾蓋馬達
Servo lidServo;

// 變數用於儲存雷達數據
int lastDist = 0;
int waveCount = 0;
unsigned long gestureStartTime = 0;

void setup() {
  Serial.begin(115200);
  pinMode(DRYER_RELAY_PIN, OUTPUT);
  digitalWrite(DRYER_RELAY_PIN, LOW); // 預設關閉乾燥機
  lidServo.attach(SERVO_PIN);
  lidServo.write(0);
}

void loop() {
  int currentDist = getRadarDistance(); // 取得當前距離 (需串接 UART 解析)

  // --- 手勢偵測：空中畫圈 (Circle Gesture) ---
  // 邏輯：在短時間內，距離值發生明顯的規律起伏
  if (abs(currentDist - lastDist) > 5 && currentDist < 40) {
    if (gestureStartTime == 0) gestureStartTime = millis();
    waveCount++;
    
    // 如果 2 秒內偵測到足夠次數的距離起伏，視為畫圈
    if (millis() - gestureStartTime < 2000 && waveCount > 15) {
      activateDryer();
      waveCount = 0;
      gestureStartTime = 0;
    }
  }

  // --- 手勢偵測：快速揮手 (Open Lid) ---
  if (currentDist < 20 && waveCount < 5) {
    openLid();
  }

  // 超時重置計數器
  if (millis() - gestureStartTime > 2000) {
    waveCount = 0;
    gestureStartTime = 0;
  }

  lastDist = currentDist;
  delay(100); // 採樣頻率
}

void activateDryer() {
  Serial.println("偵測到畫圈：啟動廚餘乾燥處理！");
  digitalWrite(DRYER_RELAY_PIN, HIGH);
  delay(2000); // 示意啟動，實際可依需求設定運行時間
}

void openLid() {
  lidServo.write(90);
  delay(3000);
  lidServo.write(0);
}

// 實際需解析 K60168-P 的數據包
int getRadarDistance() {
  // 此處應放入 UART 讀取協定
  return 20; 
}
