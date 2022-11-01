## Overview

Let's extract the ```log.tar.xz``` first.

We got a ```log.pcap``` file and ```ssl.log``` file.

We need to input the ```ssl.log``` file first in Wireshark to decrypt the pcap file.

To do that,  go to ```Edit > Preferences > Protocols > TLS``` and put the ```ssl.log``` in the ```(Pre)-Master-Secret log filename```.

After supplying the log file to Wireshark, we can see some HTTP packets.

In the HTTP info, there are 2 things that caught my attention.

```
Range: bytes=2055-2157/4410\r\n
[Request URI: https://storage.compfest.id/files/part_2.png]
```

This means there are 2 png files (part_1.png and part_2.png) and each png file is corrupted in some sort of way.

## Solution
Let's try to export the png in ```File > Export Objects > HTTP```.

In the Export Objects window, we see multiple part_1.png's and part_2.png's.
This means that the png's are broken in chunks!

Let's export all of them!
And since we know the byte ranges of the png chunks from the HTTP info, let's also export the HTTP info in ```File > Export Packet Dissections > As Plain Text...```.

Now we have the png chunks and the HTTP info, let's make a game plan.

The plan is to identify where each png chunks belong in the final png file. And then we reconstruct the final png file using the png chunks that we have! 

Let's try it with the png_1.png's first.

First step is to rename the png chunks to their byte ranges.

We can use a python script to do this!

But first, we need to extract the byte ranges from the HTTP info txt file.

```python
txt = "httpinfo_1.txt"
match = re.findall("bytes \d{1,4}-\d{1,4}", txt.read())
for x in match:
    match[match.index(x)] = re.sub("bytes ", "", match[match.index(x)])
```

After we get the byte ranges, let's rename the png's!

```python
for i in range(len(byte_ranges)):
    try:
        os.rename(f"part_1({i}).png" ,f'{byte_ranges[i]}.png')
    except FileNotFoundError:
        os.rename(f"part_1.png" ,f"part_1({i}).png")
        os.rename(f"part_1({i}).png" ,f'{byte_ranges[i]}.png')
```

Now that all the png's are renamed to their respective byte ranges, we can now reconstruct the png file!
But there is one problem, our byte range array is not sorted. We need to sort the array first!
```python
for x in byte_ranges:
    byte_ranges[byte_ranges.index(x)] = byte_ranges[byte_ranges.index(x)].split("-")
for i in range(len(byte_ranges)):
    for j in range(len(byte_ranges) - i - 1):
        if int(byte_ranges[j][0]) > int(byte_ranges[j + 1][0]):
            byte_ranges[j], byte_ranges[j + 1] = byte_ranges[j + 1], byte_ranges[j]
for x in byte_ranges:
    byte_ranges[byte_ranges.index(x)] = "-".join(byte_ranges[byte_ranges.index(x)])
```

Now is the time to reconstruct!
```python
for x in byte_ranges:
    png_chunk = open(f"{x}.png", "rb")
    final_png = open("part_1.png", "ab")
    final_png.write(png_chunk.read())
final_png.close()
```

And we found the first part of the flag!

[![part-1.png](https://i.postimg.cc/h4msp99N/part-1.png)](https://postimg.cc/5jx8tQKm)

And now we need to do it to the part_2.png's!

After getting the part_2 and putting them together, you will get the flag!

##### Solved by kawijayaa
