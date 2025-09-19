
# Automatic Smart Parking System

Smart, AI-powered, RFID-based parking management system with real-time license plate recognition, barrier control, and full audit trail. Built with Qt6, GStreamer, Tesseract OCR, YOLO ONNX, and SQLite. 

# Features
This is a production-ready automatic parking system designed for small to medium parking lots. It supports:

•	Dual-lane operation (Entrance + Exit, or configurable as All-Entrance/All-Exit)

•	RFID card scanning (HID keyboard-emulation readers on Windows)

•	Dual-camera ALPR (Automatic License Plate Recognition) — front + rear plates

•	AI-powered OCR using YOLOv8 (ONNX) + Tesseract

•	Hardware control via USB Relay for barrier gates

•	Real-time RTSP video streaming via GStreamer

•	Full session lifecycle with fee calculation, subscriptions, penalties, reporting

•	Admin UI for user/subscription/RFID/pricing management

•	Auto-recovery on network/camera failure

•	Low-latency pipeline optimized for real-time operation



# Layer Architechture

                                            Presentation (QML UI)
                                                     ↑
                                        Application (ParkingController)
                                                     ↑
                                        Domain (Interfaces + Models)
                                                     ↑
                                Infrastructure (DB, Camera, OCR, Barrier, Card Reader)

#  Technology Stack
•	OS: Windows 10/11 (x64)

•	Compiler: MSVC 2022 (Visual Studio) on Windows

•	Qt: Qt 6.9.2 MSVC 64-bit

•	GStreamer: 1.22.x MSVC x64 Runtime + Development

•	Tesseract OCR: via vcpkg or prebuilt binaries

•	ONNX Runtime: 1.20.0 Win-x64

•	Cameras: RTSP-compatible IP cameras (H.264, low-latency mode)

•	Hardware: USB Relay Barrier Controller (COM port), HID RFID Readers

# Installation and Setup
1.	Visual Studio Build Tools 2022:
•	Download from the [official website](https://visualstudio.microsoft.com/free-developer-offers/).

•	During installation, select the "Desktop development with C++" workload.

•	Ensure "MSVC v143 - VS 2022 C++ x64/x86 build tools" and a Windows SDK are selected.

2. Qt 6:
Download on [Qt Website](https://www.qt.io/download-open-source).
Use the Qt Maintenance Tool to install a Qt 6 version for MSVC 2022 64-bit.

4. GStreamer
•	Download and install both the Runtime and Development MSVC 64-bit packages from the [GStreamer website](https://gstreamer.freedesktop.org/download).

•	Install both to the same directory.

•	Add the GStreamer bin directory to your system's PATH environment variable.

•	Add the GStreamer directory 

    lib\pkgconfig 
to your PKG_CONFIG_PATH environment variable.

4.	Tesseract OCR:
•	The recommended way is to install via vcpkg: 

    vcpkg install tesseract:x64-windows leptonica:x64-windows.

•	Ensure you have the required language data files (e.g., eng.traineddata, vie.traineddata) in a tessdata directory.

5.	ONNX Runtime:
•	Download the latest pre-built binary from the [ONNX Runtime GitHub releases](https://github.com/microsoft/onnxruntime/releases).

•	Extract it and place it in the lib directory of the project.

6. Configure CMake

        cmake -B build -S . \
        -DCMAKE_TOOLCHAIN_FILE=C:/vcpkg/scripts/buildsystems/vcpkg.cmake \
        -DENABLE_TESSERACT=ON \

#Camera Optimization (Critical for Low Latency)

Edit camera settings via web UI:

![image alt](https://github.com/MDuke235/autocarsopark/blob/4ddb6b8e485e3801cbe990ed9bfbb4b066f3636a/Screenshot%202025-09-11%20161716.png)

# Hardware Setup

- USB Relay Barrier
  
Connect to COM3/COM4

Baud: 9600 and adjustable

Commands:
Open: A0 01 01 A2

Close: A0 01 00 A1

- HID RFID Readers

Must appear as keyboard devices

Bind via device path 

Auto-bind mode available for setup


# UI Workflow
 - Entrance Mode (Default)

1.	Driver scans RFID card

2.	System checks for open session → rejects if active

3.	Captures front + rear images

4.	Runs YOLO + Tesseract OCR → detects plate

5.	Creates parking session in DB

6.	Pulses barrier open 

7.	Updates UI: plate, time, card ID

- Exit Mode
 
1.	Driver scans RFID card

2.	System loads stored check-in images for guard review

3.	Captures live exit images

4.	Runs OCR → compares plates

5.	If match → closes session, calculates fee, logs revenue

6.	Pulses barrier open 

7.	Updates UI: fee, time out, status


- Admin Panel 

Manage Users, Subscriptions, RFID Cards

Configure Pricing (Static or JSON-based)

View Reports: Daily Revenue, Session History

Soft-delete users, cancel subscriptions

Mark lost cards

# Reliability Features

- Auto-reconnect: If camera stalls >3s, auto-retry with exponential backoff

- Debounce: Prevents duplicate RFID scans 

- Data Integrity: Transactions, NOT NULL, CHECK constraints
- Anonymization: On checkout, RFID is cleared from session (card reusable)

- Fallback OCR: If YOLO fails, Tesseract runs on full image
- Plate Normalization: Converts 

      A1-B2C3 → A1B2C3

# Development Notes
- Build Flags

        ENABLE_TESSERACT=ON — Enable OCR
        HAVE_ONNXRUNTIME=ON — Enable YOLO detection
  
- Extensibility

Add new card readers by implementing ICardReader

Add new barrier types by implementing IBarrier

Add new OCR backends by implementing IOcr

# License
MIT License 


