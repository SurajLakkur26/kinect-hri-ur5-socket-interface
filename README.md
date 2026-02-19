# Kinect-Based Human–Robot Interaction via Socket Interface (UR5)

This repository contains a C++ implementation of a **Kinect-based gesture and posture tracking pipeline** that streams skeletal joint positions and joint angles over a TCP socket for human–robot interaction (HRI). The current implementation targets a **UR5 robot** as the client, with **Kinova robot integration planned only as future work** (not implemented in this project).

## 1. Project Overview

The project uses a **Kinect for Xbox 360 / Kinect for Windows** sensor to track a human user's body in 3D and extract:

- Cartesian joint positions in camera space
- Joint angles for shoulders, elbows, and spine

These values are formatted as a linear array and streamed via **TCP/IP** to a client (UR5 controller, SocketTest 3.0, or Python script).

**Data flow**: Kinect → C++ server → TCP socket on port 8080 → Client interprets array for robot motions.

## 2. Functionality

The main file `Socket_Test` performs:

- Initializes **Winsock** TCP server on port `8080`
- Kinect sensor initialization via `IKinectSensor`/`IBodyFrameReader`
- Continuous body frame processing:
  - Extracts positions: spine base, left/right hand
  - Computes angles via `CalculateAngle(...)`: right/left shoulder/elbow, spine
  - Scales positions to millimeters
  - Packs into comma-separated string
  - Sends on demand (triggers like "send" command)
  - Rate control via `Sleep(3000)`

## 3. Architecture

1. **Kinect sensor**: Provides `CameraSpacePoint` joints via SDK 2.0
2. **C++ server**: Computes angles, packs data into linear array
3. **Client side**:
   - **SocketTest 3.0**: Testing/development
   - **UR5 robot**: Maps array elements to motions (current HRI)
   - **Future Kinova**: Python client uses indices 10/11 (shoulder angles)

## 4. Source Code

- `Source_Code/Socket_Test`: Kinect init, angle calculation, TCP server, streaming loop, test sections
- `Source_Code/Kinova_Python`: Kinect init, angle calculation, TCP server, streaming loop, test sections, but for Python

## 5. Requirements

**Software**:
- Windows with **Kinect for Windows SDK 2.0**
- **Visual Studio 2019+** with C++ support
- **SocketTest 3.0** (optional, for TCP testing)

**Libraries**:
```cpp
#pragma comment(lib, "ws2_32.lib")
#pragma comment(lib, "kinect20.lib")
```

## 6. Quickstart

### 6.1. Build
1. Open Visual Studio → New Win32 Console Application
2. Add `src/Final.cpp` to project
3. Ensure Kinect SDK paths configured
4. Build (Debug or Release)

### 6.2. Run Kinect Server
1. Connect and power Kinect sensor
2. Run compiled executable
3. Console shows: "Server started on port 8080"

### 6.3. Test with SocketTest 3.0
1. Open SocketTest 3.0 (TCP Client mode)
2. IP: Kinect PC address (127.0.0.1 for local)
3. Port: 8080 → Connect
4. Receive welcome message + data lines
5. Send "send" to trigger data

### 6.4. UR5 Client (Production)
UR5 TCP client connects to port 8080, reads comma-separated array, maps values to robot motions.

## 7. Future Work

**Kinova Integration** (not implemented):
- Python client reads array indices 10/11 (shoulder angles)
- Maps to Kinova pick-and-place motions
- Recommended: separate repo or `client/` directory

## 8. Safety Notes

- Test with **slow motions** and **small ranges** first
- Keep **emergency stop** accessible
- Ensure **clear workspace** around robot
- For educational/experimental use in controlled environments only
