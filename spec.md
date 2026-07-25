# NXP Cup India 2026 - Competition Specification

## 1. Overview

**Challenge Name**: Autonomous Medical Response  
**Objective**: Design and develop a Mobile Robotics Prototype (MR-Buggy3) for autonomous emergency vehicle driving algorithm using camera and lidar inputs.

---

## 2. Eligibility & Awards

### Eligibility
- Students of 18-26 years as on 1st August 2026
- Pursuing UG/PG in Engineering from registered Indian colleges
- Team Size: 1 to 4 members

### Awards
| Place | Prize |
|-------|-------|
| 1st Grand Finale | ₹1,00,000 |
| 2nd Grand Finale | ₹75,000 |
| 3rd Grand Finale | ₹50,000 |
| AI Ayush Award | ₹10,000 |
| Innovation Edge Award | ₹10,000 |
| Precision Excellence Award | ₹10,000 |
| Regional Finale Winner | ₹10,000 |

---

## 3. Timeline

| # | Phase | Dates | Description |
|---|-------|-------|-------------|
| 1 | Registration | 10 Jun – 10 Jul | Enroll, eligibility checks, qualify team |
| 2 | Virtual Training #1 | 17 Jul – 23 Jul | Gazebo simulator, ROS, AI/ML from OpenCV, Sensor data processing |
| 3 | Model Creation | 24 Jul – 4 Aug | Design autonomous rover driving algorithm using camera and lidar |
| 4 | Pre-Regional Qualifier | 7 Aug – 12 Aug | Selection via pre-recorded videos and online evaluation calls |
| 5 | Regional Finale | 18 Aug – 27 Aug | In-person across India, Gazebo simulator tasks, top teams advance |
| 6 | Virtual Training #2 | 1 Sep – 7 Sep | MR-Buggy3 HW kit, Software deployment on hardware |
| 7 | Grand Finale | Nov | At NXP India location, real-world environment |

---

## 4. Hardware Platforms

### NAVQ+ (Mission Computer)
- **MPU**: iMX8MPLUS, 8GB LPDDR4, NPU 2.3 TOPS Accelerator
- **Features**: Low power ML, vision, path planning, navigation
- **ML Framework**: eIQ (TFLite, ArmNN, ONNX)
- **Interfaces**: Dual CAN, WiFi 5, BT 5.0, 2x USB-C, Dual MIPI-CSI cameras, HDMI, LVDS
- **Security**: SE050 Edge Lock Secure element with NFC
- **Software**: Yocto Linux, Ubuntu, ROS2, Python, eIQ

### CANHUBK3 (VMU)
- **MCU**: S32K344 Automotive General-Purpose Microcontroller
- **Interfaces**: 6 CAN FD ports, 100BASE-T1 Ethernet, UART, SPI, I2C, GPIO, PWM
- **Security**: SE050 Edge Lock (Plug and Trust Secure Element)
- **Software**: Zephyr RTOS, NuttX/PX4, S32 Design Studio

### CogniPilot (Autopilot Platform)
- **License**: Apache 2 (Open-source)
- **Components**:
  - **Cranium**: ROS2 workspace for high-level computations
  - **Cerebri**: IMU/actuator interface, runs on Zephyr RTOS 3.5
  - **Synapse**: Communication layer

---

## 5. Challenge Environment

### Simulation World
- Smart city with roads, intersections, buildings, sign boards, obstacles
- Dedicated lane network with white roads and black line borders
- Buggy must follow lane discipline at all times

### Buildings
| Type | Buildings |
|------|-----------|
| Patient | PATIENT_1, PATIENT_2, PATIENT_3 |
| Hospital | HOSPITAL_1, HOSPITAL_2, HOSPITAL_3 |
| Fake Hospital | FAKE_HOSPITAL_1, FAKE_HOSPITAL_2 (penalty zones) |

### Sign Board Mapping
| Sign | Building | Direction |
|------|----------|-----------|
| A | PATIENT_1 | Left/Straight/Right |
| B | PATIENT_2 | Left/Straight/Right |
| C | PATIENT_3 | Left/Straight/Right |
| X | HOSPITAL_1 | Left/Straight/Right |
| Y | HOSPITAL_2 | Left/Straight/Right |
| Z | HOSPITAL_3 | Left/Straight/Right |

