
# Secure Wireless FPGA-to-FPGA Communication Using AES-128

## Overview

This project implements a secure communication system between two FPGA boards using AES-128 encryption and decryption. The two FPGAs communicate through a wireless link implemented using two ESP32 devices and the ESP-NOW protocol.

The system performs password-based authentication before allowing data transfer. After successful authentication, plaintext entered on PC1 is sent to FPGA1, encrypted using AES-128, transmitted wirelessly through the ESP32 bridge, received by FPGA2, decrypted, and finally displayed on PC2.

The project integrates FPGA-based cryptography, UART communication, wireless communication, password-based authentication, and ping-pong buffering into a complete end-to-end secure communication system.

---

## Project Objective

The main objective of this project is to demonstrate secure data communication between two FPGA systems over a wireless channel.

The project focuses on the following areas:

* AES-128 encryption and decryption implemented on FPGA
* Password-based authentication before data transfer
* UART communication between PC, FPGA, and ESP32
* Wireless data transmission using ESP-NOW
* Continuous data handling using ping-pong buffers
* Support for text and binary file transfer
* End-to-end communication from PC1 to PC2

The FPGA is responsible for cryptographic processing and authentication, while the ESP32 boards act as wireless communication bridges.

---

## Working Principle

The system operates in two main phases:

1. Authentication Phase
2. Secure Data Transfer Phase

Before transferring encrypted data, FPGA1 and FPGA2 must complete the authentication process successfully. Once authentication is completed, the system enters streaming mode and encrypted data can be transferred from PC1 to PC2.

---

## Authentication Phase

The authentication process uses an 8-character alphanumeric password.

The general operation is as follows:

1. The user starts the PC2 receiver program and enters an 8-character password.
2. The password is sent to FPGA2 through UART.
3. The user starts the PC1 sender program and enters the password.
4. The password is sent to FPGA1 through UART.
5. The FPGAs process the password and generate the required key or hash information.
6. FPGA1 sends synchronization and authentication-related data toward FPGA2 through the communication path.
7. FPGA2 compares the received authentication information with its locally generated information.
8. If the authentication is successful, FPGA2 sends an acknowledgment (`0xAA`).
9. Both systems then enter the data streaming mode.

If the authentication information does not match, the system does not proceed with normal encrypted data transfer.

The PC programs require the password to contain exactly 8 alphanumeric characters.

Allowed characters are:

* `A-Z`
* `a-z`
* `0-9`

Example valid passwords:

```text
Abc12345
Pass1234
AES20260
```

---

## Secure Data Transfer

After successful authentication, the user can enter plaintext or provide a file path through the PC1 sender program.

The PC1 program reads the input and prepares it for AES processing. Since AES-128 operates on 128-bit blocks, the data is divided into blocks of 16 bytes.

If the last block contains fewer than 16 bytes, zero padding is added.

The data transfer process is as follows:

1. PC1 sends plaintext to FPGA1 through UART.
2. FPGA1 receives the data and organizes it into AES blocks.
3. FPGA1 encrypts the plaintext using AES-128.
4. The resulting ciphertext is sent from FPGA1 to ESP32-1 through UART.
5. ESP32-1 forwards the ciphertext to ESP32-2 using ESP-NOW.
6. ESP32-2 receives the wireless packet.
7. ESP32-2 sends the received ciphertext to FPGA2 through UART.
8. FPGA2 decrypts the ciphertext using AES-128.
9. FPGA2 sends the recovered plaintext to PC2 through UART.
10. PC2 receives and displays the original message.

The ESP32 devices do not perform encryption or decryption. Their primary purpose is to transport data between the FPGA systems over the wireless link.

---

## AES-128 Processing

AES-128 operates on fixed-size blocks of 128 bits.

Each AES block contains:

```text
128 bits = 16 bytes
```

The plaintext is encrypted by FPGA1, producing a 128-bit ciphertext block.

The ciphertext is transmitted through the UART and ESP-NOW communication path to FPGA2.

FPGA2 receives the ciphertext and performs AES-128 decryption to recover the original plaintext.

For messages that are not an exact multiple of 16 bytes, the PC1 sender applies zero padding to complete the final AES block.

For example, a message shorter than 16 bytes is padded with `0x00` bytes before transmission.

At the receiver side, trailing zero bytes can be removed when displaying the recovered message.

The project uses an ECB-style block transfer where data is processed in 16-byte AES blocks.

---

## Ping-Pong Buffer

The system uses a ping-pong buffering technique to support continuous data processing.

A ping-pong buffer consists of two buffers that alternate their roles.

While one buffer is being processed, transmitted, or encrypted, the other buffer can receive new incoming data.

