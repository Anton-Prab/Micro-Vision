# Micro Vision on Sparkfun Edge 

Micro Vision is an example project which shows how you can use Tensorflow Lite to run a 250 kilobyte neural network to recognize people in the images provided. It is designed to run on Sparkfun Edge micro controller. 
Although Sparkfun Edge has the camera interface (OV7670), the support or the api to interface with the camera module is not available. So for this example, inference is done on the images rather than the frames from the camera. 
## Table of contents
-   [Requirements](#requirements)
-   [Setup](#setup)
-   [Compiling & flashing](#compiling-the-code)
-   [Running](#running)
-   [Debugging](#debugging)
-   [To-Do](#to-do)

## Requirements:
-   Git
-   Python3
-   Pip for Python3
-   Make 4.2.1 or higher
-   Tensorflow repository

## Setup :
Download the Tensorflow repository and extract it
git clone https://github.com/tensorflow/tensorflow.git

### Install necessary python packages
	pip3 install pycrypto pyserial –user
Convert Image to byte data
Take an image and convert it into 96*96 bmp 24 bit 
Steps to get image into data
1. Copy the person_image_data.cc person_image_data.h as two  new files say  new_person_image_data.cc and new_person_data.h Be sure that h file and cc file names are same and reference inside the H files reflect the correct names
Content of header file looks like this


	#ifndef TENSORFLOW_LITE_EXPERIMENTAL_MICRO_EXAMPLES_MICRO_VISION_NEW_PERSON_IMAGE_DATA_H_
	#define TENSORFLOW_LITE_EXPERIMENTAL_MICRO_EXAMPLES_MICRO_VISION_NEW_PERSON_IMAGE_DATA_H_

	#include <cstdint>

	extern const int g_new_person_data_size;
	extern const uint8_t g_new_person_data[];

	#endif  
	

Partial Content of the imagedata.cc looks like this

	#include "tensorflow/lite/experimental/micro/examples/micro_vision/new_person_image_data.h"

	#include "tensorflow/lite/experimental/micro/examples/micro_vision/model_settings.h"

	const int new_person_data_size = 27648;
	const uint8_t g_new_person_data[new_person_data_size] = {
	  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x80, 0x00, 0x00, 0x80, 0x00, 0x00
	       2) xxd -s 54 filename.bmp test.out
	Copy the contents of the out file and put it inside the new_person_image_data.cc, below the line 
	const uint8_t g_new_person_data[new_person_data_size] = {

	3) From the extracted microvision example, change the file – main.cc 
	In stead of getting file from provider, we need to provide a file data here.
	Below the while loop, change the content to reflect the newly generated data
	while (true) {
	    // Get image from provider.
		  //const uint8_t* person_data = g_person_data;
		  const uint8_t* person_data = g_new_person_data;
		    for (int i = 0; i < input->bytes; ++i) {
		      input->data.uint8[i] = person_data[i];
		    }

## Compiling the code
After a while, all files will get downloaded. Now run the command

	make -f tensorflow/lite/experimental/micro/tools/make/Makefile TARGET=sparkfun_edge micro_vision_bin

Check whether bin file is there in the folder

	tensorflow/lite/experimental/micro/tools/make/gen/sparkfun_edge_cortex-m4/bin

If you are funning for the first time, change the name of file to sign

	cp tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/tools/apollo3_scripts/keys_info0.py \
	tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/tools/apollo3_scripts/keys_info.py
Now geenreate signed bin

	python3 tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/tools/apollo3_scripts/create_cust_image_blob.py \
	--bin tensorflow/lite/experimental/micro/tools/make/gen/sparkfun_edge_cortex-m4/bin/micro_vision.bin \
	--load-address 0xC000 \
	--magic-num 0xCB \
	-o main_nonsecure_ota \
	--version 0x0


Then run the ommand

	python3 tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/tools/apollo3_scripts/create_cust_wireupdate_blob.py \
	--load-address 0x20000 \
	--bin main_nonsecure_ota.bin \
	-i 6 \
	-o main_nonsecure_wire \
	--options 0x1

Then flash the bin to device, after setting device and baudrate
Identify the DEVICENAME

	ls /dev/tty* 
	export DEVICENAME=put your device name here

	export BAUD_RATE=921600

Hold button 14 and reset together, release reset..
Still holding the button 14 , issue this command below

	python3 tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/tools/apollo3_scripts/uart_wired_update.py \
	-b ${BAUD_RATE} ${DEVICENAME} \
	-r 1 \
	-f main_nonsecure_wire.bin \
	-i 6

Once you see “reset done”, press reset key

## Running

Use screen command to see the output

	screen /dev/ttyUSB0  115200

You will see the score of person found and noperson found

## Debugging Image Capture
When the sample is running, check the LEDs to determine whether the inference is running correctly.The orange LED indicates that no person was found, and the green LED indicates a person was found. The red LED should never turn on, since it indicates an error.

## To-Do
-   Creating a code which automatically creates the image.cc from the image
-   Better and more infernece on images