### QR Codes
- Each building has a unique QR code
- Patient QR format: `{LOC: PATIENT_1}`
- Must detect and decode using onboard camera

---

## 6. Mission Flow

```
Start Buggy
    ↓
Navigate City
    ↓
Follow Lane Discipline
    ↓
Find Patient 1
    ↓
Scan QR
    ↓
Contact Municipality Server (inside Patient Zone)
    ↓
Receive Hospital Assignment
    ↓
Reach Correct Hospital
    ↓
Acknowledge Server (inside Hospital Zone)
    ↓
Get Next Patient
    ↓
Repeat for All Patients (3 total)
    ↓
Drop Last Patient
    ↓
Mission Complete
    ↓
Bonus Mission: Exit Lane → Park Vehicle → Stop
```

---

## 7. Scoring System

### Positive Points
| Category | Points |
|----------|--------|
| Patient Identification | +ve for each correctly decoded patient QR |
| Municipality Communication | +ve for each successful patient registration |
| Hospital Delivery | +ve for correct hospital assignment delivery |
| Mission Completion Bonus | +ve for completing all 3 patients |
| Time-Based Ranking | +ve percentile based on completion time |
| Parking Bonus | +ve for autonomous parking in designated zone |

### Negative Points (Penalties)
| Category | Penalty |
|----------|---------|
| Collision | -ve for every collision with obstacles/infrastructure |
| Lane Jump | -ve for crossing/touching black lines |
| Incorrect Delivery | -ve for wrong hospital delivery |
| Fake Hospital | -ve for attempting delivery at FAKE_HOSPITAL_1/2 |
| Patient Zone Exit | -ve for leaving patient zone without hospital assignment |
| Hospital Zone Miss | -ve for sending delivery outside hospital zone |

### Ranking
- Winner = Highest final score
- Tiebreaker = Lower mission completion time

---

## 8. Software Stack

### Requirements
- **OS**: Ubuntu 22.04.5 (fresh install recommended)
- **Framework**: CogniPilot AIRY release
- **Simulator**: Gazebo Harmonic
- **ROS**: ROS2 Humble

### Allowed Python Libraries
```
torch==2.3.0
torchvision==0.18.0
numpy==1.26.4
opencv-python==4.11.0.86
scipy==1.15.1
scikit-learn==1.5.2
tk==0.1.0
pyzbar==0.1.9
matplotlib==3.5.1
pyyaml==6.0.2
tflite-runtime==2.14.0
```

### ROS2 Package: `b3rb_ros_line_follower`

#### Node Architecture
| Node | Executable | File | Purpose |
|------|------------|------|---------|
| `/edge_vectors_publisher` | `vectors` | `b3rb_ros_edge_vectors.py` | Extract lane edge vectors from camera |
| `/line_follower` | `runner` | `b3rb_ros_line_follower.py` | Core controller (steering, planning, server comm) |
| `/object_recognizer` | `detect` | `b3rb_ros_object_recog.py` | Classify traffic signs |
| `/qr_detector` | `qr_detect` | `b3rb_ros_qr_detector.py` | Scan/decode QR codes |

#### ROS Topics
| Topic | Message Type | Direction | Purpose |
|-------|--------------|-----------|---------|
| `/camera/image_raw/compressed` | `sensor_msgs/msg/CompressedImage` | Sub | Camera feed |
| `/scan` | `sensor_msgs/msg/LaserScan` | Sub | LIDAR data |
| `/edge_vectors` | `synapse_msgs/msg/EdgeVectors` | Sub | Lane boundaries |
| `/sign_board_detection` | `std_msgs/msg/String` | Sub | Detected signs |
| `/qr_detection` | `std_msgs/msg/String` | Sub | Scanned QR codes |
| `/ServerCommunication` | `synapse_msgs/msg/ServerCommunication` | Bidirectional | Server interface |
| `/cerebri/in/joy` | `sensor_msgs/msg/Joy` | Pub | Steering/speed commands |

#### Message Formats

**Joy (Steering/Driving)**
- `msg.axes[1]`: Speed [-1.0, 1.0] (positive = forward)
- `msg.axes[3]`: Steering [-1.0, 1.0] (positive = left)
- `msg.buttons`: `[1, 0, 0, 0, 0, 0, 0, 1]` to enable manual override

