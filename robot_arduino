#include <Wire.h>
#include <Adafruit_PWMServoDriver.h>


Adafruit_PWMServoDriver pwm = Adafruit_PWMServoDriver();


// --- 서보 모터 설정 ---
#define SERVOMIN 150
#define SERVOMAX 600
#define BASE_SERVO     0
#define SHOULDER_SERVO 1
#define ELBOW_SERVO    2
#define WRIST_SERVO    3


// ★★★ 라인 트래킹 센서 및 TB6612FNG 모터 드라이버 핀 설정 (A, B 채널) ★★★
#define LEFT_IR_PIN   A0
#define RIGHT_IR_PIN  A1
#define IR_THRESHOLD  500 // 센서 감지 기준값 (실제 환경에 맞게 조절 필요)


// 모터 A (예: 왼쪽 바퀴)
#define MOTOR_PWMA    9
#define MOTOR_AIN1    8
#define MOTOR_AIN2    4


// 모터 B (예: 오른쪽 바퀴)
#define MOTOR_PWMB    10
#define MOTOR_BIN1    13
#define MOTOR_BIN2    12


#define MOTOR_STBY    6  // 드라이버 활성화


// --- 카메라 좌표 변환 설정 ---
#define X_SCALE 0.1f    // 픽셀 -> mm 변환 계수 (카메라 세로축)
#define Y_SCALE 0.12f   // 픽셀 -> mm 변환 계수 (카메라 가로축)
#define X_OFFSET 240.0f  // 가상 원점 Y좌표 (카메라 세로축 중앙: 480/2)
#define Y_OFFSET 320.0f  // 가상 원점 X좌표 (카메라 가로축 중앙: 640/2)
#define SIGN_X (+1.0f)
#define SIGN_Y (-1.0f)


// --- 릴레이 핀 설정 ---
int relayPin = 7;


// --- 데이터 저장 구조체 ---
#define MAX_COORDS 10
struct Coord { int x; int y; int z; };
Coord coords[MAX_COORDS];
int coordCount = 0;
int currentIndex = 0;
bool processing = false;


// --- 로봇팔 물리적 제원 ---
const float L1 = 130.0f;
const float L2 = 90.0f;
const float L3 = 110.0f;
const float Z0 = 385.0f;


// --- 서보 캘리브레이션 ---
const float BASE_ZERO     = 70.0f;  const int BASE_DIR     = +1;
const float SHOULDER_ZERO = 0.0f;   const int SHOULDER_DIR = +1;
const float ELBOW_ZERO    = 10.0f;  const int ELBOW_DIR    = -1;
const float WRIST_ZERO    = 80.0f;  const int WRIST_DIR    = +1;
float currentAngles[4] = {BASE_ZERO, SHOULDER_ZERO, ELBOW_ZERO, WRIST_ZERO};


// --- 서보 안전 한계 ---
const float BASE_MIN=10.0f,     BASE_MAX=150.0f;
const float SHOULDER_MIN=0.0f,  SHOULDER_MAX=180.0f;
const float ELBOW_MIN=0.0f,     ELBOW_MAX=140.0f;
const float WRIST_MIN=0.0f,     WRIST_MAX=180.0f;


// ★★★ 로봇이 실제 작업할 앞/뒤 거리 (이 값을 실제 환경에 맞게 조절) ★★★
const float WORK_DISTANCE = 100.0f; // 예: 15cm 앞에서 작업
// ★★★ 대기할 앞/뒤 거리 (작업 거리 - 6cm) ★★★
const float STANDBY_DISTANCE = WORK_DISTANCE - 60.0f;


// --- 유틸 함수 ---
inline float rad2deg(float r){ return r * 57.29577951308232f; }
inline float clampf(float v, float lo, float hi){ return (v<lo)?lo:((v>hi)?hi:v); }
inline float to_mm_x(float v_px) { return SIGN_X * (v_px - X_OFFSET) * X_SCALE; }
inline float to_mm_y(float u_px) { return SIGN_Y * (u_px - Y_OFFSET) * Y_SCALE; }




