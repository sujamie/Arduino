<h1>HC-SR04超音波感測器與RGB LED燈</h1>


## 準備材料 : 
>1. Arduino Nano板(CH340驅動程式.USB:MiniUSB) 
>2. MiniUSB 連接線 X 1 
>3. HC-SR04超音波感測器 Trig->pin2 , Echo->pin3
>4. RGB LED燈 Red->pin9 , Green->pin10 , blue->pin11
>5. 麵包板 X 1===

## 電路圖

>![](https://github.com/user-attachments/assets/cfc33d27-fa9f-46df-a119-a58b5cc93f92)

## 程式碼

``` arduino
// 定義引腳
const int trigPin = 2; // Trig引腳
const int echoPin = 3; // Echo引腳
const int redPin = 9;   // 紅燈引腳
const int greenPin = 10; // 綠燈引腳
const int bluePin = 11;  // 藍燈引腳

void setup() {
  // 初始化串列監視器
  Serial.begin(9600);
  
  // 設置引腳模式
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);
}

void loop() {
  // 清除Trig引腳
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  
  // 發送超聲波信號
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);  // 應用10微秒發射時間
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
  
  // 控制RGB LED燈的顏色
  if (distance > 70) {
    // 距離大於70公分，亮綠燈
    digitalWrite(greenPin, HIGH);
    digitalWrite(bluePin, LOW);
    digitalWrite(redPin, LOW);
  } else if (distance >= 25 && distance <= 70) {
    // 距離在25公分到70公分之間，亮白燈
    digitalWrite(redPin, HIGH);
    digitalWrite(greenPin, HIGH);
    digitalWrite(bluePin, HIGH);
    
     // 黃燈需要同時亮綠燈和紅燈
  } else {
    // 距離小於等於25公分，亮紅燈
    digitalWrite(redPin, HIGH);
    digitalWrite(greenPin, LOW);
    digitalWrite(bluePin, LOW);
  }
  
  // 等待一段時間以便進行下一次測量
  delay(500);
}