**EdgeVectors**
- `image_height`: Camera frame height (default 240)
- `image_width`: Camera frame width (default 320)
- `vector_count`: Number of boundaries (0, 1, or 2)
- `vector_1`: First boundary (Left)
- `vector_2`: Second boundary (Right)

**ServerCommunication**
```
uint8 src      # Sender (Buggy=1, Server=2)
uint8 dest     # Recipient (Buggy=1, Server=2)
uint8 uid      # Message ID counter
uint8 ack      # Acknowledgment (0=blank, 1=acknowledged)
string msg     # Payload (QR text, "PARKED", "INVALID", etc.)
```

---

## 9. Server Communication Protocol

### Handshake Sequence
```
Buggy (ID: 1)                              Server (ID: 2)
    |                                           |
    | -- [src=1, dest=2, uid=10, msg="A"] --> |  (1. Scanned QR Patient)
    | <-- [src=2, dest=1, uid=10, ack=1] ---- |  (2. Quick acknowledgment)
    |                                           |
    |               [ Brief Validation Delay ]  |
    |                                           |
    | <-- [src=2, dest=1, uid=101, msg="X"] -- |  (3. Next Target - Hospital)
    | -- [src=1, dest=2, uid=101, ack=1] ----> |  (4. Target Confirmed)
```

### Message Flow
1. **Patient/Hospital Arrival**: Send scanned QR text to server (must be inside zone)
2. **Server Destination Update**: Receive next target, send ACK
3. **Bonus Parking**: Send `msg = "PARKED"`, wait for `msg = "OK"` or `msg = "INVALID"`

### Important Notes
- Malformed packets are ignored by server
- QR text must match target AND be inside valid zone
- Server retries up to 5 times if no ACK received
- No response after 5 retries = "Connection Lost" state

---

## 10. Technical Implementation Hints

### Lane Following
- Convert to HSV space and threshold for black lane markings
- Calculate offset: `midpoint = (left_x + right_x) / 2.0`
- Adjust steering proportional to offset from center (160.0)

### Obstacle Avoidance
- Map LIDAR range array to find front-facing indices
- Override lane steering if minimum front distance < threshold
- Merge back once scan is clear

### Sign Board Detection
- Use color or shape to isolate sign boards
- Use ROI (regions of interest) for multiple signs in frame
- Develop ML model to classify sign type

### QR Code Detection
- Use `cv2.QRCodeDetector()` or `pyzbar`
- Compare scanned string to server-provided target
- Use LIDAR side scans to check range when parking

### Parking
- Use LIDAR to align buggy
- Check scan ranges to center inside parking lines
- Send "PARKED" message when inside zone

---

## 11. Debugging Tool

**NXP CUP Debugging Tool** (`buggy-control-panel.deb`)

### Installation
```bash
cd ~/cognipilot/cranium/NXP_CUP_INDIA_2026/
sudo dpkg -i buggy-control-panel.deb
sudo apt-get install -f  # if dependencies missing
sudo dpkg -i buggy-control-panel.deb
```

### Usage
```bash
buggy-control-panel
# Access at http://localhost:8888/
```

### Features
- ROS Topic Selector dropdowns
- Camera stream visualization
- LiDAR map rendering (center = buggy, colored dots = obstacles)
- Data terminals for JSON/text messages
- Server Communication Simulator for testing

---

## 12. Submission Rules

1. **Evaluation Platform**: NXP laptop with default CogniPilot setup
2. **No additional package installation** allowed without written consent
3. **Submit only**: `b3rb_ros_line_follower` folder
4. **File format**: `NXP_CUP26_<team_id>.zip`
   - Example: Team ID 3124 → `NXP_CUP26_3124.zip`
   - Inside: `NXP_CUP26_3124/b3rb_ros_line_follower/`
5. **Video submission** may be required if participants exceed threshold
6. Upload location communicated via MS Teams Channel

---

## 13. Key Resources

| Resource | Link |
|----------|------|
| GitHub Repo | https://github.com/NXP-Robotics/NXP_CUP_INDIA_2026 |
| CogniPilot Docs | https://airy.cognipilot.org/ |
| MR-B3RB Hardware | https://nxp.gitbook.io/mr-b3rb |
| Gazebo Simulator | https://gazebosim.org/home |
| Official Email | nxp-cup.india@nxp.com |
