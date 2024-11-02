<h1>HC-SR04超音波感測器與RGB LED燈 + TM1637 4位七段顯示器與蜂鳴器(感應雷達之功能)</h1>


## 準備材料 : 
>1. Arduino Nano板(CH340驅動程式.USB:MiniUSB) 
>2. MiniUSB 連接線 X 1 
>3. HC-SR04超音波感測器 Trig->pin2 , Echo->pin3
>4. RGB LED燈
>5. TM1637 4位七段顯示器
>6. 蜂鳴器
>7. 麵包板 X 1===

## 電路圖
>![](https://github.com/user-attachments/assets/78b4cbee-986e-4c92-9e61-f84fcd244e13)

## 程式碼

``` arduino
#include "TM1637.h"        //主程式需要程式庫 “TM1637.h”

// 定義引腳
const int trigPin = 2;    // Trig引腳
const int echoPin = 3;    // Echo引腳
const int redPin = 9;     // 紅燈引腳
const int greenPin = 10;  // 綠燈引腳
const int bluePin = 11;   // 藍燈引腳
const int CLK = 4;        // TM1637 CLK引腳
const int DIO = 5;        // TM1637 DIO引腳
const int buzzerPin = 12; // 蜂鳴器引腳

// 初始化TM1637顯示器
TM1637 tm1637(CLK, DIO);    // 宣告顯示晶片涵式庫

void setup() {
  // 初始化串列監視器
  Serial.begin(9600);
  
  // 設置引腳模式
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);
  pinMode(buzzerPin, OUTPUT); // 設置蜂鳴器引腳

  tm1637.init();
  tm1637.set(BRIGHT_TYPICAL);
}

void loop() {
  // 清除Trig引腳
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  
  // 發送超聲波信號
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);  // 發射時間10微秒
  digitalWrite(trigPin, LOW);
  
  // 讀取Echo引腳的持續時間
  long duration = pulseIn(echoPin, HIGH);
  
  // 計算距離（厘米）
  long distance = duration * 0.034 / 2; // 距離 = 時間 * 速度 / 2
  
  // 輸出距離到串列監視器
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");
  
  // 清除LED狀態
  digitalWrite(redPin, LOW);
  digitalWrite(greenPin, LOW);
  digitalWrite(bluePin, LOW);
  
  // 控制RGB LED燈的顏色與蜂鳴器頻率
  if (distance > 70) {
    // 距離大於70公分，亮綠燈，不發出聲音
    digitalWrite(greenPin, HIGH);
    digitalWrite(bluePin, LOW);
    digitalWrite(redPin, LOW);
    noTone(buzzerPin); // 不發出聲音
  } else if (distance >= 25 && distance <= 70) {
    // 距離在25公分到70公分之間，亮白燈，蜂鳴器以較慢頻率叫
    digitalWrite(redPin, HIGH);
    digitalWrite(greenPin, HIGH);
    digitalWrite(bluePin, HIGH);
    tone(buzzerPin, 1000); // 蜂鳴器發出1kHz聲音
    delay(200);            // 延遲200毫秒
    noTone(buzzerPin);      // 停止聲音
    delay(200);             // 間隔200毫秒
  } else {
    // 距離小於等於25公分，亮紅燈，蜂鳴器以較快頻率叫
    digitalWrite(redPin, HIGH);
    digitalWrite(greenPin, LOW);
    digitalWrite(bluePin, LOW);
    tone(buzzerPin, 1000); // 蜂鳴器發出1kHz聲音
    delay(50);            // 延遲100毫秒
    noTone(buzzerPin);      // 停止聲音
    delay(100);             // 間隔100毫秒
  } if (distance < 10) {
    // 距離小於10公分，亮紅燈並持續響蜂鳴器
    digitalWrite(redPin, HIGH);
    tone(buzzerPin, 1000);  // 蜂鳴器持續響
  }
  // 在TM1637顯示距離
  int displayDistance = (distance > 9999) ? 9999 : distance;  // 限制距離最大顯示為9999
  tm1637.display(0, displayDistance / 1000);         // 千位數
  tm1637.display(1, (displayDistance / 100) % 10);   // 百位數
  tm1637.display(2, (displayDistance / 10) % 10);    // 十位數
  tm1637.display(3, displayDistance % 10);           // 個位數
  
  // 等待一段時間以便進行下一次測量
  delay(500);
}
