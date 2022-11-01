## Overview
We're greeted with the file ```chall.png``` attached in the challenge. The image contains Kiryu (no idea who that is, to be honest) wearing a green headset in a weird purple lighting.

The image is working just fine in the image viewer, so I don't think there should be anything wrong with the file structure. Because of that, my first approach is to try and open ```chall.png``` in a steganography solver. In this case, I used [stegsolve.jar](http://www.caesum.com/handbook/Stegsolve.jar) by Caesum.

Applying ```Blue plane``` filter into the image shows some fragments of flag. It's right above Kiryu's head, written in curve. Turns out, applying ```Blue plane 3``` filter in stegsolve gives us the clearest view of the flag.

##### solved by nabilmuafa
