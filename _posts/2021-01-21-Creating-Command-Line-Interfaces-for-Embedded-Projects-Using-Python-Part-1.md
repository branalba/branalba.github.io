---
title: "Creating Command Line Interfaces for Embedded Projects Using Python - Part 1: Communication Between Your Client and Device"
date: 2021-01-21T15:34:30-04:00
toc: true
toc_sticky: true
---

Despite the embedded nature of embedded systems, a project will inevitably come along where you need something other than a debug probe between your dev board and PC. Maybe you want to send a command to start data collection and get that data sent back to your PC to stick into a spreadsheet for further analysis, or you want to be able to collect diagnostics from your device without the hassle of a debugger. There are a number of tools out there that have this basic functionality built into them in some form, such as the serial monitor present in programs like the Arduino IDE or my favorite VSCode extension, PlatformIO. But solutions like these remain a tedious ping-ponging of discrete serial lines, fine for simple applications but difficult to expand upon and impossible to deliver as a product to an possible end user. This blog post is meant for those who need more, who want a reliable, specially-made application to interface with their project in flexible and powerful ways. Unsurprisingly, the easy and cross-platform solution is found in Python. We'll be starting with an overview of a few useful modules that make all of this possible, cover how to handle sending complex and large packets of serial data using encoding, and examine good practice for designing easy-to-use command line interfaces that self-document.

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
Throw in a while loop and some ```input()``` calls and you can see how this module alone elevates your ability to interface with your microcontroller. Of course, you'll need to write the corresponding code on the embedded side. Here's a simple pair of code snippets for a PC and STM32 microcontroller that has the microcontroller return bytes it receives:

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
		// send received byte over UART
		HAL_UART_Transmit(huart,uart_rx_buffer,1,10);
		
		// reenable UART interrupt
		HAL_UART_Receive_IT(&huart4,uart_rx_buffer,1);
	}
```
One other useful function of pySerial is the ability to scan for serial ports. This can be useful when you have multiple ports open on the same PC, or when you want your program to automatically detect a serial port from a specific device. I use the following function in all of my serial communication clients, as it gets rid of the occassional headache associated with the same device getting assigned different port numbers:
```Python
# only the second module is strictly required, but you'll seldom be using this function without the rest of the serial module
import serial,serial.tools.list_ports

def scan_serial_ports(keyword):
    """scans the available serial ports and returns the port whose description contains the keyword

    Args:
        keyword (string): the keyword to look for in a port description

    Returns:
        string: the port that satisfies the keyword or 0 if no matching port found
    """
	
	# the magic line that gets the port number, description, and hardware ID for all connected serial devices
    ports = serial.tools.list_ports.comports()

	# loop through the ports
    for port, desc, hwid in sorted(ports):
        print_verbose("Found {}: {} [{}]".format(port,desc,hwid))
        # return the port name if the sought device is connected
		if keyword in desc:
            return port
    # throw an error output if the device was not found
	return 0