// =================================================================================
// ---------------------------- 역기구학(IK) 함수 -----------------------------
// =================================================================================
bool solveIK_YawWrist(float x, float y, float z,
                      float &baseDeg, float &shoulderDeg, float &elbowDeg, float &wristDeg,
                      bool elbowUp=false)
{
    float idealBaseYaw = rad2deg(atan2(y, x));
    const float YAW_MIN = (BASE_MIN - BASE_ZERO) / BASE_DIR;
    const float YAW_MAX = (BASE_MAX - BASE_ZERO) / BASE_DIR;
    float actualBaseYaw = clampf(idealBaseYaw, YAW_MIN, YAW_MAX);
    float wristYawLocal = idealBaseYaw - actualBaseYaw;


    float r = sqrt(x*x + y*y);
    float z_rel = z - Z0;
    float r_w = r - L3;
    float z_w = z_rel;
   
    float d2 = r_w*r_w + z_w*z_w;
    float d = sqrt(d2);
    float maxReach = L1 + L2;
    if (d > maxReach) d = maxReach;
    if (d < 0) d = 0;


    float D = (d2 - L1*L1 - L2*L2) / (2.0f * L1 * L2);
    D = clampf(D, -1.0f, 1.0f);
   
    float th2 = acos(D);
    float th1 = atan2(z_w, r_w) - atan2(L2 * sin(th2), L1 + L2 * cos(th2));
    float shoulderDegLocal = rad2deg(th1);
    float elbowDegLocal = rad2deg(th2);


    baseDeg     = clampf(BASE_ZERO     + BASE_DIR * actualBaseYaw, BASE_MIN, BASE_MAX);
    shoulderDeg = clampf(SHOULDER_ZERO + SHOULDER_DIR * shoulderDegLocal, SHOULDER_MIN, SHOULDER_MAX);
    elbowDeg    = clampf(ELBOW_ZERO    + ELBOW_DIR * elbowDegLocal, ELBOW_MIN, ELBOW_MAX);
    wristDeg    = clampf(WRIST_ZERO    + WRIST_DIR * wristYawLocal, WRIST_MIN, WRIST_MAX);
   
    return true;
}


// =================================================================================
// ---------------------------- 제어 함수 -----------------------------
// =================================================================================
void parseCoords(String input) {
    coordCount = 0;
    input.trim();
    if (input.endsWith("/")) input.remove(input.length()-1);


    int start = 0;
    while (coordCount < MAX_COORDS) {
        int slashIndex = input.indexOf('/', start);
        String part = (slashIndex == -1) ? input.substring(start) : input.substring(start, slashIndex);
        int commaIndex = part.indexOf(',');
        if (commaIndex == -1) break;
        coords[coordCount].x = part.substring(0, commaIndex).toInt();
        coords[coordCount].y = part.substring(commaIndex+1).toInt();
        coordCount++;
        if (slashIndex == -1) break;
        start = slashIndex + 1;
    }
}


void setServoAngle(int servo, float angle) {
    angle = constrain(angle, 0.0f, 180.0f);
    int pulse = map((int)angle, 0, 180, SERVOMIN, SERVOMAX);
    pwm.setPWM(servo, 0, pulse);
}


void moveArmSmooth(float b, float s, float e, float w, int steps=50, int delayMs=10) {
    float targetAngles[4] = {b, s, e, w};
    int servoPins[4] = {BASE_SERVO, SHOULDER_SERVO, ELBOW_SERVO, WRIST_SERVO};
    for (int i = 0; i < 4; i++) {
        int currentServo = servoPins[i];
        float startAngle = currentAngles[currentServo];
        float targetAngle = targetAngles[i];
        for (int j = 0; j <= steps; j++) {
            float ratio = (float)j / (float)steps;
            float angle = startAngle + (targetAngle - startAngle) * ratio;
            setServoAngle(currentServo, angle);
            delay(delayMs);
        }
        currentAngles[currentServo] = targetAngle;
    }
}


// ★★★ 2채널 DC 모터 제어 함수 (TB6612FNG용) ★★★
void setMotorSpeed(int leftSpeed, int rightSpeed) {
  // 왼쪽 모터 제어 (0~255)
  if (leftSpeed >= 0) {
    digitalWrite(MOTOR_AIN1, HIGH);
    digitalWrite(MOTOR_AIN2, LOW);
    analogWrite(MOTOR_PWMA, leftSpeed);
  } else {
    digitalWrite(MOTOR_AIN1, LOW);
    digitalWrite(MOTOR_AIN2, HIGH);
    analogWrite(MOTOR_PWMA, -leftSpeed);
  }


  // 오른쪽 모터 제어 (0~255)
  if (rightSpeed >= 0) {
    digitalWrite(MOTOR_BIN1, HIGH);
    digitalWrite(MOTOR_BIN2, LOW);
    analogWrite(MOTOR_PWMB, rightSpeed);
  } else {
    digitalWrite(MOTOR_BIN1, LOW);
    digitalWrite(MOTOR_BIN2, HIGH);
    analogWrite(MOTOR_PWMB, -rightSpeed);
  }
}


