#include <Arduino.h>
#include <Wire.h>
#include <M5Unified.h>

// LSM9DS1(外付けのIMU)のI2Cアドレスとレジスタ
#define LSM9DS1_AG_ADDR 0x6A
#define CTRL_REG1_G 0x10
#define OUT_X_L_G 0x18
#define OUT_X_L_XL 0x28
#define GYRO_SCALE 0.01750

// PWM設定
#define PWM_PIN 5  // PWM出力用ピン(適宜変更すべし)

const float gyroThreshold = 100.0;
bool inSpecialMode = false;
bool gyroZNegativeFlag = false;
unsigned long specialModeStartTime = 0;
unsigned long gyroZNegativeStartTime = 0;  
unsigned long lastSpecialModeEndTime = 0;  
unsigned long startTime;  
unsigned int alpha =0;  
unsigned int beta = 400;  
const unsigned int ga =1000;  
const unsigned int alphaMax = 350;  
unsigned long lastAlphaIncreaseTime = 0;  // alphaを最後に増加させた時間

//使用する前に、以下の変数を設定してください
bool use_LSM9DS1 = false;//加速度センサがLSM9DS1であるかどうか

void init_LSM9DS1() {
    Wire.begin();  // I2Cの初期化
    Wire.beginTransmission(LSM9DS1_AG_ADDR);
    Wire.write(CTRL_REG1_G);
    Wire.write(0x68);
    Wire.endTransmission();
}

float read_gyro_Z() {
    Wire.beginTransmission(LSM9DS1_AG_ADDR);
    Wire.write(OUT_X_L_G | 0x80);
    Wire.endTransmission(false);
    Wire.requestFrom(LSM9DS1_AG_ADDR, (uint8_t)6);
    uint8_t data[6];
    for (int i = 0; i < 6; i++) {
        data[i] = Wire.read();
    }
    int16_t gyroZ = (int16_t)(data[5] << 8) | data[4];
    return gyroZ * GYRO_SCALE;
}

float read_gyro_Z_MPU6886() {
  float gyroX = 0.0F, gyroY = 0.0F, gyroZ = 0.0F;
    M5.Imu.getGyroData(&gyroX, &gyroY, &gyroZ);
    return gyroZ;
}

void setup() {
    Serial.begin(115200);
    if(use_LSM9DS1){
      init_LSM9DS1();
    }

    // PWMの設定
    ledcSetup(1, 5000, 8);  // チャネル1, 5 kHz, 8bit解像度(不可動作時,要周波数上昇
    ledcAttachPin(PWM_PIN, 1);  // PWM_PIN_2をチャネル1に割り当て

    startTime = millis();  // スタートタイムを設定
    lastAlphaIncreaseTime = millis();  // 初期化時にalphaを増加させた時間を設定
}
void loop() {
    float gyroZ;
    if(use_LSM9DS1){
      gyroZ = read_gyro_Z();
    }else{
      gyroZ = read_gyro_Z_MPU6886();
    }

    unsigned long currentTime = millis();
    unsigned long elapsedTime = currentTime - startTime;
    Serial.printf("alpha: %u\n", alpha);
    Serial.printf("beta: %u\n", beta);
    Serial.printf("Gyro: %f\n", gyroZ);
    //Serial.printf("inSpecialMode: %f\n", inSpecialMode);

    // alphaを増加させる条件チェック
    if (currentTime - lastAlphaIncreaseTime > 15000) {  // 20秒ごとにチェック
        if (alpha < alphaMax) {
            alpha++;
        }
        lastAlphaIncreaseTime = currentTime;  // alpha増加時間を更新
    }
  
    // 特別モードの条件チェック
    if (abs(gyroZ) < gyroThreshold && !inSpecialMode) {
        specialModeStartTime = currentTime;
        inSpecialMode = true;
        lastSpecialModeEndTime = currentTime;
    } if (!inSpecialMode && elapsedTime > 360000 && currentTime - startTime < 365000) {
        //変化を監視するにはここに画面の色を変更するコードを書く
    } 

    if (gyroZ < -200) {
        if (!gyroZNegativeFlag) {
            gyroZNegativeStartTime = currentTime;
        }
        gyroZNegativeFlag = true;
    }

    if (gyroZNegativeFlag && (currentTime - gyroZNegativeStartTime > 500)) {
        gyroZNegativeFlag = false;
    }

    if (gyroZNegativeFlag) {
        inSpecialMode = false;
        //変化を監視するにはこの行に画面の色を変更するコードを書く
        ledcWrite(1, 255);
    }

    if (abs(gyroZ) > 200) {
        inSpecialMode = false;
        //変化を監視するにはこの行に画面の色を変更するコードを書く
        ledcWrite(1, 255);
    }

   if (abs(gyroZ) > 330) {
        inSpecialMode = false;
        //変化を監視するにはこの行に画面の色を変更するコードを書く
        ledcWrite(1, 210);
    }
    // 特別モードの実行
    if (inSpecialMode) {
        if (currentTime - specialModeStartTime <5) {
            //変化を監視するにはこの行に画面の色を変更するコードを書く
            ledcWrite(1, 200);
        } else if (currentTime - specialModeStartTime < 310) {
            //変化を監視するにはこの行に画面の色を変更するコードを書く
            ledcWrite(1, 0);
        } else if (currentTime - specialModeStartTime < 1000) {
            //変化を監視するにはこの行に画面の色を変更するコードを書く
            ledcWrite(1, 255);
        } else if (currentTime - specialModeStartTime < 20000) {
            //変化を監視するにはこの行に画面の色を変更するコードを書く
            ledcWrite(1, 200);
        } else {
            Serial.println("Exited special mode (back to blue LED)");
            //変化を監視するにはこの行に画面の色を変更するコードを書く
            inSpecialMode = false;
            
        }
    } 
    //delay(1);
}
