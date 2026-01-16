

**To transfer file from Laptop to Jetson : **

```
scp path/to/laptop skeyebots_hw@192.168.1.8:~/
```

History:
```
history
```

Iterate over a command every sec:

```
watch -n 1 <command>
```

Record rtsp stream:
```
ffmpeg -i "rtsp_link" -c copy name.mp4
```


To run local model qwen2.5:14b through aider - read only - able to access directory:
```
export OLLAMA_API_BASE=http://localhost:11434
aider --model ollama_chat/qwen2.5-coder:14b --map-tokens 1024
```
- Usefull Aider commands:
	- /add file_path - adds that file into the chat, (read-only), and can answer questions about that file selectively
	- /map - to check if the Repo map is created
		- repo map- high level structuer, the model will just look at function names, and not in detail
	- /read - Another fail proof arguements to make sure aider doesnt edit the code and only reads the added files



To mount another os:

FInd its partition using :
```
lsblk -f
or 
sudo fdisk -l
```

Then, for mounting:

```
sudo mkdir -p /mnt/ubuntu24
sudo mount /dev/sdXn /mnt/ubuntu24
for i in /dev /dev/pts /proc /sys /run; do sudo mount -B $i /mnt/ubuntu24$i; done
```


To get into the terminal:
```
sudo chroot /mnt/ubuntu24
```