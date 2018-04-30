# PiDAR

*Group Members: Eric Vanzant (Project Owner), Tre Baker (Scrum Master), Kimberly Stenho, Rueben Golyatov, Handerson Coq, Makua Vin*

## Summary
In agriculture, we as engineers combine science and machines to help us use our environment more efficiently. At the University of Kentucky, engineering students are creating new inventive ways to help farmers and environmentalist alike. This semester our group has taken on the task of creating a platform for wireless control of a LIDAR scanner mounted on a UAV for measuring alfalfa plant height. A LIDAR stands for Light Detection and Ranging. This device sends pulses out to measure light frequencies so that the LIDAR can pick up differences in height on the ground. The first part of the project is to create a mount for the LIDAR so that it will be secure to the drone. Another part of the project is to develop a code for the LIDAR to measure the required data.


#### [*PiDAR Project Video*](https://youtu.be/mEO5Grrfrk8)


## Timeline

![Gantt Chart/Timeline](https://github.com/emvanzant/PiDAR/blob/master/docs/Gantt%20Chart.png?raw=true)


## Materials
- Raspberry Pi 3
- Micro SD Card
- Scanse Sweep Scanner (LiDAR)
- Spreading Wings S1000 (UAV)
- Mounting Bracket (Drawing Provided)
- Panhead Machine Screws (4, M2.5x0.45 mm)
- Micro USB to USB Cable
- 5V Power Converter
- Power Cables

## Assembly Procedures
Copy and dowload the provided code into the Micro SD card. Insert the SD card to the Raspberry Pi. Place the Raspberry Pi onto its provided space in the bracket. Secure the LiDAR to its place in the same bracket, with the machine screws. Attach the USB cable from the Raspberry Pi to the LiDAR. Connect the power cables to the drone's battery. Attach the power converter to the power cables. Bolt the mount onto the drone. Finally, connect the cables to the Raspberry Pi.

## Drawings

![LiDAR mount drawing](https://github.com/emvanzant/PiDAR/blob/master/docs/mount%20drawing.jpg?raw=true)
[Download LiDAR mount file (.pdf)](https://github.com/emvanzant/PiDAR/blob/master/docs/LiDAR_mount_sweepclamp_Rev.2.pdf?raw=true)     
[Download LiDAR mount parts and drawings (.zip)](https://github.com/emvanzant/PiDAR/blob/master/docs/BAE305_Sweep%20LiDAR%20mount.zip?raw=true)


## Code
     
### RUN FROM BOOT CODE

     if (GPIO.input(buttonPin)):
         # This is the script that will be called #
         os.system("python /home/pi/code/sweep/pressscan.py")
         
### BUTTON CODE
     
     import RPi.GPIO as GPIO
     import os
     import time
     
     GPIO.setmode(GPIO.BCM)

     GPIO.setup(18, GPIO.IN, pull_up_down=GPIO.PUD_UP)

     while True:
    input_state = GPIO.input(18)
    if input_state == False:
        os.system("python /home/pi/code/sweep/scantest.py")
        time.sleep(0.2)

        
### LIDAR CODE

    from __future__ import division
     import serial
     import math
     import os
     import struct
     import time

     with serial.Serial("/dev/ttyUSB0",
                    baudrate = 115200, 
                    parity=serial.PARITY_NONE,  
                    bytesize = serial.EIGHTBITS,
                    stopbits = serial.STOPBITS_ONE,
                    xonxoff = False,
                    rtscts = False,
                    dsrdtr = False) as sweep:

    print "Scanse Sweep open"
    sweep.write("ID\n")
    print "Query device information"
    resp = sweep.readline()
    print "Response: " + resp

    print "Starting scanning...",
    sweep.write("DS\n")
    resp = sweep.readline()
    assert (len(resp) == 6), "Bad data"

    status = resp[2:4]
    if  status == "00":
        print "OK"
    else:
        print "Failed %s" % status
        os.exit()
        
    # Writes a new scan iteratively by adding an integer onto the name of a previous scan #    
    while os.path.exists("sweep%s.csv" % i):
        i += 1
    log = open("sweep%s.csv" % i, "w")
    log.write("angle, distance, x, y\n")

    format = '=' + 'B' * 7

    try:
    # Collects rows of data points equal to the number in xrange plus one #
        for d in xrange (99):
            line = sweep.read(7)
            assert (len(line) == 7), "Bad data read: %d" % len(line)
            data = struct.unpack(format, line)
            assert (len(data) == 7), "Bad data type conversion: %d" % len(data)

            azimuth_lo = data[1]
            azimuth_hi = data[2]
            angle_int = (azimuth_hi << 8) + azimuth_lo
            degrees = (angle_int >> 4) + (angle_int & 15) / 16

            distance_lo = data[3]
            distance_hi = data[4]
            distance = ((distance_hi << 8) + distance_lo) / 100

            x = distance * math.cos(degrees * math.pi / 180)
            y = distance * math.sin(degrees * math.pi / 180)

            log.write("%f, %f, %f, %f\n" % (degrees, distance, x, y))
        else:
            log.close()

       # Catch Ctrl-C #
        except KeyboardInterrupt as e:
        pass        

       # Catch incorrect assumption bugs #
        except AssertionError as e:
        print e


## Test Equipment
- Complete assembly (componenets listed in the "Materials" secion)
- Monitor
- Keyboard and mouse
- 22V battery

## Test Procedure
Use the HDMI cable to connect the pi to the monitor. Connect the keyboard to the pi. and attach the power cables to the 22V battery. From the pi's home page, navigate to the sweep file, then run the pressscan.py function in the same folder. That call is path-dependent, and it's visible in the pressscan.py code. Press the button on the mount to initialize the program. When the button activates the pressscan.py code, the LiDAR begins collecting data, saving it in bundles of 100 rows as a .csv file. The program adds an integer onto the end of the name sweep.csv if it alreay exists. Press the button a second time to stop the program.


## Test Results
Below is a sample of the data collected and stored on the SD card after a successful trial.

        angle, distance, x, y

     263.687500, 2.050000, -0.225400, -2.037571
     266.062500, 2.020000, -0.138710, -2.015232
     268.625000, 1.980000, -0.047512, -1.979430
     271.187500, 1.950000, 0.040412, -1.949581
     273.750000, 1.930000, 0.126228, -1.925868
     276.125000, 1.910000, 0.203793, -1.899097
     278.687500, 1.890000, 0.285475, -1.868316
     281.250000, 1.870000, 0.364819, -1.834068
     283.812500, 1.860000, 0.444066, -1.806213
     286.562500, 1.870000, 0.533064, -1.792412
     288.750000, 1.850000, 0.594663, -1.751821
     291.000000, 1.830000, 0.655813, -1.708452
     293.562500, 1.850000, 0.739536, -1.695755
     296.125000, 1.840000, 0.810209, -1.652017
     298.687500, 1.860000, 0.892860, -1.631687
     301.375000, 1.880000, 0.978798, -1.605103
     303.937500, 1.900000, 1.060748, -1.576329
     316.750000, 0.010000, 0.007284, -0.006852
     329.562500, 0.090000, 0.077596, -0.045594
     342.375000, 0.150000, 0.142959, -0.045418
     351.937500, 0.190000, 0.188122, -0.026648
     ...
     
## Discussion
The program is able to operate normally and performs the intended task. A code (pressscan.py) starts upon booting (by editing /etc/rc.local) which, when the button on the mount is pressed, calls on another program (scantest.py) to run a scan with the LiDAR, which saves data iteratively as a .csv file. This file includes angle and distance, and can be exported.

### Design
Design considerations for the physical, 3-D printed mount and assembly include:
- Minimum mount thickness: 0.1 in.
- Additional material added to base considering stress points
- Depressions for hex nuts to maintain consistency with exisitng mount components
- Neat and compact storage of components and wiring to prevent interference during flight

### Testing
The program is able to operate normally and performs the intended task. A code (pressscan.py) starts upon booting (by editing /etc/rc.local) which, when the button on the mount is pressed, calls on another program (scantest.py) to run a scan with the LiDAR, which saves data iteratively as a .csv file. This file includes angle and distance, and can be exported.

## LiDAR User Manual
[Scanse Sweep v1.0 User Manual](https://github.com/emvanzant/PiDAR/blob/master/docs/Sweep_user_manual.pdf)
