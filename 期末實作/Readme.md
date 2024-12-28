<h1>HC-SR04超音波感測器與RGB LED燈 + TM1637 4位七段顯示器與MP3模組，使用按鈕控制實現倒車雷達之功能</h1>


## 準備材料 : 
>1. Arduino Nano板(CH340驅動程式.USB:MiniUSB) 
>2. MiniUSB 連接線 X 1 
>3. HC-SR04超音波感測器 Trig->pin2 , Echo->pin3
>4. RGB LED燈 Red->pin9 , Green->pin10 , blue->pin11
>5. TM1637 4位七段顯示器 CLK->pin4 , DIO->pin5
>6. button 按鈕-> pin6
>7. DFPlayer mini  TX->8pin , RX->7pin
>8. 麵包板 X 1===

## 電路圖
>![](https://github.com/sujamie/Arduino/blob/main/%E6%9C%9F%E6%9C%AB%E5%AF%A6%E4%BD%9C/%E9%9B%BB%E8%B7%AF%E5%9C%96.jpg?raw=true)

## 引入函式庫
>![](https://github.com/sujamie/Arduino/blob/main/%E6%9C%9F%E6%9C%AB%E5%AF%A6%E4%BD%9C/TM1367.png?raw=true)

##流程圖
>![](https://github.com/sujamie/Arduino/blob/main/%E6%9C%9F%E6%9C%AB%E5%AF%A6%E4%BD%9C/%E6%B5%81%E7%A8%8B%E5%9C%96.png?raw=true)

##實作影片

<a href= "https://youtube.com/shorts/ZH6xR6i-68Y?feature=share">
<img src= "https://i.ytimg.com/vi/ZH6xR6i-68Y/oar2.jpg?sqp=-oaymwEoCJUDENAFSFqQAgHyq4qpAxcIARUAAIhC2AEB4gEKCBgQAhgGOAFAAQ==&rs=AOn4CLA1kXOIQcs2IWTwRVDh0-mNiFULkA/0.jpg"></a>

## 程式碼

``` arduino
#include "TM1637.h"        // 引入 TM1637 顯示器的程式庫
#include <DFMiniMp3.h>     // 引入 DFMiniMp3 播放器程式庫
#include <SoftwareSerial.h>  // 引入軟體序列通訊 (SoftwareSerial) 程式庫，用於與 MP3 模組溝通

// 定義超音波感測器和顯示器的引腳
const int trigPin = 2;   // 超音波感測器 Trig (觸發) 引腳
const int echoPin = 3;   // 超音波感測器 Echo (回波) 引腳
const int CLK = 4;       // TM1637 顯示器時鐘引腳
const int DIO = 5;       // TM1637 顯示器資料引腳
const int buttonPin = 6; // 啟動按鈕引腳
SoftwareSerial mySerial(8, 7);  // 定義軟體序列埠 (TX, RX)，用於 MP3 模組通訊
const int redPin = 9;    // 紅色 LED 引腳
const int greenPin = 10; // 綠色 LED 引腳
const int bluePin = 11;  // 藍色 LED 引腳

bool isRunning = false;  // 設定初始狀態，預設為不啟動
const int sw = 3000;     // 定義播放音效的長度 (3 秒)

// 宣告 Mp3Notify 類別，供 MP3 播放器回呼使用
class Mp3Notify; 
typedef DFMiniMp3<SoftwareSerial, Mp3Notify> DfMp3;  // 定義 MP3 播放器物件類型
DfMp3 dfmp3(mySerial);   // 建立 MP3 播放器物件，使用 mySerial 與模組溝通

// 定義 Mp3Notify 類別，用於處理 MP3 播放器的回呼事件
class Mp3Notify {
public:
  // 列印來源 (SD, USB, Flash) 和行為 (例如插入、移除)
  static void PrintlnSourceAction(DfMp3_PlaySources source, const char *action) {
    if (source & DfMp3_PlaySources_Sd) {   // 若來源為 SD 卡
      Serial.print("SD Card, ");
    }
    if (source & DfMp3_PlaySources_Usb) {  // 若來源為 USB
      Serial.print("USB Disk, ");
    }
    if (source & DfMp3_PlaySources_Flash) { // 若來源為 Flash 記憶體
      Serial.print("Flash, ");
    }
    Serial.println(action);  // 列印動作，例如 online, inserted 等
  }
  
  // 處理錯誤訊息的回呼函式
  static void OnError(DfMp3 &mp3, uint16_t errorCode) {
    Serial.print("Com Error ");  // 印出錯誤代碼
    Serial.println(errorCode);
  }
  
  // 播放完成時的回呼函式
  static void OnPlayFinished(DfMp3 &mp3, DfMp3_PlaySources source, uint16_t track) {
    Serial.print("Play finished for #");  // 播放完成後印出軌道號碼
    Serial.println(track);
  }

  // 音源上線時觸發
  static void OnPlaySourceOnline(DfMp3 &mp3, DfMp3_PlaySources source) {
    PrintlnSourceAction(source, "online");
  }

  // 音源插入時觸發
  static void OnPlaySourceInserted(DfMp3 &mp3, DfMp3_PlaySources source) {
    PrintlnSourceAction(source, "inserted");
  }

  // 音源移除時觸發
  static void OnPlaySourceRemoved(DfMp3 &mp3, DfMp3_PlaySources source) {
    PrintlnSourceAction(source, "removed");
  }
};

// 初始化 TM1637 顯示器物件
TM1637 tm1637(CLK, DIO);

void setup() {
  Serial.begin(9600);  // 設定序列埠通訊速率為 9600
  pinMode(trigPin, OUTPUT);  // 設定 Trig 引腳為輸出
  pinMode(echoPin, INPUT);   // 設定 Echo 引腳為輸入
  pinMode(redPin, OUTPUT);   // 設定紅燈引腳為輸出
  pinMode(greenPin, OUTPUT); // 設定綠燈引腳為輸出
  pinMode(bluePin, OUTPUT);  // 設定藍燈引腳為輸出
  pinMode(buttonPin, INPUT_PULLUP);  // 設定按鈕引腳為上拉輸入 (預設高電位)

  Serial.println("initializing...");
  dfmp3.begin();  // 初始化 MP3 播放器
  dfmp3.reset();  // 重置 MP3 播放器
  delay(1000);    // 延遲 1 秒，等待模組穩定
  dfmp3.setVolume(28);  // 設定音量為 28 (範圍 0-30)
  Serial.println("Ready to start. Waiting for button press...");

  tm1637.init();  // 初始化 TM1637 顯示器
  tm1637.set(BRIGHT_TYPICAL);  // 設定顯示器亮度

  updateLEDs(9999);  // 啟動時關閉所有 LED
  updateDisplay(9999);  // 顯示器顯示 ---- 表示未啟動

  // 等待按鈕被按下 (啟動)
  while (digitalRead(buttonPin) == HIGH) {
    delay(100);  // 每 100 毫秒檢查一次
  }
  isRunning = true;  // 啟動程式
  Serial.println("Button Pressed. Program Started");
}

// 觸發超音波感測器
void triggerUltrasonic() {
  digitalWrite(trigPin, LOW);  // Trig 引腳先拉低
  delayMicroseconds(2);        // 等待 2 微秒
  digitalWrite(trigPin, HIGH); // 發送 10 微秒的脈衝
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);  // 拉低 Trig 引腳，結束觸發
}

// 讀取超音波感測器的距離
long readDistance() {
  long duration = pulseIn(echoPin, HIGH, 30000);  // 讀取 Echo 引腳的高電位時間，最多等待 30ms
  return (duration == 0) ? 9999 : duration * 0.034 / 2;  // 計算距離 (單位 cm)，若超時回傳 9999
}

void loop() {
  static bool lastButtonState = HIGH;
  bool currentButtonState = digitalRead(buttonPin);  // 讀取按鈕狀態

  // 檢查按鈕是否從未按下變為按下
  if (currentButtonState == LOW && lastButtonState == HIGH) {
    delay(150);  // 防止按鈕彈跳
    while (digitalRead(buttonPin) == LOW) {
      delay(10);
    }
    isRunning = !isRunning;  // 切換程式狀態
    Serial.println(isRunning ? "Program Started" : "Program Stopped");
    if (!isRunning) {
      updateLEDs(9999);
      updateDisplay(9999);
    }
    delay(500);
  }
  lastButtonState = currentButtonState;

  if (isRunning) {
    triggerUltrasonic();   // 觸發超音波
    long distance = readDistance();
    Serial.print("Distance: ");
    Serial.println(distance);
    updateLEDs(distance);  // 更新 LED 狀態
    updateDisplay(distance);  // 更新顯示器
  }
}
// 更新 LED 顏色根據距離
void updateLEDs(long distance) {
  digitalWrite(redPin, LOW);   // 預設關閉紅燈
  digitalWrite(greenPin, LOW); // 預設關閉綠燈
  digitalWrite(bluePin, LOW);  // 預設關閉藍燈

  if (distance > 70) {  // 若距離大於 70cm
    digitalWrite(greenPin, HIGH);  // 亮綠燈
    dfmp3.playMp3FolderTrack(1);   // 播放 MP3 資料夾中第 1 首音樂
  } else if (distance >= 25) {  // 距離介於 25cm 到 70cm
    digitalWrite(redPin, HIGH);   // 亮紅燈
    digitalWrite(greenPin, HIGH); // 同時亮綠燈
    digitalWrite(bluePin, HIGH);  // 同時亮藍燈 (白光)
    dfmp3.playMp3FolderTrack(2);  // 播放第 2 首音樂
  } else if (distance > 10) {  // 距離介於 10cm 到 25cm
    digitalWrite(bluePin, HIGH);  // 亮藍燈
    dfmp3.playMp3FolderTrack(3);  // 播放第 3 首音樂
  } else {  // 距離小於 10cm
    digitalWrite(redPin, HIGH);   // 亮紅燈
    dfmp3.playMp3FolderTrack(4);  // 播放第 4 首音樂
  }
  delay(sw);  // 每次 LED 亮起後延遲 3 秒，避免音效重疊播放
}

// 更新 TM1637 顯示器數值顯示距離
void updateDisplay(long distance) {
  if (distance > 9999) {  // 若距離超過 9999，顯示 ----
    tm1637.display(0, 10);  // 顯示器上每位數字設定為 10，表示不顯示數字
    tm1637.display(1, 10);  
    tm1637.display(2, 10);  
    tm1637.display(3, 10);  
  } else {  // 正常距離顯示
    tm1637.display(0, distance / 1000);          // 顯示千位數
    tm1637.display(1, (distance / 100) % 10);    // 顯示百位數
    tm1637.display(2, (distance / 10) % 10);     // 顯示十位數
    tm1637.display(3, distance % 10);            // 顯示個位數
  }
}

