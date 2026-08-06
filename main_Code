// =========================
// Mikey Config Header
// =========================

// --- Motor Calibration Trim ---
#define LEFT_MOTOR_TRIM    0  
#define RIGHT_MOTOR_TRIM   40

// --- Default LED Color (Electric Ice Cyan) ---
#define DEFAULT_LED_COLOR  strip.Color(0, 210, 255)

// --- Mood LED Colors ---
#define LED_COLOR_NEUTRAL  strip.Color(0, 210, 255)   // cyan
#define LED_COLOR_HAPPY    strip.Color(255, 180, 60)  // warm orange/yellow
#define LED_COLOR_PLAYFUL  strip.Color(60, 180, 255)  // bright blue/cyan
#define LED_COLOR_SHY      strip.Color(100, 80, 180)  // dim purple/blue

// --- Motor Ramping ---
#define MOTOR_RAMP_STEP 10      // PWM change per 20ms step
#define MOTOR_RAMP_INTERVAL_MS 20

// --- IR Thresholds (defaults, can be changed via web) ---
#define DEFAULT_LEFT_CLIFF_THRESHOLD   600
#define DEFAULT_RIGHT_CLIFF_THRESHOLD  2000
#define DEFAULT_WALL_DANGER_THRESHOLD  1000

// Cliff margin for auto-threshold: threshold = current_reading + CLIFF_MARGIN
#define CLIFF_MARGIN 40

// --- Mood System ---
#define MOOD_DECAY_MS 60000UL       // 1 minute mood duration

// --- Idle / Inactivity ---
#define AUTO_IDLE_TIMEOUT_MS 45000UL
#define AUTO_NOISE_INTERVAL_MS 9000UL
#define IDLE_LOOK_AROUND_CHANCE 4   // 1 in 4 chance to do idle look-around

// --- Breathing LED ---
#define BREATHING_MIN_BRIGHT 5
#define BREATHING_MAX_BRIGHT 100

// --- Touch Timing ---
#define PET_MIN_DURATION_MS 500
#define TAP_FACE_TIME_MS 1000
#define TAP_MOOD_TIME_MS 4000

// --- Face Reset / Timing ---
#define DEFAULT_FACE_RESET_MS 3000
#define SNEEZE_FACE_RESET_MS 3000
#define SLEEP_FACE_RESET_MS 6000
#define CHIRP_FACE_RESET_MS 2500
#define PET_FACE_RESET_MS 4500
#define STORY_FACE_RESET_MS 4000

// --- Audio Timing (approximate, adjust if needed) ---
#define AUDIO_BOOT_DUR 4000
#define AUDIO_SHORT_INTRO_DUR 3000
#define AUDIO_FULL_INTRO_DUR 8000
#define AUDIO_WALL_ALERT_DUR 3500
#define AUDIO_CLIFF_ALERT_DUR 3500
#define AUDIO_TAP_DUR 4000
#define AUDIO_PET_DUR 4500
#define AUDIO_MODE_DUR 3000
#define AUDIO_DANCE_DUR 16000
#define AUDIO_SNEEZE_DUR 2500
#define AUDIO_SLEEP_DUR 5000
#define AUDIO_CHIRP_DUR 2000
#define AUDIO_DIALOGUE_DUR 4000
#define AUDIO_STORY_PART_DUR 4000
#define AUDIO_SFX_BLIP_DUR 400

// --- Pins ---
#define PIN_IR_LEFT    0
#define PIN_IR_WALL    1
#define PIN_IR_RIGHT   2
#define MOTOR_A_IN1    3
#define MOTOR_A_IN2    4
#define MOTOR_B_IN1    6
#define MOTOR_B_IN2    5
#define PIN_LED        7
#define OLED_SDA       8
#define OLED_SCL       9
#define PIN_TOUCH      10
#define DFPLAYER_TX    20
#define DFPLAYER_RX    21

// =========================
// Includes & Globals
// =========================

#include <Arduino.h>
#include <Wire.h>
#include <WiFi.h>
#include <esp_wifi.h>
#include <WebServer.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SH110X.h>
#include <Adafruit_NeoPixel.h>
#include <DFRobotDFPlayerMini.h>

// --- DFPlayer Sound Track Mapping ---
// 1: Boot
// 2: Short Intro
// 3: Full Intro
// 4: Wall Alert
// 5: Cliff Alert
// 6: Ouch/Tapped
// 7: Petting Happy
// 8: Manual Mode
// 9: Auto Mode
// 10: Action Mode
// 11: Dance 1 Music
// 12: Dance 2 Music
// 13: Dance 3 Music
// 14: Curious Sound
// 15: Happy Sound
// 16: Sneeze Sound
// 17: Sleep Sound
// 18: Chirp/Whistle SFX
// 19: Boing/Giggly Pitch Mod SFX
// 20: Dialogue 1
// 21: Dialogue 2
// 22: Dialogue 3
// 23: Wall Lesson line
// 24: Story part 1 (curious)
// 25: Story part 2 (realization)
// 26: Story part 3 (happy)
// 27: SFX Blip (optional)

// --- Objects ---
Adafruit_SH1106G display(128, 64, &Wire, -1);
Adafruit_NeoPixel strip(8, PIN_LED, NEO_GRB + NEO_KHZ800);
HardwareSerial FPSerial(1);
DFRobotDFPlayerMini myDFPlayer;
WebServer server(80);

// --- State Machines ---
enum RobotMode { MANUAL, AUTO, ACTION };
RobotMode currentMode = MANUAL;

enum FaceState { NORMAL, DOWN, SURPRISED, PAIN, ANGRY, HAPPY, CURIOUS, SLEEP, SNEEZE, THINKING, SHOCKED };
FaceState currentFace = NORMAL;

// --- Mood System ---
enum RobotMood { MOOD_NEUTRAL, MOOD_HAPPY, MOOD_SHY, MOOD_PLAYFUL };
RobotMood currentMood = MOOD_NEUTRAL;
unsigned long moodDecayTimer = 0;

// --- Dynamic thresholds ---
int LEFT_CLIFF_THRESHOLD = DEFAULT_LEFT_CLIFF_THRESHOLD;
int RIGHT_CLIFF_THRESHOLD = DEFAULT_RIGHT_CLIFF_THRESHOLD;
int WALL_DANGER_THRESHOLD = DEFAULT_WALL_DANGER_THRESHOLD;

// --- Global Variables ---
bool oledConnected = false; 
String webAlertMsg = "System Online";
unsigned long lastSensorCheck = 0;
unsigned long faceResetTime = 0;
unsigned long lastAutoMove = 0;
unsigned long lastAutoNoise = 0;
unsigned long lastAutoActivity = 0;

// Animation & Audio Timers
unsigned long audioBusyUntil = 0; 
unsigned long audioSpeechStartTime = 0; 
FaceState faceSequenceNext = NORMAL;
unsigned long faceSequenceTime = 0;

// OLED Refresh Tracking
FaceState lastDrawnFace = NORMAL;
bool lastDrawnSpeaking = false;

