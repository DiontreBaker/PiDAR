*Group Members: Eric Vanzant (Project Owner), Tre Baker (Scrum Master), Kimberly Stenho, Rueben Golyatov, Handerson Coq, Makua Vin*

## Summary
In agriculture, we as engineers combine science and machines to help us use our environment more efficiently. At the University of Kentucky, engineering students are creating new inventive ways to help farmers and environmentalist alike. This semester our group has taken on the task of creating a platform for wireless control of a LIDAR scanner mounted on a UAV for measuring alfalfa plant height. A LIDAR stands for Light Detection and Ranging. This device sends pulses out to measure light frequencies so that the LIDAR can pick up differences in height on the ground. The first part of the project is to create a mount for the LIDAR so that it will be secure to the drone. Another part of the project is to develop a code for the LIDAR to measure the required data.

## Timeline

[Gantt Chart/Timeline](https://github.com/emvanzant/PiDAR/blob/master/docs/Gantt%20Chart.png?raw=true)


## Materials
## Assembly Procedures
### Drawings

[LiDAR mount drawing](https://github.com/emvanzant/PiDAR/blob/master/docs/mount%20drawing.jpg?raw=true)
[Download LiDAR mount file (.pdf)](https://github.com/emvanzant/PiDAR/blob/master/docs/LiDAR_mount_sweepclamp_Rev.2.pdf?raw=true)     
[Download LiDAR mount file (.dwg)](https://github.com/emvanzant/PiDAR/blob/master/docs/LiDAR_mount_sweepclamp_Rev.2.dwg?raw=true)


### Code
     
BUTTON CODE
     
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

        
LIDAR CODE

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
    while os.path.exists("sweep%s.csv" % i):
        i += 1
    log = open("sweep%s.csv" % i, "w")
    log.write("angle, distance, x, y\n")

    format = '=' + 'B' * 7

    try:
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

       # Catch Ctrl-C
        except KeyboardInterrupt as e:
        pass        

       # Catch incorrect assumption bugs
        except AssertionError as e:
        print e


## Test Equipment


## Test Procedure
Navigate to the sweep file, then run the pressscan.py function in the same folder. That call is path-dependent, and it's visible in the pressscan.py code. Press the button on the mount to initialize the program. When the button activates the pressscan.py code, the LiDAR begins collecting data, saving it in bundles of 100 rows as a .csv file. The program adds an integer onto the end of the name sweep.csv if it alreay exists. Press the button a second time to stop the program.

## Test Results
The program is able to operate normally and performs the intended task. A code (pressscan.py) starts upon booting (by editing /etc/rc.local) which, when the button on the mount is pressed, calls on another program (scantest.py) to run a scan with the LiDAR, which saves data iteratively as a .csv file. This file includes angle and distance, and can be exported.

## Discussion
### Design
### Testing
## LiDAR User Manual
[Scanse Sweep v1.0 User Manual](https://github.com/emvanzant/PiDAR/blob/master/docs/Sweep_user_manual.pdf)
_________________________

