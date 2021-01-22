---
title: "Creating Command Line Interfaces for Embedded Projects Using Python"
date: 2021-01-21T15:34:30-04:00
toc: true
toc_sticky: true
---

Despite the nature of embedded systems, a project will inevitably come along where you need more than a debug probe between your dev board and PC. Maybe you want to send a command to start data collection and get that data sent back to your PC to stick into a spreadsheet for further analysis, or you want to be able to collect diagnostics from your device without the hassle of a debugger. There are a number of tools out there that have this basic functionality built into them in some form, such as the serial monitor present in programs like the Arduino IDE or my favorite VSCode extension, PlatformIO. But solutions like these remain a tedious ping-ponging of discrete serial lines, fine for simple applications but difficult to expand upon and impossible to deliver as a product to an possible end user. This blog post is meant for those who need more, who want a reliable, specially-made application to interface with their project in flexible and powerful ways. Unsurprisingly, the easy and cross-platform solution is found in Python. We'll be starting with an overview of a few useful modules that make all of this possible, cover how to handle sending complex and large packets of serial data using encoding, and examine good practice for designing easy-to-use command line interfaces that self-document.

## Useful Modules
### pyserial
If you've ever explored communicating with embedded devices using Python, chances are you've encountered the popular and powerful [pySerial](https://pyserial.readthedocs.io/en/latest/pyserial.html) module. This module lets you communicate over the serial port on all major operating systems using Python, and is the most critical and fundamental module mentioned in this post. Alone, it replaces the Arduino-style serial monitor as the communication medium between your PC and microcontroller. It's also incredibly simple to implement:
```Python
# install with "pip install serial"
import serial

serial_port = "/dev/ttyUSB0" # or COM3, for example, on Windows
baudrate = 115200
# this is the only line needed to set up and open your serial port
ser = serial.Serial(serial_port,baudrate,timeout=200)

# waits for a line (stream of bytes terminated by the newline character \n) to be received over the opened serial port, and stores it as a byte array into rx
rx = ser.readline()

# send a string over serial 
tx = "my message"
ser.write(tx.encode())

# send an integer over serial
tx_int = (42).to_bytes(1,"big")
ser.write(tx_int)
```
Throw in a while loop and some ```input()``` calls and you can see how this module alone elevates your ability to interface with your microcontroller. Of course, you'll need to write the corresponding code on the embedded side. Here's a simple pair of code snippets for a PC and STM32 microcontroller that has the microcontroller return bytes it recieves:
**Client:**
```Python
import serial

ser = serial.Serial ("/dev/ttyUSB0",115200,timeout=200)

while True:
	tx = input("Send a byte over serial: ")
	ser.write (tx.encode())
	
	rx = str(ser.readline())
	print (rx)
```
**STM32 (HAL Library):**
```C
// receive buffer for UART
uart_rx_buffer[1];

int main(void) {
	// stuff
	
	// enable UART interrupt
	HAL_UART_Receive_IT(&huart4,uart_rx_buffer,1);
	while(1)
	{
		// more stuff
	}	
}

/*
 * @brief Interrupt triggered when rx data received 
 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
	// assume we are using UART4
	if (huart-> Instance == UART4)
	{
		// send recieved byte over UART
		HAL_UART_Transmit(huart,uart_rx_buffer,1,10);
		
		// reenable UART interrupt
		HAL_UART_Receive_IT(&huart4,uart_rx_buffer,1);
	}
```
### os

### sys

## Encoding Data Packets

##
