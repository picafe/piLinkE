# July 24, 2026: Started the schematic and sourced symbols

After watching [this](https://www.youtube.com/watch?v=FJ3fFXFcsLk) video on youtube, I thought this would be a nice project to do as I thought the WCHLinkE is quite expensive for what it is on Aliexpress (~$7). At the same time, I wanted to learn how to use Altium and be comfortable using it more, so I decided it would be good practice.

After some very brief research, I found other similar projects on the Chinese version of Oshwstar (Oshwstar), mostly the following which had almost the same overall design:
https://web.archive.org/web/20240525210835/https://oshwhub.com/fanhuacloud/wch-linke/
https://oshwhub.com/huashuo0804/wch-linke

I also found this, which will let me use the WCHLinkE as a high speed ARM DAP Link debugger, which is a nice feature to have:
https://github.com/prosper00/CH32V305-DAPLink-HS

I opened up Altium and began trying to place all the symbols on the board, since this would very much be the most tedious part (sourcing symbols).

For USB-C, I wanted to be sure I had the same FP as the component I was getting, so I used [a tool]() to convert LCSC/EasyEDA components into Altium symbols and FPs and manually copy pasted the component from the library into my own library for the project. However, I very much dislike the way EasyEDA lays out their USB-C symbol as you can see:

![alt text](images/X2_ZH0H0JdxrJ.png)

I ended up opening another window of KiCad and opening up the USB-C symbol from there and copying overall layout of the symbol in Altium. I used SamacSys Altium LL to get the CH32 symbol, but again it was fairly messy when I tried to use it, so I ended up doing the same process of copying the layout of the KiCad symbol.

I also had to manually add and adjust the 3D model for the USB-C connector, since the one from  was not quite right.

![alt text](images/X2_NQhasIx2KL.png)

I also tried importing directly from KiCad into Altium with the KiCad import tool plugin, but it maintained a lot of the KiCad formatting (fonts, line widths, text sizes, etc.), which didn't look too good in Altium.

After some work, I finally got most of the components placed:

![alt text](images/X2_yZOQvRJLsW.png)

**Total time spent: 3.5 hours**

# July 25, 2026: Placed components and configured design rules

I started on the passives, which were just generic components from my org's component library which I added LCSC number properties to after finding a suitable LCSC part for each in the schematic. For the custom parts, I used the property editor in the schematic library and for generic parts, I edited the properties in the schematic.

![alt text](images/X2_hZkCKR5rUM.png)


The footprints I mostly used the default for the symbols, and the footprints in my org's library for the passives after adding the values to each.

![alt text](images/X2_OQCKwfkN0V.png)

I also cleaned up the schematic as you can see, and finally ran an ERC check to make sure I didn't miss anything.

I then ran the ECO to apply the changes to the PCB, which had no errors (yay!).

![alt text](images/X2_ZcdWW1e9Vj.png)

![alt text](images/X2_sd3yh0AalE.png)

The first thing I noticed after grouping the components together and zooming in was the rule violations on the TSSOP and USB-C pads (green highlighted) which were caused by the spacing between the pads. I had never setup a new project before, so after some searching it seemed that default clearances were set to defaults that are uncommon (10 mil for every clearance type), so I had to explore the rules manager to adjust the clearances.

![alt text](images/X2_eN1riPqIEi.png)

![alt text](images/X2_GE4X17RYtz.png)

After doing that, I started component placement:

![alt text](images/X2_wHSpiim2PM.png)

I had to adjust the default via diameter and pad radius since the defaults were surprisingly large.

This is what I had roughly after getting used to navigating Altium:

![alt text](images/X2_76KszubPjD.png)

One thing I had to fix was the keepout area for some components. For some reason, the keepout clearance included the silkscreen of the component (since I guess it's a primitive layer), so I had to manually make the keepout smaller for the LED pads and change the pin indicator to be under the component.

![alt text](images/X2_3x1mdOQA5V.png)

This took quite a while since I had to hunt down the footprint in my org's library and adjust it there, but after trying to do that for a while, I just gave up and updated the primitive on the board directly (unlocking primitive layers to edit them).

![alt text](images/hjAUqkuWCg.png)

Then, I decided to setup the impedance control for the USB differential pairs, which I had to search up a tutorial how to do. Since Altium does it's own calculations for the trace width and gap based on board spec, I had to find the exact JLC board spec instead of using their impedance calculator. Overall, I ended up with the following, which is almost the same as the impedance calculator's results:

![alt text](images/X2_ATmdlMhT1S.png)

This was the guide I followed and it was very helpful: https://www.youtube.com/watch?v=ULK_fRNbZcs

**Total time spent: 5 hours**

# July 28, 2026: Routed most of the board

I started (and basically finished) most of the routing for the board, although there is still quite a bit of work to be done. I still need to route the power traces, planes, and some other minor things. I also had to search on how to adjust the board outline, but it was fairly simple. I did this while crammed in the middle seat of a plane so I wasn't the most efficient with my trackpad.

![alt text](images/X2_bTGnkn2e9A.png)

**Total time spent: 3 hours**

# August 14, 2026: Routed power and filled copper planes

I started with routing the power traces, and then moved on to planes which was a bit harder than I thought. I got ground planes to work (or at least I thought) but local polygons inside the ground plane didn't seem to fill. After some searching, I used the polygon manager to order based on fill priority, change the property to fill over all items with the same net, and then force refill all the polygons. I also used this to duplicate the GND plane and move it to the GND layer and bottom layer.

![alt text](images/X2_ZH0H0JdxrJ.png)

After that, the PCB finally looked almost finished:

![alt text](images/X2_nBD9jmYK47.png)

**Total time spent: 1.2 hours**

# August 15, 2026: Ran DRC and added silkscreen

I ran DRC for the first time, which was quite interesting. Other than a few stray traces, the only errors seemed to be spam from me not setting proper values in the rules manager correctly, except I had pretty much no idea what the rule violations were. 

![alt text](images/X2_NUOUpmMQZQ.png)

The main issue was minimum solder mask silver constraint, which turns out to represent the width of narrowest gap between 2 regions of area without solder mask. Since I had vias in my planes to connect to ground and power planes, it basically meant I had to adjust the constraint to be smaller and move the vias a bit further away, although the vias will be tented anyways so it shouldn't be a problem (apparently from a quick search there's no option to set this in Altium that isn't convoluted). There were also violations in areas where silkscreen was too close to areas of exposed board, which I just decided to ignore since it's not a big deal.

![alt text](images/X2_mLiMObf77l.png)

After that, I started adding things on the silkscreen; some labels to the board for the 10 pin JST connector to help me identify the pins of the WCHLinkE and orpheus drinking boba. Lastly, I fixed some minor things such as consistent track widths (I had some data tracks that were 0.3mm wide instead of 10mil, which was the default and which I left it at), and rounding the board edges.

![alt text](images/X2_ORDCPWgpSc.png)

**Total time spent: 1.5 hours**

# August 16, 2026: Exported production files

Exporting prod files (gerber, drill, bom csv) was quite similar to how it is on KiCad if you do it manually (without the JLCPCB production file extension). Took me a bit to do since JLC doesn't support gerber X2 exports, which is what the youtube guide told me to do. 

![alt text](images/X2_VaOR04Ht9g.png)

![alt text](images/X2_NzWxSeaZGV.png)

**Total time spent: 0.4 hours**

# August 17, 2026: 

One thing I forgot to do earlier was add stitching vias for the GND plane, but after doing that I realized that my vias had thermal reliefs (which idk how I didn't really notice until now). 

I looked into rules manager and first found this under advanced plane connect style, which I adjusted but after refilling polygons, it was still the same.

![alt text](images/X2_x2YcmgxiNB.png)

I forgot earlier that I added plane connects by net since I wanted to have entirely solid connects for vias in internal layers, so the way I did it might not have been the best way, since I could have just use the advanced plane connect style to have solid connects for vias in all polygons/layers.

![alt text](images/X2_K2JUIRLhdU.png)

Anyways, this is what it looks like now:

![alt text](images/X2_75cauMuAWf.png)

I breifly ran DRC, and then re-exported the production files.

**Total time spent: 0.5 hours**
