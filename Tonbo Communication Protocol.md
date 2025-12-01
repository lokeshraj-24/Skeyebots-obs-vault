
-> 26 bits comms
#### Command frame:

Format in which the AI node should send command packets to communicate with the unit - pg10
	- Header (1 byte) : 
		- Single byte to signal the beginning of a packet. It should always be 0xE0.
	- Packet Sequence (2 byte): 
		- Packet sequence number is used by the application to keep track of the command sent. The response packet has the same packet sequence number. The number should be reinitialised to zero once it reaches the maximum value of 0xFFFD (or 65533). MSB is the first byte and LSB is the second byte.
	- Device ID (1 byte) : 
		- ID of the device to which the command should be sent.
	- Device number (1 byte) : 
		- Multiple devices within a unit sharing the same device ID are distinguished by device number. The value should be zero if there is a single device of a unique ID type. Device index number should always start from 0. Device number 0xFF would broadcast to all devices of type device ID.
	- Length (1 byte) :
		- Length of the message from command type to data in bytes. Minimum message size is 3 bytes.
	- Command Type (1 byte) : 
		- Following are the various commands types supported:
			- 'R' : get the value
			- 'W' : set the value
	- Command Code (2 byte)
		- A unique 2-byte identifier for every command.
	- Data (0 - 252 byte)
		- Data associated with the command.
	- Checksum (1 byte):
		- Summing of all the bytes starting from Device ID to Data (modulo256).
	- Stop Bytes (2 bytes):
		- Signals the end of the command packet. There should always be 2 bytes 0xFF, 0xFE.

Set of Device IDs - pg10
 ![[Pasted image 20251016170403.png]]


![[Pasted image 20251016170634.png]]
#### Response frame: pg11

Format in which the Units sends response to AI node, in return for a command message
	- **Header** (1 byte) : 
		- Single byte to signal the beginning of a packet. It should always be 0xE1.
	- **Packet Sequence Number** (2 byte): 
		- Response packet has the same packet sequence number as that of the command packet which requested the operation. MSB is the first byte and LSB is the second byte.
	- **Device ID** (1 byte) : 
		- ID of the device to which the command was sent to.
	- **Device number** (1 byte) : 
		- Multiple devices within a unit sharing the same device id is distinguished by device number. Value should be zero if there is a single device of type unique id. Device number 0xFF denotes a single response packet from all the devices. Also, for device number 0xFF, a command is successful if all the devices returned a successful response. Fails if any of the devices failed to respond.
	- **Length** (1 byte) :
		- Length of the response in bytes from command response type to data. Min length is 2 bytes.
	- **Command Response Type** (1 byte) : 
		- Response/Value is returned corresponding to the following command types requested:
			- 'R' : get the value
			- 'W' : set the value
			- 'S' : Command type ‘S’ indicates that this is a stream packet.
	- **Command Status** (1 byte)
		- It is a 1-byte field which can take the following values:
			- 0x00: Command Executed Successfully
			- 0x01: Checksum error
			- 0x02: Stop Bytes Mismatch/Corrupted
			- 0x03: Device Not Responding or timed out.
			- 0x04: Invalid Data
			- 0x05: Invalid Command ID
			- 0x06: Invalid Device ID
			- 0x0D: Invalid Command
	- **Command Code** (2 byte)
		- A unique 2-byte identifier for every command.
	- **Data** (0 - 252 byte)
		- Data associated with the command.
	- **Checksum** (1 byte):
		- Summing of all the bytes starting from Device ID to Data (modulo256).
	- **Stop Bytes** (2 bytes):
		- Signals the end of the command packet. There should always be 2 bytes 0xFF, 0xFE.

![[Pasted image 20251016171412.png]]



### Communication Ports:

IP Addr = 192.168.2.42 -> over UDP


| Service    | Local Port | Remote Address    |
| ---------- | ---------- | ----------------- |
| main       | 5000       | 192.168.2.42:5000 |
| ptu_stream | 5001       | 192.168.2.42:5001 |
| geo_stream | 5002       | 192.168.2.42:5002 |
|            |            |                   |

RTSP URL:

EO - Electro-optical (RGB)

| Stream         | URL                            |
| -------------- | ------------------------------ |
| eo (RGB)       | rtsp://192.168.2.42:9001/video |
| ir (Grayscale) | rtsp://192.168.2.42:9000/video |

Dataframe communication -> over UDP

Port: 5000
IP: 192.168.2.71
