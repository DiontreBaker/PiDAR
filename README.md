*Group Members: Eric Vanzant (Project Owner), Tre Baker (Scrum Master), Kimberly Stenho, Rueben Golyatov, Handerson Coq, Makua Vin*

## Summary
In agriculture, we as engineers combine science and machines to help us use our environment more efficiently. At the University of Kentucky, engineering students are creating new inventive ways to help farmers and environmentalist alike. This semester our group has taken on the task of creating a platform for wireless control of a LIDAR scanner mounted on a UAV for measuring alfalfa plant height. A LIDAR stands for Light Detection and Ranging. This device sends pulses out to measure light frequencies so that the LIDAR can pick up differences in height on the ground. The first part of the project is to create a mount for the LIDAR so that it will be secure to the drone. Another part of the project is to develop a code for the LIDAR to measure the required data.

## Timeline

![Gantt Chart/Timeline](https://github.com/emvanzant/PiDAR/blob/master/docs/Gantt%20Chart.png)

## Materials
## Assembly Procedures
### Drawings

![LiDAR mount drawing](https://github.com/emvanzant/PiDAR/blob/master/docs/mount%20drawing.jpg)
[Download LiDAR mount file (.pdf)](https://github.com/emvanzant/PiDAR/blob/master/docs/LiDAR_mount_sweepclamp_Rev.2.pdf)     
[Download LiDAR mount file (.dwg)](https://github.com/emvanzant/PiDAR/blob/master/docs/LiDAR_mount_sweepclamp_Rev.2.dwg)

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
## Test Procedures
## Test Results
## Discussion
### Design
### Testing
## LiDAR User Manual
[Scanse Sweep v1.0 User Manual](https://github.com/emvanzant/PiDAR/blob/master/docs/Sweep_user_manual.pdf)
_________________________
## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/DiontreBaker/PiDAR/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/DiontreBaker/PiDAR/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