// Procedural Animation Variables
unsigned long lastBlinkTime = 0;
unsigned long nextBlinkDelay = 3000;
bool isBlinking = false;
unsigned long blinkStartTime = 0;
float pupilOffsetX = 0;
float pupilOffsetY = 0;
unsigned long lastEyeDart = 0;
int lastDrawnEyeHeight = 26; 

// Dance Logic
bool isDancing = false;
int activeDanceType = 1;
int dancePhase = 0;
unsigned long nextDanceMove = 0;

int liveLeftCliff = 0;
int liveWall = 0;
int liveRightCliff = 0;

bool dangerAlertActive = false;
bool isEvading = false;
unsigned long evasionTimer = 0;
int evasionState = 0;

// Touch Logic
bool lastTouchState = false;
unsigned long touchStartTime = 0;
bool isTouching = false;
bool isPetting = false; 

// --- Motor Ramping ---
int targetSpeedL = 0;
int targetSpeedR = 0;
int currentSpeedL = 0;
int currentSpeedR = 0;
unsigned long lastMotorRampUpdate = 0;

// --- Story / Script Player ---
struct ScriptStep {
  FaceState face;
  int soundTrack;      // 0 = no sound
  unsigned long soundDur;
  int moveL;           // -255..255
  int moveR;
  unsigned long stepDur;
};

bool isRunningScript = false;
unsigned long scriptStartTime = 0;
ScriptStep currentScriptStep;
int currentScriptIndex = 0;
const int SCRIPT_STORY_LEN = 3;

// Updated story: 
// Step 0: curious, no motion
// Step 1: move forward a little during "Wait... is that a new pen...?"
// Step 2: move slightly backward during "This is awesome..."
ScriptStep scriptStory[SCRIPT_STORY_LEN] = {
  { CURIOUS, 24, AUDIO_STORY_PART_DUR,   0,   0, AUDIO_STORY_PART_DUR },
  { SHOCKED, 25, AUDIO_STORY_PART_DUR,  70,  70, AUDIO_STORY_PART_DUR }, // forward
  { HAPPY,   26, AUDIO_STORY_PART_DUR, -40, -40, AUDIO_STORY_PART_DUR }  // slight backward
};

// --- Helpers for mood-based LED ---
uint32_t getMoodColor() {
  switch (currentMood) {
    case MOOD_HAPPY:   return LED_COLOR_HAPPY;
    case MOOD_PLAYFUL: return LED_COLOR_PLAYFUL;
    case MOOD_SHY:     return LED_COLOR_SHY;
    default:           return DEFAULT_LED_COLOR;
  }
}

void setMoodColor() {
  if (isDancing) return; // dance overrides LED
  uint32_t c = getMoodColor();
  strip.fill(c);
  strip.show();
}

// =========================
// Web Dashboard HTML
// =========================

