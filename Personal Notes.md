

Game folder: home/.Games/

Setup Folder: home/.Notes


TO hide a folder: "Add . at the beginning of its name"




Lutris step by step guide:


1) Add locally Installed Game
2) In Game Info 
	1) Write a Name, 
	2) Runner  -> Wine
3) Game Options:
	1) Create an empty directory in .Games/ folder
	2) Select that as the wine prefix
4) Runner options
	1) Wine Version -> GE-Proton10-21
5) Save
6) Select 'Run ECE inside Wine Prefix' option
7) Navigate to the FG-repack and run the setup File
	1) Installation location -> C drive (just  c drive)
	2) Run/select all VScode C++, and update directrix too, just in case
8) Right click on the game
9) Select 'configure'
10) Navigate to 'Game options'
	1) Executable -> The final application that runs the game, which should be found inside the wine prefix, C drive
11) Save and play the game


To get Full screen:

1) Gamescope is now enabled so the options to get fulle screen:
	1) Game res -> 1920x1200
	2) Output res -> 1920x1200
	3) Window mode -> Full screen

To install Gamescope:
Gamescope version:
git clone --depth 1 --branch 3.12.3 https://github.com/ValveSoftware/gamescope.git
cd gamescope
git submodule update --init --recursive
meson setup build --buildtype=release
sudo ninja -C build install


Possible missing packages error during meson build, and the resp installation req:
	sudo apt install libbenchmark-dev
	sudo apt install glslang-tools
	sudo apt install libsdl2-dev
	sudo apt install hwdata
	sudo apt install libxmu-dev libxres-dev libxrandr-dev libxinerama-dev libxcursor-dev libx11-dev libxxf86vm-dev




Vid recovery:

WITH INTERNET CONNECTION:
sudo add-apt-repository universe
sudo apt update
sudo apt install testdisk
sudo photorec
