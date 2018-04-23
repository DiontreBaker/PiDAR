*Group Members: Eric Vanzant (Project Owner), Tre Baker (Scrum Master), Kimberly Stenho, Rueben Golyatov, Handerson Coq, Makua Vin*

## Summary
In agriculture, we as engineers combine science and machines to help us use our environment more efficiently. At the University of Kentucky, engineering students are creating new inventive ways to help farmers and environmentalists alike. This semester our group has taken on the task of creating a platform for control of a LiDAR scanner mounted on a UAV for measuring plant height. A LiDAR stands for Light Detection and Ranging. This device sends pulses out to measure light frequencies so that the LIDAR can pick up differences in height on the ground. The first part of the project is to create a mount for the LIDAR so that it will be secure to the drone; another part is to develop a code for the LIDAR to measure the required data.

## Timeline
![Gantt Chart/Timeline](https://github.com/emvanzant/PiDAR/blob/master/docs/Gantt%20Chart.png?raw=true)

## Materials
- Raspberry pi 3
- Micro SD Card
- Scanse Sweep Scanner (LiDAR)
- Spreading Wings S1000 (UAV)
- Mounting bracket (Drawing provided)
- Panhead machine Screws (4, M2.5x0.45 mm)
- Micro USB to USB Cable
- 5V power converter
- Power cables

## Assembly Procedures
Copy and dowload the provided code into the Micro SD card. Then, insert the SD card to the raspberry pi. Place the raspberry pi into its provided space in the bracket. Also secure the LiDAR to its place in the same bracket, with the machine screws. Attach the USB cable from the raspberry pi to the LiDAR. Connect the power cables to the drone's battery. Then, attach the power converter to the power cables. Bolt the mount onto the drone. Finally, connect the cables to the raspeberry pi.

### Drawings
![LiDAR mount drawing](https://github.com/emvanzant/PiDAR/blob/master/docs/mount%20drawing.jpg?raw=true)
[Download LiDAR mount file (.pdf)](https://github.com/emvanzant/PiDAR/blob/master/docs/LiDAR_mount_sweepclamp_Rev.2.pdf?raw=true)     
[Download LiDAR mount file (.dwg)](https://github.com/emvanzant/PiDAR/blob/master/docs/LiDAR_mount_sweepclamp_Rev.2.dwg?raw=true)

### Code
     
BUTTON CODE
    import RPi.GPIO as GPIO
      import time
      import os

    #adjust for where your switch is connected
    buttonPin = 17
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(buttonPin,GPIO.IN)

    while True:
      #assuming the script to call is long enough we can ignore bouncing
     if (GPIO.input(buttonPin)):
        #this is the script that will be called (as root)
        os.system("python /home/pi/start.py")
LIDAR CODE

    from __future__ import division
    import serial
    import math
    import os
    import struct
    

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

    log = open("sweep.csv", "wb")
    log.write("angle, distance, x, y\n")

    format = '=' + 'B' * 7

    try:
        while True:
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

    #--------------------------------------------------------------------------
    # Catch Ctrl-C
    #--------------------------------------------------------------------------
    except KeyboardInterrupt as e:
        pass        

    #--------------------------------------------------------------------------
    # Catch incorrect assumption bugs
    #--------------------------------------------------------------------------
    except AssertionError as e:
        print e

    #--------------------------------------------------------------------------
    # Cleanup regardless otherwise the next run picks up data from this
    #--------------------------------------------------------------------------
    finally:
    	print "Stop scanning"
    	sweep.write("DX\n")
    	resp = sweep.read()
    	print "Response: %s" % resp
      # Save file under iterative names
    	log.close()
    import time
    if os.path.exists('result.png'):
      plt.savefig('result_{}.png'.format(int(time.time())))
    else:
      plt.savefig('result.png')
      __________________________________
      import RPi.GPIO as GPIO
    import time
    import os

    #adjust for where your switch is connected
    buttonPin = 17
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(buttonPin,GPIO.IN)

    while True:
      #assuming the script to call is long enough we can ignore bouncing
     if (GPIO.input(buttonPin)):
        #this is the script that will be called (as root)
        os.system("python /home/pi/start.py")

## Test Equipment
No additional equipment is required. Please follow test procedure.
## Test Procedures
Initiate the program by pressing the button on the mount.
## Test Results
## Discussion
### Design
### Testing
_________________________