const char INDEX_HTML[] PROGMEM = R"rawliteral(
<!DOCTYPE html><html><head><meta name="viewport" content="width=device-width, initial-scale=1">
<style>
  body { font-family: Arial; text-align: center; background: #111; color: #eee; padding: 15px; }
  h2 { margin: 0 0 10px; }
  .alert { background: #dc3545; padding: 10px; border-radius: 8px; margin: 10px auto; max-width: 340px; font-weight: bold; min-height:20px;}
  .status-bar { background: #222; padding: 8px; border-radius: 8px; margin: 10px auto; max-width: 340px; display:flex; justify-content:space-around; font-size:13px; }
  .sensor-box { background: #222; padding: 10px; border-radius: 8px; margin: 10px auto; max-width: 300px; font-family: monospace; font-size: 14px; text-align: left; }
  .section { border: 1px solid #333; margin-top: 12px; padding: 10px; border-radius: 8px; max-width: 340px; margin-left: auto; margin-right: auto;}
  .section h3 { margin-top: 0; font-size: 16px; }
  button { padding: 12px 14px; margin: 4px; font-size: 14px; border-radius: 8px; border: none; background: #007bff; color: #fff; cursor: pointer; font-weight: bold;}
  button:active { background: #0056b3; }
  .grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; max-width: 260px; margin: 10px auto;}
  input[type="color"] { width: 80px; height: 40px; border: none; border-radius: 4px; }
  .btn-dance { background: #28a745; }
  .btn-sfx { background: #17a2b8; }
  .btn-sleep { background: #6c757d; }
  .btn-sneeze { background: #e83e8c; }
  .btn-dialogue { background: #6f42c1; }
  .btn-story { background: #fd7e14; }
  .btn-calib { background: #fd7e14; }
  input[type="number"] { width: 80px; padding: 6px; border-radius: 4px; border: 1px solid #555; background: #111; color: #eee; }
  .threshold-row { display: flex; justify-content: space-between; align-items: center; max-width: 300px; margin: 6px auto; }
  .threshold-row label { flex: 1; text-align: left; }
  .threshold-row input { flex: 1; }
  .threshold-row button { flex: 1; margin-left: 6px; }
</style></head><body>
  <h2>Mikey Dashboard</h2>
  <div class="alert" id="alertBox">Connecting...</div>

  <div class="status-bar">
    <div>Mode: <span id="modeText">-</span></div>
    <div>Mood: <span id="moodText">-</span></div>
  </div>
  
  <div class="sensor-box">
    <strong>Live IR Sensors:</strong><br>
    • Left Cliff : <span id="irL">0</span><br>
    • Front Wall : <span id="irW">0</span><br>
    • Right Cliff: <span id="irR">0</span>
  </div>

  <div class="section">
    <h3>Modes</h3>
    <button onclick="cmd('mode','manual')">Manual</button>
    <button onclick="cmd('mode','auto')">Auto Roam</button>
    <button onclick="cmd('mode','action')">Action</button>
  </div>

  <div class="section">
    <h3>Manual Drive</h3>
    <div class="grid">
      <div></div><button onmousedown="cmd('move','fwd')" onmouseup="cmd('move','stop')" ontouchstart="cmd('move','fwd')" ontouchend="cmd('move','stop')">^</button><div></div>
      <button onmousedown="cmd('move','left')" onmouseup="cmd('move','stop')" ontouchstart="cmd('move','left')" ontouchend="cmd('move','stop')"><</button>
      <button onmousedown="cmd('move','stop')" ontouchstart="cmd('move','stop')">STOP</button>
      <button onmousedown="cmd('move','right')" onmouseup="cmd('move','stop')" ontouchstart="cmd('move','right')" ontouchend="cmd('move','stop')">></button>
      <div></div><button onmousedown="cmd('move','bwd')" onmouseup="cmd('move','stop')" ontouchstart="cmd('move','bwd')" ontouchend="cmd('move','stop')">v</button><div></div>
    </div>
  </div>

  <div class="section">
    <h3>Global Actions</h3>
    <button onclick="cmd('action','introFull')">Full Intro</button>
    <button onclick="cmd('action','introShort')">Hi Mikey!</button><br>
    <button class="btn-sfx" onclick="cmd('action','chirp')">Chirp / Whistle</button>
    <button class="btn-sneeze" onclick="cmd('action','sneeze')">Sneeze</button>
    <button class="btn-sleep" onclick="cmd('action','sleep')">Sleep</button><br><br>
    <button class="btn-dance" onclick="cmd('action','dance1')">Dance 1: Shuffle</button>
    <button class="btn-dance" onclick="cmd('action','dance2')">Dance 2: Moonwalk</button>
    <button class="btn-dance" onclick="cmd('action','dance3')">Dance 3: Jitter</button><br><br>
    <button class="btn-dialogue" onclick="cmd('action','dialogue1')">Dialogue 1</button>
    <button class="btn-dialogue" onclick="cmd('action','dialogue2')">Dialogue 2</button>
    <button class="btn-dialogue" onclick="cmd('action','dialogue3')">Dialogue 3</button><br><br>
    <button class="btn-story" onclick="cmd('action','story')">Mini Story</button><br><br>
    <label>LED Color: </label>
    <input type="color" id="ledColor" value="#00d2ff" onchange="changeColor(this.value)">
  </div>

  <div class="section">
    <h3>IR Thresholds</h3>
    <div class="threshold-row">
      <label>Left Cliff:</label>
      <input type="number" id="leftTh" value="725">
      <button onclick="setThreshold('left')">Set</button>
    </div>
    <div class="threshold-row">
      <label>Right Cliff:</label>
      <input type="number" id="rightTh" value="2700">
      <button onclick="setThreshold('right')">Set</button>
    </div>
    <div class="threshold-row">
      <label>Wall:</label>
      <input type="number" id="wallTh" value="1000">
      <button onclick="setThreshold('wall')">Set</button>
    </div>
    <div style="margin-top:10px;">
      <button class="btn-calib" onclick="autoCalibrateCliffs()">Auto Calibrate Left/Right</button>
    </div>
  </div>

<script>
  let thresholdsLoaded = false;
  let currentMode = 'unknown';
  let currentMood = 'unknown';

  function cmd(type, val) { fetch('/cmd?' + type + '=' + val); }
  function changeColor(hex) { fetch('/color?hex=' + hex.substring(1)); }
  function setThreshold(side) {
    var val;
    if(side === 'left') val = document.getElementById('leftTh').value;
    else if(side === 'right') val = document.getElementById('rightTh').value;
    else if(side === 'wall') val = document.getElementById('wallTh').value;
    fetch('/threshold?side=' + side + '&val=' + val);
  }
  function autoCalibrateCliffs() {
    fetch('/calibrateCliffs');
  }

  setInterval(() => {
    fetch('/status').then(r => r.json()).then(d => {
      document.getElementById('alertBox').innerText = d.alert;
      document.getElementById('irL').innerText = d.irL;
      document.getElementById('irW').innerText = d.irW;
      document.getElementById('irR').innerText = d.irR;

      if (!thresholdsLoaded) {
        document.getElementById('leftTh').value = d.leftTh;
        document.getElementById('rightTh').value = d.rightTh;
        document.getElementById('wallTh').value = d.wallTh;
        thresholdsLoaded = true;
      }

      currentMode = d.mode;
      currentMood = d.mood;
      document.getElementById('modeText').innerText = d.mode;
      document.getElementById('moodText').innerText = d.mood;
    });
  }, 500);
</script>
</body></html>
)rawliteral";

// =========================
// Motor & Movement
// =========================

void setMotorsRaw(int speedL, int speedR) {
  if (speedL > 0) speedL = min(255, speedL + LEFT_MOTOR_TRIM);
  else if (speedL < 0) speedL = max(-255, speedL - LEFT_MOTOR_TRIM);

  if (speedR > 0) speedR = min(255, speedR + RIGHT_MOTOR_TRIM);
  else if (speedR < 0) speedR = max(-255, speedR - RIGHT_MOTOR_TRIM);

  if (speedL >= 0) { analogWrite(MOTOR_A_IN1, speedL); analogWrite(MOTOR_A_IN2, 0); } 
  else { analogWrite(MOTOR_A_IN1, 0); analogWrite(MOTOR_A_IN2, -speedL); }
  
  if (speedR >= 0) { analogWrite(MOTOR_B_IN1, speedR); analogWrite(MOTOR_B_IN2, 0); } 
  else { analogWrite(MOTOR_B_IN1, 0); analogWrite(MOTOR_B_IN2, -speedR); }
}

void setMotors(int speedL, int speedR) {
  targetSpeedL = speedL;
  targetSpeedR = speedR;
}

void stopMotors() { 
  setMotors(0, 0);
}

void updateMotorRamp() {
  if (millis() - lastMotorRampUpdate >= MOTOR_RAMP_INTERVAL_MS) {
    lastMotorRampUpdate = millis();
    if (currentSpeedL < targetSpeedL) currentSpeedL = min(targetSpeedL, currentSpeedL + MOTOR_RAMP_STEP);
    else if (currentSpeedL > targetSpeedL) currentSpeedL = max(targetSpeedL, currentSpeedL - MOTOR_RAMP_STEP);

    if (currentSpeedR < targetSpeedR) currentSpeedR = min(targetSpeedR, currentSpeedR + MOTOR_RAMP_STEP);
    else if (currentSpeedR > targetSpeedR) currentSpeedR = max(targetSpeedR, currentSpeedR - MOTOR_RAMP_STEP);

    setMotorsRaw(currentSpeedL, currentSpeedR);
  }
}

// =========================
// Audio & Mood
// =========================

void playProtectedAudio(int trackNum, unsigned long durationMs, unsigned long speechDelayMs) {
  myDFPlayer.playMp3Folder(trackNum);
  audioBusyUntil = millis() + durationMs; 
  audioSpeechStartTime = millis() + speechDelayMs; 
}

void setMood(RobotMood newMood, unsigned long durationMs) {
  currentMood = newMood;
  moodDecayTimer = millis() + durationMs;
  setMoodColor();
}

void updateMood() {
  if (moodDecayTimer > 0 && millis() > moodDecayTimer) {
    currentMood = MOOD_NEUTRAL;
    moodDecayTimer = 0;
    setMoodColor();
  }
}

RobotMood pickBiasedMood() {
  if (currentMood != MOOD_NEUTRAL && random(0, 2) == 0) {
    return currentMood;
  }
  int r = random(0, 3);
  if (r == 0) return MOOD_NEUTRAL;
  if (r == 1) return MOOD_HAPPY;
  return MOOD_PLAYFUL;
}

FaceState moodToFace(RobotMood m) {
  if (m == MOOD_HAPPY) return HAPPY;
  if (m == MOOD_PLAYFUL) return CURIOUS;
  if (m == MOOD_SHY) return NORMAL;
  return NORMAL;
}

// =========================
// Face Engine (with THINKING & SHOCKED)
// =========================

void drawFace(FaceState face, bool speaking, int eyeHeight) {
  if (!oledConnected) return; 
  display.clearDisplay();
  int cx = 64, cy = 32;

  switch (face) {
    case NORMAL:
      display.fillRoundRect(cx - 32, cy - (eyeHeight/2), 24, eyeHeight, 7, SH110X_WHITE);
      display.fillRoundRect(cx + 8, cy - (eyeHeight/2), 24, eyeHeight, 7, SH110X_WHITE);
      
      if (eyeHeight > 12) {
        display.fillRoundRect((cx - 24) + pupilOffsetX, (cy - 6) + pupilOffsetY, 8, 12, 3, SH110X_BLACK); 
        display.fillRoundRect((cx + 16) + pupilOffsetX, (cy - 6) + pupilOffsetY, 8, 12, 3, SH110X_BLACK); 
        display.fillRect(cx - 30, cy - (eyeHeight/2) + 3, 4, 4, SH110X_WHITE);
        display.fillRect(cx + 10, cy - (eyeHeight/2) + 3, 4, 4, SH110X_WHITE);
      }
      
      if (speaking) {
        int mouthOpen = 6 + (sin(millis() / 50.0) * 5);
        display.fillRoundRect(cx - 12, cy + 12, 24, mouthOpen, 4, SH110X_WHITE);
      } else {
        display.fillCircle(cx, cy + 10, 6, SH110X_WHITE);
        display.fillCircle(cx, cy + 8, 6, SH110X_BLACK); 
      }
      break;
      
    case DOWN: 
      display.fillRoundRect(cx - 32, cy - 10, 24, 20, 6, SH110X_WHITE);
      display.fillRoundRect(cx + 8, cy - 10, 24, 20, 6, SH110X_WHITE);
      display.fillTriangle(cx - 34, cy - 12, cx - 8, cy - 12, cx - 34, cy + 2, SH110X_BLACK);
      display.fillTriangle(cx + 34, cy - 12, cx + 8, cy - 12, cx + 34, cy + 2, SH110X_BLACK);
      
      if (speaking) display.fillRoundRect(cx - 12, cy + 12, 24, 10, 4, SH110X_WHITE);
      else display.fillRoundRect(cx - 10, cy + 15, 20, 3, 1, SH110X_WHITE);
      break;
      
    case SURPRISED: 
      display.fillRoundRect(cx - 30, cy - 16, 20, 28, 8, SH110X_WHITE);
      display.fillRoundRect(cx + 10, cy - 16, 20, 28, 8, SH110X_WHITE);
      display.fillCircle(cx - 20, cy - 2, 4, SH110X_BLACK);
      display.fillCircle(cx + 20, cy - 2, 4, SH110X_BLACK);
      display.drawLine(cx - 30, cy - 20, cx - 10, cy - 22, SH110X_WHITE);
      display.drawLine(cx + 10, cy - 22, cx + 30, cy - 20, SH110X_WHITE);

      if (speaking) {
        display.fillRoundRect(cx - 10, cy + 10, 20, 14, 6, SH110X_WHITE);
      } else {
        display.fillRoundRect(cx - 6, cy + 12, 12, 12, 5, SH110X_WHITE);
        display.fillRoundRect(cx - 3, cy + 14, 6, 8, 3, SH110X_BLACK);
      }
      break;

    case CURIOUS:
      display.fillRoundRect(cx - 32, cy - 14, 22, 26, 6, SH110X_WHITE);
      display.fillRoundRect(cx + 6, cy - 18, 26, 30, 8, SH110X_WHITE);
      display.fillRoundRect(cx - 24, cy - 4, 8, 10, 3, SH110X_BLACK);
      display.fillRoundRect(cx + 16, cy - 8, 10, 12, 3, SH110X_BLACK);
      display.drawLine(cx + 6, cy - 22, cx + 32, cy - 20, SH110X_WHITE);
      
      if (speaking) {
        display.fillRoundRect(cx - 10, cy + 12, 20, 10, 4, SH110X_WHITE);
      } else {
        display.drawLine(cx - 8, cy + 14, cx + 4, cy + 14, SH110X_WHITE);
        display.drawLine(cx + 4, cy + 14, cx + 12, cy + 10, SH110X_WHITE);
      }
      break;
      
    case PAIN: 
      display.fillTriangle(cx - 34, cy - 12, cx - 12, cy, cx - 34, cy + 12, SH110X_WHITE);
      display.fillTriangle(cx + 34, cy - 12, cx + 12, cy, cx + 34, cy + 12, SH110X_WHITE);
      
      if (speaking) {
        display.fillRoundRect(cx - 15, cy + 10, 30, 14, 4, SH110X_WHITE);
        display.drawLine(cx - 15, cy + 17, cx + 15, cy + 17, SH110X_BLACK);
      } else {
        display.drawLine(cx - 12, cy + 14, cx - 6, cy + 10, SH110X_WHITE);
        display.drawLine(cx - 6, cy + 10, cx + 6, cy + 18, SH110X_WHITE);
        display.drawLine(cx + 6, cy + 18, cx + 12, cy + 14, SH110X_WHITE);
      }
      break;
      
    case ANGRY: 
      display.fillRoundRect(cx - 32, cy - 12, 24, 24, 6, SH110X_WHITE);
      display.fillRoundRect(cx + 8, cy - 12, 24, 24, 6, SH110X_WHITE);
      display.fillTriangle(cx - 36, cy - 16, cx - 4, cy - 16, cx - 4, cy - 2, SH110X_BLACK);
      display.fillTriangle(cx + 36, cy - 16, cx + 4, cy - 16, cx + 4, cy - 2, SH110X_BLACK);
      display.fillRoundRect(cx - 20, cy - 4, 8, 8, 2, SH110X_BLACK);
      display.fillRoundRect(cx + 12, cy - 4, 8, 8, 2, SH110X_BLACK);

      if (speaking) display.fillRoundRect(cx - 15, cy + 14, 30, 10, 3, SH110X_WHITE);
      else {
        display.drawLine(cx - 10, cy + 18, cx + 10, cy + 14, SH110X_WHITE);
        display.drawLine(cx - 10, cy + 19, cx + 10, cy + 15, SH110X_WHITE);
      }
      break;
      
    case HAPPY: 
      display.fillCircle(cx - 22, cy - 6, 14, SH110X_WHITE);
      display.fillCircle(cx - 22, cy - 2, 14, SH110X_BLACK);
      display.fillCircle(cx + 22, cy - 6, 14, SH110X_WHITE);
      display.fillCircle(cx + 22, cy - 2, 14, SH110X_BLACK);
      
      if (speaking) {
        display.fillRoundRect(cx - 16, cy + 6, 32, 18, 8, SH110X_WHITE);
        display.fillRect(cx - 18, cy + 4, 36, 10, SH110X_BLACK);
      } else {
        display.fillCircle(cx, cy + 10, 8, SH110X_WHITE);
        display.fillCircle(cx, cy + 6, 8, SH110X_BLACK);
      }
      break;

    case SNEEZE: 
      display.drawLine(cx - 32, cy - 12, cx - 14, cy - 2, SH110X_WHITE);
      display.drawLine(cx - 32, cy - 11, cx - 14, cy - 1, SH110X_WHITE);
      display.drawLine(cx - 14, cy - 2, cx - 32, cy + 8, SH110X_WHITE);
      display.drawLine(cx - 14, cy - 1, cx - 32, cy + 9, SH110X_WHITE);

      display.drawLine(cx + 14, cy - 2, cx + 32, cy - 12, SH110X_WHITE);
      display.drawLine(cx + 14, cy - 1, cx + 32, cy - 11, SH110X_WHITE);
      display.drawLine(cx + 32, cy + 8, cx + 14, cy - 2, SH110X_WHITE);
      display.drawLine(cx + 32, cy + 9, cx + 14, cy - 1, SH110X_WHITE);

      if (speaking) {
        display.fillCircle(cx, cy + 12, 8, SH110X_WHITE);
      } else {
        display.fillCircle(cx, cy + 12, 4, SH110X_WHITE);
      }
      break;

    case SLEEP: 
      display.drawCircle(cx - 20, cy - 2, 12, SH110X_WHITE);
      display.fillRect(cx - 34, cy - 2, 28, 14, SH110X_BLACK);
      display.drawCircle(cx + 20, cy - 2, 12, SH110X_WHITE);
      display.fillRect(cx + 6, cy - 2, 28, 14, SH110X_BLACK);

      display.setCursor(cx + 32, cy - 22);
      display.print("z");
      display.setCursor(cx + 42, cy - 30);
      display.print("Z");

      display.fillCircle(cx, cy + 12, 3, SH110X_WHITE);
      break;

    case THINKING:
      display.fillRoundRect(cx - 32, cy - 14, 22, 26, 6, SH110X_WHITE);
      display.fillRoundRect(cx + 8, cy - 18, 20, 22, 6, SH110X_WHITE);
      display.fillRoundRect(cx - 24, cy - 4, 8, 10, 3, SH110X_BLACK);
      display.fillRoundRect(cx + 16, cy - 10, 6, 8, 2, SH110X_BLACK);
      display.drawLine(cx + 8, cy - 22, cx + 28, cy - 20, SH110X_WHITE);

      if (speaking) {
        display.fillRoundRect(cx - 10, cy + 12, 20, 10, 4, SH110X_WHITE);
      } else {
        display.drawLine(cx - 8, cy + 14, cx + 4, cy + 14, SH110X_WHITE);
        display.drawLine(cx + 4, cy + 14, cx + 12, cy + 10, SH110X_WHITE);
      }
      break;

    case SHOCKED:
      display.fillRoundRect(cx - 34, cy - 18, 24, 32, 10, SH110X_WHITE);
      display.fillRoundRect(cx + 10, cy - 18, 24, 32, 10, SH110X_WHITE);
      display.fillCircle(cx - 22, cy - 2, 6, SH110X_BLACK);
      display.fillCircle(cx + 22, cy - 2, 6, SH110X_BLACK);

      if (speaking) {
        display.fillRoundRect(cx - 12, cy + 8, 24, 16, 6, SH110X_WHITE);
      } else {
        display.fillRoundRect(cx - 8, cy + 10, 16, 12, 5, SH110X_WHITE);
        display.fillRoundRect(cx - 4, cy + 12, 8, 8, 3, SH110X_BLACK);
      }
      break;
  }
  display.display();
}

void setTemporaryFace(FaceState face, int durationMS) {
  currentFace = face;
  faceResetTime = millis() + durationMS;
}

// =========================
// LED & Breathing
// =========================

void updateBreathingLED() {
  if (currentMode != ACTION && currentFace == NORMAL && !isDancing && !isRunningScript) {
    float breath = (exp(sin(millis() / 2000.0 * PI)) - 0.36787944) * 108.0;
    int brightness = constrain(breath, BREATHING_MIN_BRIGHT, BREATHING_MAX_BRIGHT); 
    strip.setBrightness(brightness);
    strip.show();
  }
}

void resetRobotState() {
  isDancing = false;
  isEvading = false;
  isRunningScript = false;
  dancePhase = 0;
  currentFace = NORMAL;
  faceResetTime = 0;
  faceSequenceTime = 0;
  lastAutoActivity = millis();
  stopMotors();
  strip.setBrightness(40);
  strip.fill(DEFAULT_LED_COLOR);
  strip.show();
}

// =========================
// Web Handlers
// =========================

void handleRoot() { server.send(200, "text/html", INDEX_HTML); }

void handleStatus() {
  String modeStr = (currentMode == MANUAL) ? "Manual" : (currentMode == AUTO ? "Auto" : "Action");
  String moodStr = (currentMood == MOOD_NEUTRAL) ? "Neutral" :
                   (currentMood == MOOD_HAPPY) ? "Happy" :
                   (currentMood == MOOD_SHY) ? "Shy" : "Playful";

  String json = "{\"alert\":\"" + webAlertMsg + "\",";
  json += "\"irL\":" + String(liveLeftCliff) + ",";
  json += "\"irW\":" + String(liveWall) + ",";
  json += "\"irR\":" + String(liveRightCliff) + ",";
  json += "\"leftTh\":" + String(LEFT_CLIFF_THRESHOLD) + ",";
  json += "\"rightTh\":" + String(RIGHT_CLIFF_THRESHOLD) + ",";
  json += "\"wallTh\":" + String(WALL_DANGER_THRESHOLD) + ",";
  json += "\"mode\":\"" + modeStr + "\",";
  json += "\"mood\":\"" + moodStr + "\"}";
  server.send(200, "application/json", json);
}

void handleColor() {
  if (server.hasArg("hex")) {
    String hexStr = server.arg("hex");
    long color = strtol(hexStr.c_str(), NULL, 16);
    strip.fill(color);
    strip.show();
  }
  server.send(200, "text/plain", "OK");
}

void startStoryScript() {
  isRunningScript = true;
  currentScriptIndex = 0;
  scriptStartTime = millis();
  currentScriptStep = scriptStory[0];
  currentFace = currentScriptStep.face;
  if (currentScriptStep.soundTrack > 0) {
    playProtectedAudio(currentScriptStep.soundTrack, currentScriptStep.soundDur, 0);
  }
  setMotors(currentScriptStep.moveL, currentScriptStep.moveR);
  webAlertMsg = "Playing Mini Story";
}

void handleCmd() {
  lastAutoActivity = millis();

  // Mode switching (only thing that depends on action arg meaning "mode")
  if (server.hasArg("mode")) {
    String m = server.arg("mode");
    resetRobotState(); 
    if (m == "manual") { currentMode = MANUAL; playProtectedAudio(8, AUDIO_MODE_DUR, 0); }
    else if (m == "auto") { currentMode = AUTO; playProtectedAudio(9, AUDIO_MODE_DUR, 0); }
    else if (m == "action") { currentMode = ACTION; playProtectedAudio(10, AUDIO_MODE_DUR, 0); }
    webAlertMsg = "Mode: " + m;
    server.send(200, "text/plain", "OK");
    return;
  }

  // Manual drive (only in MANUAL mode)
  if (server.hasArg("move") && currentMode == MANUAL) {
    String move = server.arg("move");
    if (move == "fwd") setMotors(150, 150);
    else if (move == "bwd") setMotors(-150, -150);
    else if (move == "left") setMotors(-130, 130);
    else if (move == "right") setMotors(130, -130);
    else if (move == "stop") setMotors(0, 0);
    webAlertMsg = "Manual Control: " + move;
    server.send(200, "text/plain", "OK");
    return;
  }

  // Global action buttons (work in all modes)
  if (server.hasArg("action")) {
    String act = server.arg("action");

    if (act == "introFull") {
      resetRobotState();
      playProtectedAudio(3, AUDIO_FULL_INTRO_DUR, 0);
      currentFace = HAPPY;
      faceResetTime = millis() + DEFAULT_FACE_RESET_MS;
    }
    else if (act == "introShort") {
      resetRobotState();
      playProtectedAudio(2, AUDIO_SHORT_INTRO_DUR, 0);
      currentFace = HAPPY;
      faceResetTime = millis() + DEFAULT_FACE_RESET_MS;
    }
    else if (act == "chirp") triggerChirpBehavior();
    else if (act == "sneeze") triggerSneezeBehavior();
    else if (act == "sleep") triggerSleepBehavior();
    else if (act == "dance1") { isDancing = true; activeDanceType = 1; dancePhase = 0; playProtectedAudio(11, AUDIO_DANCE_DUR, 99999); }
    else if (act == "dance2") { isDancing = true; activeDanceType = 2; dancePhase = 0; playProtectedAudio(12, AUDIO_DANCE_DUR, 99999); }
    else if (act == "dance3") { isDancing = true; activeDanceType = 3; dancePhase = 0; playProtectedAudio(13, AUDIO_DANCE_DUR, 99999); }
    else if (act == "dialogue1") {
      currentFace = CURIOUS;
      faceResetTime = millis() + STORY_FACE_RESET_MS;
      playProtectedAudio(20, AUDIO_DIALOGUE_DUR, 0);
      webAlertMsg = "Dialogue 1";
    }
    else if (act == "dialogue2") {
      currentFace = CURIOUS;
      faceResetTime = millis() + STORY_FACE_RESET_MS;
      playProtectedAudio(21, AUDIO_DIALOGUE_DUR, 0);
      webAlertMsg = "Dialogue 2";
    }
    else if (act == "dialogue3") {
      currentFace = CURIOUS;
      faceResetTime = millis() + STORY_FACE_RESET_MS;
      playProtectedAudio(22, AUDIO_DIALOGUE_DUR, 0);
      webAlertMsg = "Dialogue 3";
    }
    else if (act == "story") {
      startStoryScript();
    }

    webAlertMsg = "Action: " + act;
  }

  server.send(200, "text/plain", "OK");
}

void handleThreshold() {
  if (server.hasArg("side") && server.hasArg("val")) {
    String side = server.arg("side");
    int val = server.arg("val").toInt();
    if (side == "left") {
      LEFT_CLIFF_THRESHOLD = val;
    } else if (side == "right") {
      RIGHT_CLIFF_THRESHOLD = val;
    } else if (side == "wall") {
      WALL_DANGER_THRESHOLD = val;
    }
    webAlertMsg = "Threshold updated: " + side + " = " + String(val);
  }
  server.send(200, "text/plain", "OK");
}

void handleCalibrateCliffs() {
  int leftReading = analogRead(PIN_IR_LEFT);
  int rightReading = analogRead(PIN_IR_RIGHT);
  LEFT_CLIFF_THRESHOLD = leftReading + CLIFF_MARGIN;
  RIGHT_CLIFF_THRESHOLD = rightReading + CLIFF_MARGIN;
  webAlertMsg = "Auto Calibrated: L=" + String(LEFT_CLIFF_THRESHOLD) + " R=" + String(RIGHT_CLIFF_THRESHOLD);
  server.send(200, "text/plain", "OK");
}

void calibrateCliffsAtBoot() {
  long sumL = 0, sumR = 0;
  const int samples = 50;
  for (int i = 0; i < samples; i++) {
    sumL += analogRead(PIN_IR_LEFT);
    sumR += analogRead(PIN_IR_RIGHT);
    delay(10);
  }
  int avgL = sumL / samples;
  int avgR = sumR / samples;
  LEFT_CLIFF_THRESHOLD = avgL + CLIFF_MARGIN;
  RIGHT_CLIFF_THRESHOLD = avgR + CLIFF_MARGIN;
}

// =========================
// Behaviors
// =========================

void triggerSneezeBehavior() {
  setMotors(-160, -160);
  delay(120);
  stopMotors();
  currentFace = SNEEZE;
  strip.fill(strip.Color(255, 100, 200));
  strip.show();
  playProtectedAudio(16, AUDIO_SNEEZE_DUR, 0);
  faceResetTime = millis() + SNEEZE_FACE_RESET_MS;
  webAlertMsg = "Achoo! Sneeze!";
}

void triggerSleepBehavior() {
  stopMotors();
  currentFace = SLEEP;
  strip.fill(strip.Color(10, 20, 80));
  strip.show();
  playProtectedAudio(17, AUDIO_SLEEP_DUR, 0);
  faceResetTime = millis() + SLEEP_FACE_RESET_MS;
  webAlertMsg = "Zzz... Sleeping...";
}

void triggerChirpBehavior() {
  currentFace = HAPPY;
  strip.fill(strip.Color(255, 255, 0));
  strip.show();
  playProtectedAudio(18, AUDIO_CHIRP_DUR, 0);
  faceResetTime = millis() + CHIRP_FACE_RESET_MS;
  webAlertMsg = "Chirp Chirp!";
}

// =========================
// Setup
// =========================

void setup() {
  pinMode(MOTOR_A_IN1, OUTPUT); digitalWrite(MOTOR_A_IN1, LOW);
  pinMode(MOTOR_A_IN2, OUTPUT); digitalWrite(MOTOR_A_IN2, LOW);
  pinMode(MOTOR_B_IN1, OUTPUT); digitalWrite(MOTOR_B_IN1, LOW);
  pinMode(MOTOR_B_IN2, OUTPUT); digitalWrite(MOTOR_B_IN2, LOW);
  
  Serial.begin(115200);
  delay(100); 
  
  pinMode(PIN_TOUCH, INPUT);

  strip.begin();
  strip.setBrightness(40);
  strip.fill(DEFAULT_LED_COLOR);
  strip.show();

  Wire.begin(OLED_SDA, OLED_SCL);
  if(display.begin(0x3C, true)) {
    oledConnected = true;
    display.clearDisplay();
    display.setTextSize(1);
    display.setTextColor(SH110X_WHITE);
    display.setCursor(0, 0);
    display.println("1. OLED OK");
    display.display();
  } else { oledConnected = false; }

  if (oledConnected) { display.println("2. Init Audio..."); display.display(); }
  FPSerial.begin(9600, SERIAL_8N1, DFPLAYER_RX, DFPLAYER_TX);
  delay(1500);
  
  if (myDFPlayer.begin(FPSerial, false)) { 
    if (oledConnected) { display.println("   Audio OK!"); display.display(); }
    myDFPlayer.volume(22); 
    playProtectedAudio(1, AUDIO_BOOT_DUR, 0);
  } else {
    if (oledConnected) { display.println("   Audio FAILED!"); display.display(); }
  }

  delay(1000);
  if (oledConnected) { display.println("3. Init WiFi..."); display.display(); }

  WiFi.mode(WIFI_AP); 
  esp_wifi_set_max_tx_power(WIFI_POWER_8_5dBm); 
  WiFi.softAP("Mikey_Robot", "12345678"); 
  
  if (oledConnected) { display.println("   WiFi OK!"); display.display(); }
  delay(1500);

  calibrateCliffsAtBoot();

  server.on("/", handleRoot);
  server.on("/cmd", handleCmd);
  server.on("/status", handleStatus);
  server.on("/color", handleColor);
  server.on("/threshold", handleThreshold);
  server.on("/calibrateCliffs", handleCalibrateCliffs);
  server.begin();

  lastAutoActivity = millis();
  lastMotorRampUpdate = millis();
}

// =========================
// Loop
// =========================

void loop() {
  server.handleClient();
  
  updateBreathingLED();
  updateMood();
  updateMotorRamp();

  // --- Script Player (Mini Story) ---
  if (isRunningScript) {
    unsigned long stepElapsed = millis() - scriptStartTime;
    if (stepElapsed >= currentScriptStep.stepDur) {
      currentScriptIndex++;
      if (currentScriptIndex >= SCRIPT_STORY_LEN) {
        // End of script
        isRunningScript = false;
        stopMotors();
        currentFace = NORMAL;
        faceResetTime = 0;
        webAlertMsg = "Story Finished";
      } else {
        // Next step
        scriptStartTime = millis();
        currentScriptStep = scriptStory[currentScriptIndex];
        currentFace = currentScriptStep.face;
        if (currentScriptStep.soundTrack > 0) {
          playProtectedAudio(currentScriptStep.soundTrack, currentScriptStep.soundDur, 0);
        }
        setMotors(currentScriptStep.moveL, currentScriptStep.moveR);
      }
    }
  }

  // --- Speaking detection ---
  bool shouldBeSpeaking = false;
  if (millis() < audioBusyUntil && millis() >= audioSpeechStartTime) {
    shouldBeSpeaking = ((millis() / 150) % 2 == 0);
  }

  // --- Saccades & Blinking ---
  if (millis() - lastEyeDart > random(500, 2500)) {
    lastEyeDart = millis();
    if (currentFace == NORMAL || currentFace == CURIOUS || currentFace == THINKING) {
      pupilOffsetX = random(-4, 5); 
      pupilOffsetY = random(-2, 3);
    } else {
      pupilOffsetX = 0; pupilOffsetY = 0;
    }
  }

  if (!isBlinking && (millis() - lastBlinkTime > nextBlinkDelay)) {
    isBlinking = true;
    blinkStartTime = millis();
  }
  
  int currentEyeHeight = 26; 
  if (isBlinking) {
    long blinkProgress = millis() - blinkStartTime;
    if (blinkProgress < 50) currentEyeHeight = 26 - (blinkProgress / 2); 
    else if (blinkProgress < 100) currentEyeHeight = 2; 
    else if (blinkProgress < 150) currentEyeHeight = 2 + ((blinkProgress - 100) / 2); 
    else {
      isBlinking = false;
      currentEyeHeight = 26;
      lastBlinkTime = millis();
      nextBlinkDelay = random(2000, 6000); 
    }
  }

  // --- Emotion sequence logic ---
  if (faceSequenceTime > 0 && millis() > faceSequenceTime) {
    currentFace = faceSequenceNext; 
    faceSequenceTime = 0;
  }

  if (faceResetTime > 0 && millis() > faceResetTime) {
    currentFace = NORMAL;
    faceResetTime = 0;
    if (!isDancing && !isRunningScript) {
      webAlertMsg = "All Clear";
      strip.fill(getMoodColor());
      strip.show();
    }
  }

  // --- Smart screen refresh ---
  if (currentFace != lastDrawnFace || shouldBeSpeaking != lastDrawnSpeaking || currentEyeHeight != lastDrawnEyeHeight || pupilOffsetX != 0) {
    drawFace(currentFace, shouldBeSpeaking, currentEyeHeight);
    lastDrawnFace = currentFace;
    lastDrawnSpeaking = shouldBeSpeaking;
    lastDrawnEyeHeight = currentEyeHeight;
  }

  // --- Sensor Safety Overrides ---
  if (millis() - lastSensorCheck > 50) {
    lastSensorCheck = millis();
    liveLeftCliff  = analogRead(PIN_IR_LEFT);
    liveWall       = analogRead(PIN_IR_WALL);
    liveRightCliff = analogRead(PIN_IR_RIGHT);

    bool cliffDanger = (liveLeftCliff > LEFT_CLIFF_THRESHOLD || liveRightCliff > RIGHT_CLIFF_THRESHOLD);
    bool wallDanger  = (liveWall < WALL_DANGER_THRESHOLD);

    if ((cliffDanger || wallDanger) && currentMode == AUTO && !isEvading && !isRunningScript) {
      stopMotors();
      isEvading = true;
      evasionState = 1;
      evasionTimer = millis();
      lastAutoActivity = millis();
      if (cliffDanger) {
        setTemporaryFace(DOWN, 4000);
        playProtectedAudio(5, AUDIO_CLIFF_ALERT_DUR, 0); 
        webAlertMsg = "ALERT: CLIFF EDGE!";
      } else {
        setTemporaryFace(SURPRISED, 4000);
        playProtectedAudio(4, AUDIO_WALL_ALERT_DUR, 0); 
        webAlertMsg = "ALERT: WALL CLOSE!";
        faceSequenceNext = THINKING;
        faceSequenceTime = millis() + 4000;
        playProtectedAudio(23, AUDIO_DIALOGUE_DUR, 0);
      }
    }

    if ((cliffDanger || wallDanger) && currentMode == MANUAL) {
      if (!dangerAlertActive) {
        dangerAlertActive = true;
        stopMotors();
        if (cliffDanger) {
          setTemporaryFace(DOWN, 3000);
          playProtectedAudio(5, AUDIO_CLIFF_ALERT_DUR, 0);
          webAlertMsg = "ALERT: CLIFF EDGE!";
        } else {
          setTemporaryFace(SURPRISED, 3000);
          playProtectedAudio(4, AUDIO_WALL_ALERT_DUR, 0);
          webAlertMsg = "ALERT: WALL CLOSE!";
          playProtectedAudio(23, AUDIO_DIALOGUE_DUR, 0);
        }
      }
    } else if (!cliffDanger && !wallDanger) {
      dangerAlertActive = false;
    }
  }

  // --- Auto Evasion Maneuver ---
  if (isEvading && currentMode == AUTO) {
    if (evasionState == 1 && millis() - evasionTimer > 1200) {
      setMotors(-130, -130);
      evasionTimer = millis();
      evasionState = 2;
    } else if (evasionState == 2 && millis() - evasionTimer > 800) {
      setMotors(140, -140);
      evasionTimer = millis();
      evasionState = 3;
    } else if (evasionState == 3 && millis() - evasionTimer > 600) {
      stopMotors();
      isEvading = false;
      evasionState = 0;
      lastAutoMove = millis();
    }
  }

  // --- Advanced Touch Sequences ---
  bool touchState = digitalRead(PIN_TOUCH);
  if (touchState) lastAutoActivity = millis();
  
  if (touchState && !lastTouchState) {
    touchStartTime = millis(); 
    isTouching = true;
    isPetting = false; 
  }
  
  if (touchState && isTouching && !isPetting && (millis() - touchStartTime > PET_MIN_DURATION_MS)) {
    isPetting = true; 
    currentFace = HAPPY;
    faceResetTime = millis() + PET_FACE_RESET_MS;
    playProtectedAudio(7, AUDIO_PET_DUR, 1500); 
    webAlertMsg = "Mikey is Happy!";
    setMood(MOOD_HAPPY, MOOD_DECAY_MS);
    lastAutoNoise = millis(); 
  }
  
  if (!touchState && lastTouchState) {
    if (isTouching && !isPetting) {
      currentFace = PAIN; 
      faceSequenceTime = millis() + TAP_FACE_TIME_MS; 
      faceSequenceNext = ANGRY;              
            
      faceResetTime = millis() + TAP_MOOD_TIME_MS;
      playProtectedAudio(6, AUDIO_TAP_DUR, 0); 
      webAlertMsg = "Ouch! Tapped!";
      setMood(MOOD_SHY, MOOD_DECAY_MS);
      lastAutoNoise = millis();
    }
    isTouching = false; 
    isPetting = false;
  }
  lastTouchState = touchState;

  // --- AUTO Mode Roaming & Idle Personality ---
  if (currentMode == AUTO && currentFace == NORMAL && !isDancing && !isEvading && !isRunningScript) {
    if (millis() - lastAutoActivity > AUTO_IDLE_TIMEOUT_MS) {
      lastAutoActivity = millis();
      if (random(0, 2) == 0) triggerSneezeBehavior();
      else triggerSleepBehavior();
    } else {
      // Normal roaming with arc bias
      if (millis() - lastAutoMove > 1200) {
        lastAutoMove = millis();
        int r = random(0, 5);
        if (r == 0) {
          setMotors(110, 110); // forward
        } else if (r == 1) {
          setMotors(100, 130); // gentle right arc
        } else if (r == 2) {
          setMotors(130, 100); // gentle left arc
        } else if (r == 3) {
          setMotors(-110, 110); // left turn
        } else {
          setMotors(110, -110); // right turn
        }
      }
      
      // Idle "look around" + soft noise
      if (millis() - lastAutoNoise > AUTO_NOISE_INTERVAL_MS && millis() > audioBusyUntil) {
        lastAutoNoise = millis();
        RobotMood biased = pickBiasedMood();
        FaceState suggested = moodToFace(biased);

        int rIdle = random(0, IDLE_LOOK_AROUND_CHANCE);
        if (rIdle == 0) {
          setMotors(60, 90);
          setTemporaryFace(CURIOUS, 2000);
          playProtectedAudio(14, AUDIO_STORY_PART_DUR, 0); // curious sound
          setMood(MOOD_PLAYFUL, MOOD_DECAY_MS);
        } else {
          int rAudio = random(14, 16); 
          if (rAudio == 14) {
            setTemporaryFace(CURIOUS, 3000);
            playProtectedAudio(14, AUDIO_STORY_PART_DUR, 0);
          } else {
            setTemporaryFace(HAPPY, 3000);
            playProtectedAudio(15, AUDIO_STORY_PART_DUR, 0);
          }
          if (random(0, 3) == 0) {
            currentFace = suggested;
            faceResetTime = millis() + 3000;
          }
        }
      }
    }
  }

  // --- Dance Routines ---
  if (isDancing) {
    if (millis() > nextDanceMove) {
      dancePhase = (dancePhase + 1) % 8;
      
      // Flash Neon Stage LED
      strip.fill(strip.Color(random(50, 255), random(50, 255), random(50, 255)));
      strip.show();

      if (activeDanceType == 1) {
        switch (dancePhase) {
          case 0: setMotors(160, 160); currentFace = HAPPY; nextDanceMove = millis() + 350; break;
          case 1: setMotors(-150, -150); currentFace = SURPRISED; nextDanceMove = millis() + 300; break;
          case 2: setMotors(180, -180); currentFace = HAPPY; nextDanceMove = millis() + 500; break;
          case 3: setMotors(-180, 180); currentFace = CURIOUS; nextDanceMove = millis() + 500; break;
          case 4: setMotors(150, 50); currentFace = HAPPY; nextDanceMove = millis() + 400; break;
          case 5: setMotors(50, 150); currentFace = NORMAL; nextDanceMove = millis() + 400; break;
          case 6: setMotors(-160, -160); currentFace = SURPRISED; nextDanceMove = millis() + 300; break;
          case 7: stopMotors(); currentFace = HAPPY; nextDanceMove = millis() + 250; break;
        }
      } else if (activeDanceType == 2) {
        switch (dancePhase) {
          case 0: setMotors(-140, -140); currentFace = CURIOUS; nextDanceMove = millis() + 450; break;
          case 1: stopMotors(); currentFace = NORMAL; nextDanceMove = millis() + 150; break;
          case 2: setMotors(-140, -140); currentFace = CURIOUS; nextDanceMove = millis() + 450; break;
          case 3: setMotors(170, 70); currentFace = HAPPY; nextDanceMove = millis() + 500; break;
          case 4: setMotors(70, 170); currentFace = HAPPY; nextDanceMove = millis() + 500; break;
          case 5: setMotors(160, -160); currentFace = SURPRISED; nextDanceMove = millis() + 350; break;
          case 6: setMotors(-160, 160); currentFace = HAPPY; nextDanceMove = millis() + 350; break;
          case 7: stopMotors(); currentFace = NORMAL; nextDanceMove = millis() + 200; break;
        }
      } else if (activeDanceType == 3) {
        switch (dancePhase) {
          case 0: setMotors(170, -170); currentFace = HAPPY; nextDanceMove = millis() + 250; break;
          case 1: setMotors(-170, 170); currentFace = SURPRISED; nextDanceMove = millis() + 250; break;
          case 2: setMotors(170, -170); currentFace = HAPPY; nextDanceMove = millis() + 250; break;
          case 3: setMotors(150, 150); currentFace = HAPPY; nextDanceMove = millis() + 350; break;
          case 4: setMotors(-150, -150); currentFace = CURIOUS; nextDanceMove = millis() + 350; break;
          case 5: setMotors(180, -180); currentFace = HAPPY; nextDanceMove = millis() + 600; break;
          case 6: setMotors(-180, 180); currentFace = SURPRISED; nextDanceMove = millis() + 600; break;
          case 7: stopMotors(); currentFace = HAPPY; nextDanceMove = millis() + 300; break;
        }
      }
    }
  }
}