void stopMotor() {
  setMotorSpeed(0, 0);
}


// ★★★ 라인 트래킹 동작 함수 (2모터 차동 구동 방식) ★★★
void startLineTracking() {
    while (true) {
        bool leftSeesLine = (analogRead(LEFT_IR_PIN) > IR_THRESHOLD);
        bool rightSeesLine = (analogRead(RIGHT_IR_PIN) > IR_THRESHOLD);


        if (leftSeesLine && rightSeesLine) {
            stopMotor();
            Serial.println("END");   // ✅ 라즈베리파이에 전달
            break;
        }
        else if (leftSeesLine && !rightSeesLine) {
            setMotorSpeed(80, 200);
        }
        else if (!leftSeesLine && rightSeesLine) {
            setMotorSpeed(200, 80);
        }
        else {
            setMotorSpeed(200, 200);
        }


        delay(10);
    }
}




// =================================================================================
// ---------------------------- 메인 실행 영역 -----------------------------
// =================================================================================
void setup() {
    Serial.begin(115200);
    pwm.begin();
    delay(10);
    pwm.setPWMFreq(50);


    // 모터 핀 6개 및 릴레이 핀 초기화
    pinMode(MOTOR_PWMA, OUTPUT);
    pinMode(MOTOR_AIN1, OUTPUT);
    pinMode(MOTOR_AIN2, OUTPUT);
    pinMode(MOTOR_PWMB, OUTPUT);
    pinMode(MOTOR_BIN1, OUTPUT);
    pinMode(MOTOR_BIN2, OUTPUT);
    pinMode(MOTOR_STBY, OUTPUT);
    pinMode(relayPin, OUTPUT);


    digitalWrite(relayPin, LOW);
    digitalWrite(MOTOR_STBY, HIGH);
    stopMotor();


    moveArmSmooth(currentAngles[0], currentAngles[1], currentAngles[2], currentAngles[3]);
}


void loop() {
  // 1. 라즈베리파이로부터 명령 수신
  if (Serial.available() > 0) {
    String input = Serial.readStringUntil('\n');
    input.trim();


    if (input == "None") {
        startLineTracking();
    }
    else {
        parseCoords(input);
        if (coordCount > 0) {
         
         
          processing = true;
          currentIndex = 0;
        }
    }
  }


  // 2. 수확 작업 처리
  if (processing) {
    if (currentIndex < coordCount) {
      Coord target = coords[currentIndex];
     
      float y_mm = to_mm_y(target.x);
      float z_mm = map(target.y, 0, 480, 280, 120); // 측정된 값으로 수정


      float b, s, e, w;


      // 2-1. 대기 위치로 이동
     
      bool success_standby = solveIK_YawWrist(STANDBY_DISTANCE, y_mm, z_mm, b, s, e, w);
     
      if (success_standby) {
        moveArmSmooth(b, s, e, w);
        delay(500);


        // 2-2. 작업 위치로 전진
       
        bool success_work = solveIK_YawWrist(WORK_DISTANCE, y_mm, z_mm, b, s, e, w);


        if (success_work) {
          moveArmSmooth(b, s, e, w);


          // 2-3. 수확
          digitalWrite(relayPin, HIGH);
          delay(2000);
          digitalWrite(relayPin, LOW);
          delay(1000);


          // 2-4. 대기 위치로 후진
         
          solveIK_YawWrist(STANDBY_DISTANCE, y_mm, z_mm, b, s, e, w);
          moveArmSmooth(b, s, e, w);
        }
       
      } else {
        Serial.println("Could not solve IK for STANDBY coordinate.");
      }
     
      currentIndex++;
    }
    // 3. 한 묶음의 수확이 모두 끝나면
    else {
      processing = false;
     
      moveArmSmooth(BASE_ZERO, SHOULDER_ZERO, ELBOW_ZERO, WRIST_ZERO);


      // 라즈베리파이에게 "다시 확인해줘!" 신호 전송
      Serial.println("CHECK");
    }
  }
}