```
## Encoding Data Packets
### Why Encode?
The encoding of serial data might sound unnecessary at first -- even if the application requires a constant stream of data samples, such as the output of a gyroscope/accelerometer, can't you just terminate each data set with the newline character, slap a function like ```readline()``` on the client side, and be done with it? In many cases, yes, but this is neither good practice nor 100% reliable. To see why, first remember that the newline character is not inherently different from any other byte (in fact, you can use any byte or bytes as a newline character if you wish, the newline character is just the standard). Its ASCII code, or true binary representation, is simply ```00001010 = 10```. Remember, the only indicator pySerial uses to detect a "line" of serial data is the newline. So going back to the question of why encoding is useful, ask youself if using the number 10 as a trigger for the end of a packet is a reliable way to separate a stream of packets from one another.

The answer, of course, is that it is not reliable. In fact, by relying on the number 10 to terminate a packet, we are resigning ourselves to splitting our data packets in two every time one of our bytes of actual data is equal to 10! Some serial communicators mitigate this issue by adding another "special" byte, the carriage return character ```'\r' = 00001101 = 13```. So an example data packet from a microcontroller would look like: ```"done\r\n"``` or ```[100,111,110,101,13,10]``` represented as a list of ASCII codes. However, it should be clear that all this does is reduce the likelihood of transmitting a bad packet. It is _less likely_, but far from impossible, to end up with the byte sequence ```00001101 00001010``` in a data packet compared to just ```00001010```. And in any situation in which you are counting up, perhaps keeping track of time or you are measuring a value that inherently will steadily increase, you are _guaranteed_ to encounter this sequence of bits if they represent a 16-bit integer or larger (specifically, it's the number 3,338).

All of this is not to say that you shouldn't use the newline and carriage return approach to terminate your packets, just that doing so places a "forbidden" sequence of bytes your system cannot properly transmit. In some applications, this is perfectly fine, but in others it can be completely unacceptable. And in the context of a client, one that you won't want to muck around with while you're interacting with your project and may even deploy to a client eventually, you need a robust line of communication. The solution to this is not to continue adding bytes to our newline assignment -- we want a bulletproof solution, not one that only works most of the time. This is where encoding comes in. There are a lot of methods out there, but my favorite due to its simpliticy and low overhead (it only adds two extra bytes to an unencoded packet's length) is known as consistent overhead byte stuffing, or COBS.

### COBS
COBS directly addresses the problem of choosing a terminating byte (the terminating byte can be called the "delimiter") for a packet in a surprisingly direct way -- eliminate all occurences of the delimiter present in the original data packet by replacing them with a different value, so the chance of the packet "splitting" goes from slim to none. But then haven't we changed the content of the data packet? Clearly, we need some way of knowing what bytes in a data set were replaced. This is accomplished via the "overhead" byte, a single value appended to the beginning of the dataset. Its value is the number of bytes _after itself_ that a replaced byte is at. The replaced byte, in turn, has a value corresponding to the number of bytes after itself that the next replaced byte is at, on and on until the final replaced byte points to the number of bytes after itself that the true delimiter byte is at. The receiver (decoder) knows this is the true delimiter byte because it is already the delimiter value -- it was never replaced.

To illustrate the above method, let's start with a sample packet of data that we need to encode: ```[45,23,0,249,21,0,53]```. Let's say we've chosen the number 0 as our delimiter. Thus, our partially encoded data set becomes ```[45,23,0,249,21,0,53,0]```. Now, we add the overhead bit, which should have a value telling us how many bytes into the data set that the first instance of the delimiter byte 0 appears. In this case, the first 0 in the data set is the third byte in the list. Thus, our overhead byte is 3 and our packet now looks like ```[3,45,23,0,249,21,0,53,0]```. The last step is to replace the 0s in the data set. The 0 that our overhead byte pointed to needs to be replaced by the number of bytes after iself the next 0 is at, which from our dataset turns out to be another 3. That 0, in turn, needs to point to the next one, which is also our delimiter and is two items away from that second 0, so we replace it with 2. That's it! Our final encoded data set is now ```[3,45,23,3,249,21,2,53,0]```. All a recipient, or decoder, needs to know is the value of the delimiter byte. Everything else is easy -- the first byte received will be the overhead byte, and the decoder therefore knows to throw it away after using it to find the location of the first replaced 0. 

Think of the overhead byte like the beginning of a treasure map -- it points you to the next clue, which points you to the next and the next until the last clue finally points you to the treasure. In this case, however, the clues are our overhead byte followed by the replaced 0s, and the treasure is our delimiter byte. If you're still having trouble understanding COBS, check out its [Wikipedia page](https://en.wikipedia.org/wiki/Consistent_Overhead_Byte_Stuffing) for another perspective. 

### Example
Below are two implementations of COBS, an encoder written in C (intended for a microcontroller) and a decoder written in Python (intended for our CLI client). 

**Encoder:**
```C
/*
 * @brief Encodes a packet of data using the Consistent Overhead Byte Stuffing method
 * @params
 * @retval none
 */

void COBS_Encode_Packet(uint8_t *input, uint8_t input_len, uint8_t *output, uint8_t output_len, uint8_t delimiter) {
	int offset_index;
	int count = 0;

	//Set overhead byte to 0
	output [0] = 0;

	//Loop through output buffer
	for(int i=1;i<(output_len-1);i++) {
		count++;
		if (input[i-1] == delimiter) {
			if (output[0] == 0) {
				output[0] = count;
			} else {
				output[offset_index] = count;
			}
			offset_index = i;
			count = 0;
		}
		output[i] = input[i-1];
	}

	output[output_len-1] = delimiter;
	if(output[0] == 0) {
		output[0] = count+1;
	} else {
		output[offset_index] = count+1;
	}
}
```
**Decoder:**
```Python
def COBS_Decode_Packet(delim):
    """Decodes incoming serial data according to the Consistent Overhead Byte Stuffing method

    Args:
        delim (8-bit integer): number used to indicate end of packet, needed for decoding

    Returns:
        list: Int list containing the decoded packet
    """
    out = []
    count = 0
    overhead_received = False
    byte = int.from_bytes(ser.read(),"big")

    while (byte != delim):
        if overhead_received == False:
            pos = byte
            overhead_received = True
        else:
            if count == pos:
                out.append(delim)
                pos = count + byte
            else:
                out.append(byte)
        count += 1
        byte = int.from_bytes(ser.read(),"big")

    return out
```
Of course the reverse is also nice to have (a decoder on your microcontroller and an encoder on your client) but the need for the former is far more common -- rarely do you need to stream a ton of arbitrary data to a microcontroller from your PC. Commands sent from your PC are generally specific command words that you can choose, avoiding the need for encoding and decoding by avoiding the use of newline or carriage return. Something I like to do for commands, to keep things simple, is use single bytes as commands to a microcontroller. If it's only processing one byte at a time, there's no need for any sort of packet termination. The byte's presence alone terminates the "packet".

