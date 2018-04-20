*Group Members: Eric Vanzant (Project Owner), Tre Baker (Scrum Master), Kimberly Stenho, Rueben Golyatov, Handerson Coq, Makua Vin*

## Summary

![Gantt Chart/Timeline](https://github.com/emvanzant/PiDAR/blob/master/docs/Gantt%20Chart.png)

## Materials
## Assembly Procedures
### Drawings

![LiDAR mount drawing](https://github.com/emvanzant/PiDAR/blob/master/docs/mount%20drawing.jpg)
[LiDAR mount dwg. file](https://github.com/emvanzant/PiDAR/blob/master/docs/LiDAR_mount_sweepclamp_Rev.2.dwg)

### Code
  #!/usr/bin/env python

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

        #-----------------------------------------------------------------------
        # Missing here is stopping the scanning - it will still be running next 
        # time code is initiated and it all gets very messy / confusong separating
        # binary data from ASCII command / response.  Really need to do a subset of
        # the finally: branch below.
        #-----------------------------------------------------------------------
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
    	log.close()
## Test Equipment
## Test Procedures
## Test Results
## Discussion
### Design
### Testing
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