After the current processing operation is completed, the roles of the buffers are exchanged.

For example:

```text
Buffer A: Processing current data
Buffer B: Receiving next data
```

After switching:

```text
Buffer A: Receiving next data
Buffer B: Processing current data
```

This alternating operation continues during continuous data transfer.

---

## Why Ping-Pong Buffering Is Needed

Without ping-pong buffering, a single buffer may need to perform all operations sequentially.

The system would need to:

1. Receive data
2. Wait for the buffer to be processed
3. Encrypt or decrypt the data
4. Transmit the data
5. Receive the next data

During processing or transmission, the system may not be able to accept the next block efficiently.

Using two buffers allows one buffer to handle incoming data while the other buffer is being processed.

This provides several advantages:

* Improved data throughput
* More continuous operation
* Reduced waiting time between blocks
* Better utilization of the communication interface
* Better handling of continuous AES block transfers

Ping-pong buffering is particularly useful when the input data is larger than a single AES block and multiple blocks must be processed continuously.

---

## Wireless Communication

The wireless communication between the two FPGA systems is implemented using two ESP32 boards.

Each ESP32 acts as a bridge between an FPGA UART interface and the ESP-NOW wireless link.

ESP32-1 is connected to FPGA1 through UART.

ESP32-2 is connected to FPGA2 through UART.

The communication process is bidirectional.

For FPGA-to-wireless communication:

1. The ESP32 receives bytes from the FPGA through UART.
2. The received bytes are stored in a UART buffer.
3. After a short UART timeout, the collected bytes are treated as a packet.
4. The packet is transmitted to the other ESP32 using ESP-NOW.

For wireless-to-FPGA communication:

1. An ESP-NOW packet is received by the ESP32.
2. The received data is copied into a receive buffer.
3. The main program processes the received buffer.
4. The data is forwarded to the connected FPGA through UART.

The current ESP32 bridge implementation uses a maximum packet buffer size of 240 bytes and a UART timeout of approximately 5 ms to determine when a UART packet has finished arriving.

---

## Why Wireless Communication?

The encryption and decryption system can also be tested using a direct wired connection between FPGA1 and FPGA2.

A direct wired connection can be useful for verifying:

* AES encryption
* AES decryption
* UART communication
* Authentication logic
* Data synchronization
* FPGA functionality

However, the main objective of this project is not only to encrypt and decrypt data, but also to demonstrate secure communication across a wireless link.

A wired connection provides a direct physical path between the two systems. In this project, the FPGA boards can communicate even when they are physically separated, with the ESP32 devices providing the wireless transport layer.

The wireless link makes the project more relevant to applications such as:

* Secure IoT communication
* Remote embedded systems
* Industrial wireless communication
* Secure sensor networks
* Distributed FPGA systems
* Wireless hardware accelerators

The use of wireless communication also demonstrates that cryptographic processing can remain inside dedicated FPGA hardware while the wireless communication layer is handled by separate embedded devices.

---

## Why ESP-NOW?

ESP-NOW is used for communication between the two ESP32 boards.

ESP-NOW allows direct communication between ESP32 devices without requiring a traditional Wi-Fi router or internet connection.

Some advantages of ESP-NOW include:

* Direct peer-to-peer communication
* No Wi-Fi router required
* No internet connection required
* Low communication overhead
* Suitable for embedded and IoT applications
* Fast communication between ESP32 devices

In this project, ESP-NOW is used only as the wireless transport mechanism.

The actual AES encryption and decryption are performed by the FPGA hardware.

---

## ESP32 UART-to-ESP-NOW Bridge

Both ESP32 boards implement a bidirectional UART-to-wireless bridge.

The bridge supports the following data paths:

```text
FPGA -> UART -> ESP32 -> ESP-NOW -> Remote ESP32
```

and:

```text
ESP-NOW -> ESP32 -> UART -> FPGA
```

The ESP32 software includes:

* UART initialization
* ESP-NOW initialization
* Peer configuration using MAC addresses
* UART receive buffering
* ESP-NOW receive buffering
* ESP-NOW transmission
* ESP-NOW reception callbacks
* Send status monitoring
* Hexadecimal data display through Serial Monitor

The Serial Monitor can also be used to manually send hexadecimal data for testing.

For example:

```text
DE AD BE EF 12 34
```

The ESP32 program converts this hexadecimal input into raw bytes and transmits it through ESP-NOW.

---

## UART Configuration

The UART communication uses the following configuration:

| Parameter | Value  |
| --------- | ------ |
| Baud Rate | 115200 |
| Data Bits | 8      |
| Parity    | None   |
| Stop Bits | 1      |

The UART format is therefore:

```text
8N1
```

