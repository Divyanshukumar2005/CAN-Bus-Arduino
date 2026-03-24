# 🚗 CAN Bus Communication using Arduino + MCP2515

> Implemented a real-time CAN Bus communication system using two Arduino nodes with MCP2515 CAN controllers and TJA1050 transceivers — transmitting live temperature, humidity, and potentiometer data over the CAN Bus with full performance metrics analysis.

---

## 📋 Overview

CAN (Controller Area Network) Bus is a robust, differential two-wire communication protocol originally designed by Robert Bosch for automotive applications. This project demonstrates a working hardware implementation of CAN Bus between two Arduino nodes, with one acting as transmitter (reading sensors) and the other as receiver (displaying data on LCD).

Performance metrics including **Throughput**, **Goodput**, **Latency**, and **Bus Utilization** were measured in real-time on the receiver node.

---

## 🛠️ Hardware Used

| Component | Role |
|---|---|
| Arduino Uno (×2) | Main microcontroller for each node |
| MCP2515 | External SPI-based CAN controller |
| TJA1050 | CAN Bus transceiver (drives CAN_H / CAN_L) |
| DHT22 Sensor | Temperature & Humidity measurement |
| Potentiometer | Analog value simulation |
| 16×2 LCD | Data display on receiver side |
| 120Ω Resistors (×2) | Bus termination resistors |
| Connecting Wires | CAN Bus line + SPI connections |

---

## ⚡ CAN Bus Architecture

```
┌─────────────────────────────────┐     CAN_H ────────────────────
│         TRANSMITTER NODE        │                                │
│  ┌──────────┐   ┌──────────┐   │     ┌─────────────────────────┐│
│  │  Arduino  │──▶│ MCP2515  │──▶│─────│       TJA1050           ││
│  │          │SPI│ CAN Ctrl │   │     │   CAN Transceiver       ││
│  └──────────┘   └──────────┘   │     └─────────────────────────┘│
│       ▲                         │                                │
│  [DHT22] [Potentiometer]        │     CAN_L ────────────────────
└─────────────────────────────────┘

┌─────────────────────────────────┐
│          RECEIVER NODE          │
│  ┌──────────┐   ┌──────────┐   │
│  │  Arduino  │◀──│ MCP2515  │◀──│── CAN Bus
│  │          │SPI│ CAN Ctrl │   │
│  └──────────┘   └──────────┘   │
│       │                         │
│    [LCD Display]                │
└─────────────────────────────────┘
```

**Bus Termination:** 120Ω resistors at both ends of CAN_H line.

---

## 📦 CAN Frame Format Used

```
┌─────┬──────────────┬───────────────┬────────────┬───────────┬──────────┬─────┐
│ SOF │  Identifier  │ Control Field │ Data Field │ CRC Field │ ACK Field│ EOF │
│ 1b  │   11 bits    │  DLC (4 bits) │  0–8 bytes │  15 bits  │  2 bits  │ 7b  │
└─────┴──────────────┴───────────────┴────────────┴───────────┴──────────┴─────┘
```

**CAN IDs used in this project:**
- `0xAA` — Potentiometer value (1 byte)
- `0xBB` — Temperature + Humidity + Timestamp (6 bytes)

---

## 🔌 Wiring & Pin Connections

### MCP2515 → Arduino SPI

| MCP2515 Pin | Arduino Pin |
|---|---|
| VCC | 5V |
| GND | GND |
| SCK | Pin 13 (SCK) |
| MOSI | Pin 11 (MOSI) |
| MISO | Pin 12 (MISO) |
| CS | Pin 10 |
| INT | Pin 2 |

### DHT22 Sensor (Transmitter)

| DHT22 Pin | Arduino Pin |
|---|---|
| VCC | 5V |
| GND | GND |
| DATA | Pin A1 |

### CAN Bus Lines

```
MCP2515 (TX) CAN_H ──────────────── MCP2515 (RX) CAN_H
             CAN_L ──────────────── CAN_L
```

---

## 💻 Software & Libraries

```cpp
#include <SPI.h>
#include <mcp2515.h>    // Seeed-Studio/CAN_BUS_Shield or autowp/mcp2515
#include <DHT.h>        // adafruit/DHT-sensor-library
```

**CAN Bus Config:**
```cpp
#define CAN_SPEED   CAN_500KBPS
#define MCP_CRYSTAL MCP_8MHZ
```

---

## 📂 Repository Structure

```
can-bus-arduino-mcp2515/
│
├── README.md
├── transmitter/
│   └── transmitter.ino         ← Sender: reads DHT22 + Pot → sends CAN frames
├── receiver/
│   └── receiver.ino            ← Receiver: reads CAN frames → LCD + Serial metrics
├── docs/
│   ├── circuit_diagram.png     ← Full wiring diagram
│   ├── serial_output.png       ← Serial monitor showing live metrics
│   └── lcd_output.jpg          ← LCD displaying received sensor data
└── presentation/
    └── CAN_Bus_Slides.pdf      ← Project presentation
```

---

## 📊 Results & Performance Metrics

From live hardware testing at **500 kbps**:

| Metric | Value |
|---|---|
| **Frames/sec** | 23 |
| **Throughput** | 2.10 kbps |
| **Goodput** | 0.66 kbps |
| **Bus Utilization** | 0.42% |
| **TX Time (avg)** | 182.17 µs |
| **TX Time (min/max)** | 130 / 230 µs |
| **Latency (avg)** | 308,628 ms* |
| **Power** | 0.0512 W |

> *High latency value is due to cumulative delay measurement from program start, not per-frame latency. Per-frame TX time is ~182 µs.

---

## 🚀 How to Run

1. **Install libraries** via Arduino IDE → Library Manager:
   - `mcp2515` by autowp
   - `DHT sensor library` by Adafruit

2. **Upload** `transmitter/transmitter.ino` to **Arduino 1** (sender)

3. **Upload** `receiver/receiver.ino` to **Arduino 2** (receiver)

4. **Wire** both MCP2515 modules together:
   - Connect CAN_H to CAN_H
   - Connect CAN_L to CAN_L
   - Add 120Ω termination resistors at both ends

5. **Open Serial Monitor** at `115200 baud` on receiver to see live metrics

6. **LCD** will show: `Pot=XX | T=XXC | H=XX%`

---

## 📈 Key Learnings

- SPI communication between Arduino and MCP2515
- CAN frame construction (ID, DLC, data bytes, CRC)
- Differential signaling for noise immunity
- Multi-node bus architecture with collision-free arbitration
- Real-time performance metric calculation (throughput, goodput, bus utilization)

---

## 🔧 Future Improvements

- Efficiency vs Payload Size analysis (1–8 bytes)
- Bitrate scaling study: 500 kbps → 1 Mbps
- Add error handling for CRC failures and ACK retries
- Extend to 3+ node network

---

## 👤 Author

**Divyanshu Kumar**  
B.Tech ECE, University of Delhi (2023–2027)  
[LinkedIn](https://www.linkedin.com/in/divyanshu-kumar-7625a8315) | [GitHub](https://github.com/Divyanshukumar2005)

---

## 📄 License

MIT License — feel free to use for educational purposes.
