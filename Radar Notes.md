
RDP -> Radar Data Processor


| Acronym | Full Form                        |
| ------- | -------------------------------- |
| RDP     | Radar Data Processor             |
| RSP     | Radar Signal Processor           |
| ERC     | Embedded Radar Control unit      |
| RCDS    | Radar Control and Display System |
| PTD     | Periodic Track Data              |

RSP, RDP and ERC are all in the same SoC chip

RCDS -> the only remote node, which is the companion system attached to the radar (ex: AI node or Jetson in Drone)

PTD -> the only message that matters, containing all tracking data and metadata

ERC -> sole choke point to the outside world.
     -> Every message received as an external RCDS-equivalent system has RDP as the logical SRC field, but it physically comes through ERC
     -> A LAN gateway where the RCDS is simply connected, but its not logical destinatino/source for anything, just a relay  node


Fields to monitor:

RDS: 
- Sent once per scan irrespective of no of detections

PTD: 
- Sents once per scan when there tracks are present
- None is sent when no tracker is present


### RDS (0x0008): the once-per-scan heartbeat

[Certain] It's sent every scan whether or not anything was detected, as a fixed 28-byte packet.

|Field|Bytes|Meaning|
|---|---|---|
|CTC|B9B8|Confirmed + manually-initiated track count|
|TTC|B11B10|Total active tracks (the doc says "including" confirmed and manual)|
|DATC / DGTC|B12 / B13|Tracks classified as aerial / ground|
|SSRC|B15B14|Plots found this scan; **0xFFFF = overload**|
|ScE|B17|0x00 OK, 0xF0 scan too fast, 0x0F scan too slow|
|StE|B18|0 OK; 1/2/3 = still awaiting location / time / comm setup (radar not fully initialised)|
|ScN|B21B20|Scan number, 1–65535 (handle wraparound)|
|RTC|B25–B22|Hours, minutes, and milliseconds within the minute (0–59999). No date.|
![[Pasted image 20261006120628.png]]
![[Pasted image 20261006121440.png|682]]


### PTD (0x0009): the track list

[Certain] It carries an 8-byte header (0xFFFF, MSGID, SRC, DST, LENGTH), then one 20-byte record per track, then a 2-byte EOM. LENGTH = 10 + (tracks × 20), so tracks = (LENGTH − 10) / 20. It's sent once per scan, and **not at all** when there are no tracks.

|Bytes|Field|Decode|
|---|---|---|
|B9B8|TRK_NAME|Track ID, 1–100 (VSR) or 1–300 (SR)|
|B10 low nibble|TRK_SRC|0 sensor, 1 BITE, 2 manual (TI), 3 manual (TV)|
|B10 high nibble|TRK_STATUS|0 FREE, 1 NEW, 2 MAN, 3 TEST, 4 CLUTTER, 5 CONFIRM, 6 TROUBLE|
|B11 bits 7–6|TGT_TYPE|1 ground, 2 aerial, 0 undefined|
|B11 bits 5–3|Subtype|Ground: 1 crawling, 2 walking, 3 running, 4 animal, 5 light vehicle, 6 heavy vehicle. Aerial: 1–2 drone, 3 helicopter, 4 large aircraft, 5 small aircraft|
|B11 bits 2–0|CFN|0 unknown, 1 permitted/test, 2 benign, 3 hostile|
|B12|GRP_STR|1 = solo, 2–8 = group size|
|B15B14|RNG|Raw × **0.5 m**|
|B17B16|AZM|Raw × 0.01°, from North|
|B19B18|SPD|Raw × 0.1 m/s (0–45 m/s)|
|B21B20|HDG|Raw × 0.01°, from North|
|B23B22|DOPLR|Signed (2's complement) × 0.1 m/s, ±45 m/s|
|B27–B24|STR|32-bit Doppler strength|
![[Pasted image 20261006120611.png]]


![[Pasted image 20261006121253.png]]
![[Pasted image 20261006121316.png]]
![[Pasted image 20261006121340.png|646]]

### Deleting a track (TD, 0x0013)

[Certain] The packet is 10 bytes, with SRC = 0x09 (you) and DST = 0x07 (RDP):

| Bytes | Field           | Value                     |
| ----- | --------------- | ------------------------- |
| B1B0  | Header          | 0xFFFF                    |
| B3B2  | MSGID           | 0x0013                    |
| B4    | SRC             | 0x09                      |
| B5    | DST             | 0x07                      |
| B7B6  | TN (track name) | 1–100 (VSR) or 1–300 (SR) |
| B9B8  | EOM             | 0x0000                    |

![[Pasted image 20261006120706.png]]