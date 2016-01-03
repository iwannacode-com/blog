PHOTON_ID=YOUR_PHOTON_ID

all: compile flash

compile:
	particle compile photon ./photon --saveTo app.bin

usbflash:
	sudo particle flash --usb app.bin

flash:
	particle flash $(PHOTON_ID) app.bin

monitor:
	sudo particle serial monitor
