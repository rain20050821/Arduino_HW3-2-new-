# Arduino_HW3-2-new-
const int buttonPin = 8;
const int ledPin = 3;

unsigned long Lastmillis;
int T[] = { 0, 1000, 500 ,100, 0 };
int state = 0;

// 新增變數來記錄按鈕上一次的狀態，以防止連發
bool lastButtonState = LOW; 
// 新增變數來記錄 LED 目前是亮還是暗 (用於閃爍)
int currentLedState = HIGH; 

// 宣告函式
bool readDebounced(int pin);

void setup() {
  pinMode(ledPin, OUTPUT);
  pinMode(buttonPin, INPUT); 
  
  state = 0;
  Lastmillis = millis();
  
  // 初始狀態為恆亮
  digitalWrite(ledPin, HIGH); 
}

void loop() {
  // 1. 讀取經過防彈跳的按鈕狀態
  bool reading = readDebounced(buttonPin);

  // 2. 偵測「按下瞬間」 (這次是 HIGH，上次是 LOW)
  if (reading == HIGH && lastButtonState == LOW) {
    state++;
    if (state > 3) {
      state = 0;
    }
    // 切換模式時，重置計時器跟燈光狀態，讓反應更即時
    Lastmillis = millis(); 
    if (state == 0) digitalWrite(ledPin, HIGH); // 切回恆亮模式馬上亮
  }
  
  // 更新上一次的按鈕狀態，供下一圈迴圈比對
  lastButtonState = reading;

  // 3. 燈光控制邏輯
  if (state == 0) {
    // 模式 0: 恆亮
    digitalWrite(ledPin, HIGH);
  } 
  else {
    // 模式 1~3: 閃爍 (根據 T[state] 的時間)
    if (millis() - Lastmillis >= T[state]) {
      Lastmillis = millis(); // 更新時間
      
      // 反轉 LED 狀態 (亮變暗、暗變亮)
      if (currentLedState == HIGH) {
        currentLedState = LOW;
      } else {
        currentLedState = HIGH;
      }
      digitalWrite(ledPin, currentLedState);
    }
  }
}

// 防彈跳函式
bool readDebounced(int pin) {
  static bool lastRawState = HIGH;
  static bool stableState = HIGH;
  static unsigned long lastDebounceTime = 0;
  const int debounceDelay = 50;

  bool reading = digitalRead(pin);

  if (reading != lastRawState) 
    lastDebounceTime = millis();

  if ((millis() - lastDebounceTime) > debounceDelay) {
    if (reading != stableState)
      stableState = reading;
  }

  lastRawState = reading;
  return stableState;
}
