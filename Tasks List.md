
github Token: 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

IN Bangalore:

- [ ] Finish Tonbo model
- [ ] Finish automating the integration -> ignore responses for now
- [ ] Get a Hoodie
- [ ] Innerwears

- [ ] Create a [[Tonbo model]] + T for Sensor level (RGB, IR, SWIR)  --------------P1-------  Oct15/16
	- [ ] Decide on [[Dataset]]
- [x] Create [[Drone Model]] model+ T for drone level (RGB) ----------------------P0---------------- Oct15
- [ ] Establish pipeline between obtro sens to AI Node to GCS ---------- Oct17-20 (Based on API, Oct ? exp)
- [x] Reduce pipeline latency ----------------------------Parallel with P0------------------Oct15
- [ ] Bangalore Test ----------------------------Oct21 - 27/28----------testing the pipeline


- [x] Ask AK abt github code upload on perception
	- [ ] Add it to mds_ground folder in main branch 
- [x] Establish [[Jetson Pipeline]] ---------------------------- E5
- [x] Establish connection from Jetson to HM30 ------------------------------- E4


Oct 15:
- [x]  Finish setting up the venv
	- [x]  Set up a jetpipe_setup.sh for automated entire pipeline setup
- [x] Expand Jetson RAM, addional 16GB swap memory



- [ ] Jetson RTSP stream
- [ ] TATA 




RTSP:

gst_out = (

'appsrc name=appsrc is-live=true format=time do-timestamp=true '

f'caps=video/x-raw,format=BGR,width={W},height={H},framerate={FPS}/1 ! '

'videoconvert ! video/x-raw,format=I420 ! '

'x264enc bitrate=4000 tune=zerolatency key-int-max=30 ! '

'h264parse ! rtph264pay name=pay0 pt=96 config-interval=1 '

)

this one didnt;' work, same stream timed out error, but it worked after increasing bitrate to 4000 and removed tune=zerolatency, but the vid is still very choppy, and it only works in UDP, not TCP




[[FIELD TRIAL CHECK LIST]]






/:  Bus 08.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/0p, 5000M
/:  Bus 07.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/1p, 480M
    |__ Port 1: Dev 2, If 0, Class=Video, Driver=uvcvideo, 480M
    |__ Port 1: Dev 2, If 1, Class=Video, Driver=uvcvideo, 480M
    |__ Port 1: Dev 2, If 2, Class=Application Specific Interface, Driver=, 480M
/:  Bus 06.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 10000M
/:  Bus 05.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 480M
    |__ Port 1: Dev 2, If 0, Class=Human Interface Device, Driver=usbhid, 12M
/:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 10000M
/:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 480M
    |__ Port 1: Dev 2, If 0, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 1: Dev 2, If 1, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 2: Dev 3, If 0, Class=Wireless, Driver=btusb, 480M
    |__ Port 2: Dev 3, If 1, Class=Wireless, Driver=btusb, 480M
    |__ Port 2: Dev 3, If 2, Class=Wireless, Driver=, 480M
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 20000M/x2
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 480M




IP: 192.168.1.200
uer: admin
pwd: sata@124


Siya pwd: V_mlaxmy@2020