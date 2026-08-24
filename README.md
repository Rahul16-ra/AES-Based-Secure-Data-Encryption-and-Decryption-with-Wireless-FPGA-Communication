# AES-Based-Secure-Data-Encryption-and-Decryption-with-Wireless-FPGA-Communication
Designed an FPGA-based secure communication system using AES encryption/decryption. Integrated 64-bit password-based key generation and authentication, USB-UART, ESP32 ESP-NOW wireless transfer, and Python interfaces to securely transmit encrypted data between two FPGA systems and recover the original data.
# Secure Wireless FPGA-to-FPGA Communication Using AES-128

This project implements a secure wireless communication system between two FPGAs using **AES-128 encryption/decryption**, **ESP-NOW wireless communication**, UART interfaces, and password-based authentication.

The system allows data entered on **PC1** to be encrypted by FPGA1, transmitted wirelessly through two ESP32 devices, decrypted by FPGA2, and displayed on **PC2**.



Features
AES-128 encryption and decryption
Password-based authentication
Wireless FPGA-to-FPGA communication
ESP-NOW communication between ESP32 devices
UART communication between FPGA and ESP32
16-byte AES block processing
Ping-pong buffering for continuous data transfer
>Python-based PC sender and receiver programs
Hex monitoring and debugging through ESP32 Serial Monitor

Working Principle
1. Authentication

Before transferring data, both FPGA systems perform password-based authentication.

>The user enters an 8-character alphanumeric password on both PCs.
>The password is processed inside the FPGA.
>A hash/key value is generated.
>FPGA1 sends synchronization and authentication information to FPGA2.
>FPGA2 compares the received authentication data with its locally generated value.
>If authentication is successful, FPGA2 sends an 0xAA acknowledgment.
>The system then enters data transfer mode.

If the password does not match, data transfer is not allowed.

Data Transfer

After successful authentication:

>PC1 sends plaintext to FPGA1 through UART.
>FPGA1 groups the data into 16-byte blocks.
>If required, the final block is padded with zeros.
>FPGA1 encrypts each block using AES-128.
>The ciphertext is sent to ESP32-1 through UART.
>ESP32-1 transmits the ciphertext wirelessly using ESP-NOW.
>ESP32-2 receives the ciphertext.
>ESP32-2 forwards it to FPGA2 through UART.
>FPGA2 decrypts the AES block.
>The plaintext is sent to PC2.


Ping-Pong Buffer

The FPGA uses a ping-pong buffer to improve continuous data processing.

A ping-pong buffer uses two memory buffers:
             Incoming Data
                   |
                   v
              +---------+
              | Buffer A |
              +---------+
                   |
                   v
              AES Processing


At the same time:

              +---------+
              | Buffer B |
              +---------+
                   ^
                   |
             Receiving Data


  Cycle 1:

Buffer A -> AES Processing
Buffer B -> Receiving Data


Cycle 2:

Buffer A -> Receiving Data
Buffer B -> AES Processing


This prevents the system from stopping data reception while the previous block is being processed.

Without ping-pong buffering, the system may need to wait for encryption or transmission to finish before receiving the next data block.

Therefore, ping-pong buffering improves:

>Data throughput
>Continuous operation
>Buffer utilization
>Overall communication efficiency


Why Wireless Communication?

The system can also be tested using a direct wired connection between the two FPGAs.
This is useful for testing AES encryption, decryption, UART communication, and overall FPGA functionality.

However, the purpose of this project is to demonstrate secure communication over a wireless link.
Using wireless communication makes the project closer to a real secure communication system.

The FPGA performs computational operations such as:

Password processing
Authentication
AES encryption
AES decryption
Data buffering

The ESP32 devices are mainly used as wireless bridges.

Why ESP-NOW?

ESP-NOW is used because it provides direct communication between ESP32 devices without requiring a traditional Wi-Fi router.

Advantages include:

Low latency
Direct ESP32-to-ESP32 communication
No internet connection required
Simple peer-to-peer communication
Suitable for embedded and IoT applications

Software Components

PC1 Sender
Responsibilities:

Accept an 8-character password
Send the password to FPGA1
Wait for authentication
Accept text or file input
Pad data to AES 16-byte blocks
Send plaintext to FPGA1
Receive ciphertext echo for verification


PC2 Receiver

pc2_ecb_receiver.py

Responsibilities:

Accept the authentication password
Send the password to FPGA2
Wait for authentication
Receive decrypted AES blocks
Display received plaintext
Optionally save received data to a binary file
ESP32-1

Acts as a bridge between FPGA1 and ESP32-2.

FPGA1 UART -> ESP32-1 -> ESP-NOW
ESP-NOW -> ESP32-1 -> FPGA1 UART
ESP32-2

Acts as a bridge between FPGA2 and ESP32-1.

FPGA2 UART -> ESP32-2 -> ESP-NOW
ESP-NOW -> ESP32-2 -> FPGA2 UART

The ESP32 programs also support:

Raw byte transmission
ESP-NOW callbacks
UART buffering
Hex display through Serial Monitor
AES Processing

AES-128 operates on a 128-bit data block.

Plaintext (128 bits)
        |
        v
   AES Encryption
        |
        v
Ciphertext (128 bits)
        |
        | Wireless Transmission
        v
Ciphertext (128 bits)
        |
        v
   AES Decryption
        |
        v
Plaintext (128 bits)

The encryption and decryption operations are implemented in the FPGA, while the ESP32 devices only transport the data.

Data Format

AES requires data in blocks of:

16 Bytes = 128 Bits

Example:

Input:
Hello World

After Zero Padding:

48 65 6C 6C 6F 20 57 6F
72 6C 64 00 00 00 00 00

The padded block is encrypted and transmitted through the system.

Hardware Used
2 × FPGA boards
2 × ESP32 boards
USB-UART interfaces
UART connections
ESP-NOW wireless communication
UART Configuration
Parameter	Value
Baud Rate	115200
Data Bits	8
Parity	None
Stop Bits	1
Project Flow
1. Start PC2 receiver

2. Enter password on PC2

3. Start PC1 sender

4. Enter the same password on PC1

5. FPGA1 and FPGA2 perform authentication

6. Authentication successful

7. Enter plaintext or file on PC1

8. FPGA1 encrypts the data using AES-128

9. ESP32-1 sends ciphertext through ESP-NOW

10. ESP32-2 forwards ciphertext to FPGA2

11. FPGA2 decrypts the ciphertext

12. PC2 receives the original plaintext




                                     