The ESP32 UART connection uses:

* RX: GPIO16
* TX: GPIO17
* Baud Rate: 115200

The Python programs also communicate with the FPGA through UART at 115200 baud.

---

## PC1 Sender

The PC1 sender application is implemented in:

```text
pc1_ecb_sender.py
```

Its main responsibilities are:

* Opening the UART connection to FPGA1
* Accepting an 8-character password
* Sending the password to FPGA1
* Waiting for the authentication handshake
* Accepting text input
* Reading binary or text files
* Padding data to 16-byte AES block boundaries
* Sending plaintext to FPGA1
* Receiving ciphertext echo
* Displaying the received ciphertext in hexadecimal form

The program allows the user to enter either plaintext or a file path.

If the entered text corresponds to a valid file, the program reads the file as binary data.

Otherwise, the input is treated as a text message and encoded using UTF-8.

The sender also calculates a timeout based on the UART baud rate and the number of transmitted bytes.

---

## PC2 Receiver

The PC2 receiver application is implemented in:

```text
pc2_ecb_receiver.py
```

Its main responsibilities are:

* Opening the UART connection to FPGA2
* Accepting an 8-character password
* Sending the password to FPGA2
* Waiting for the authentication handshake
* Receiving decrypted AES blocks
* Collecting received data in the background
* Displaying plaintext
* Displaying hexadecimal data
* Optionally saving the received data to a binary file

The receiver expects decrypted data in 16-byte AES blocks.

It uses configurable timeouts for:

* Waiting for the first byte of a block
* Waiting for subsequent bytes in the block

This allows the receiver to distinguish between delayed data and the end of transmission.

---

## Background Serial Readers

Both Python applications use a background thread to continuously monitor incoming UART data.

The sender uses an `EchoReader` to collect ciphertext returned from FPGA1.

The receiver uses a `ByteReader` to collect decrypted bytes received from FPGA2.

The background reader approach allows serial data to be collected continuously without blocking the main user interface.

A thread-safe buffer and lock are used to protect the received data.

---

## Data Padding

AES-128 requires data to be processed in 16-byte blocks.

If the input length is not an exact multiple of 16 bytes, the sender adds zero bytes to the final block.

For example:

```text
Original data: 11 bytes
Padding required: 5 bytes
Final AES block: 16 bytes
```

The receiver can remove trailing zero bytes when displaying text.

This implementation is intended for the current project data flow and demonstration.

---

## Hardware Requirements

The project requires:

* Two FPGA boards
* Two ESP32 boards
* UART connections between FPGA and ESP32
* USB-UART connection between PC and FPGA
* Two computers or serial interfaces for PC1 and PC2
* ESP-NOW communication between the ESP32 boards

The FPGA boards perform:

* Password processing
* Authentication
* AES encryption
* AES decryption
* Data buffering
* Data control and synchronization

The ESP32 boards perform:

* UART communication
* ESP-NOW communication
* Wireless packet forwarding

---

## Software Requirements

### Python

Python 3 is required.

Install the serial communication library using:

```bash
pip install pyserial
```

### ESP32

The ESP32 code can be compiled using the Arduino IDE with ESP32 board support installed.

The implementation uses:

* WiFi library
* ESP-NOW
* ESP Wi-Fi functions

Required headers include:

```cpp
#include <WiFi.h>
#include <esp_now.h>
#include <esp_wifi.h>
```

---

## Running the Project

### Step 1: Program the FPGA Boards

Program FPGA1 with the design containing the required encryption, authentication, buffering, and communication modules.

Program FPGA2 with the design containing the required decryption, authentication, buffering, and communication modules.

### Step 2: Program the ESP32 Boards

Upload the ESP32 bridge programs to both ESP32 devices.

Make sure the MAC address of each ESP32 is correctly configured in the peer information.

### Step 3: Start the PC2 Receiver

Start the PC2 receiver before starting the sender.

Example:

```bash
python pc2_ecb_receiver.py --port COM5
```

Enter the 8-character password when prompted.

The receiver waits for the authentication process and incoming decrypted data.

### Step 4: Start the PC1 Sender

Run the PC1 sender:

```bash
python pc1_ecb_sender.py --port COM4
```

Enter the same password used on PC2.

The sender waits for the authentication handshake to complete.

### Step 5: Send Data

Enter a plaintext message or provide a file path.

For example:

```text
Hello FPGA
```

or:

```text
input.txt
```

The data is padded to 16-byte AES blocks if necessary and sent to FPGA1.

FPGA1 encrypts the data, and the ciphertext is transmitted through the wireless ESP-NOW link.

FPGA2 receives and decrypts the data, and PC2 displays the recovered plaintext.

---

