
Needed commands:
1. ICR - Auto
2. Auto Exposure - Auto (0)
3. Gain - 1
4. Shutter
5. Iris
6. Brightness
7. Digital Zoom
8. Camera power 
9. Camera Restore default
10. One push focus
11. step by step focus
12. Select focus mode - Auto (0)
13. Zoom presets
14. Match Zoom with IR - ON (1)
15. Step by Step zoom
16. Goto zoom position
17. Query FOV
18. Camera BIT(Built in Test) Status




Commands::

STOP ZOOM: printf '\x81\x01\x04\x07\x00\xFF' | sudo tee /dev/ttyUSB0 > /dev/null


