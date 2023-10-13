
# Dumb-Down-Pwnagotchi-Guide

## Introduction

![A picture of the Pwnagotchi](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/2245fbbd-dd6e-4fb5-8ad7-cfb2ebbe11a8)

> Pwnagotchi is an A2C-based “AI” powered by bettercap and running on a Raspberry Pi Zero W that learns from its surrounding WiFi environment in order to maximize the crackable WPA key material it captures (either through passive sniffing or by performing deauthentication and association attacks). This material is collected on disk as PCAP files containing any form of handshake supported by hashcat, including full and half WPA handshakes as well as PMKIDs.

At least that's what the website says, admittedly there are some better more effecient ways to collect handshakes, especially since the actual website is somewhat abondon with no updates except from the loving community that took in the adorable thing. Inspired from the Tamagotchi toy, the Pwnagotchi follows the same premise but instead of eating treats, you are eating packets, and **you clean up the PCAP files**.

The website itself can be useful for this as some information can still be use as well as understand things for more clarity. So here's the [site](https://pwnagotchi.ai/).

## How Does It work, and how it communicates?

### WiFi collecting
The Pwnagotchi essentially uses AI to tune it's paramenters to *get better at pwning WiFi things in the enviroments you expose it to*. You can even use tune it but it's advise not to as your new toy would still need to learn. The way the AI really learns is to tally up a points via this equation coded in the system which is known as a A2C agent

```
# state contains the information of the last epoch
# epoch_n is the number of the last epoch
tot_epochs = epoch_n + 1e-20 # 1e-20 is added to avoid a division by 0
tot_interactions = max(state['num_deauths'] + state['num_associations'], state['num_handshakes']) + 1e-20
tot_channels = wifi.NumChannels

# ideally, for each interaction we would have an handshake
h = state['num_handshakes'] / tot_interactions
# small positive rewards the more active epochs we have
a = .2 * (state['active_for_epochs'] / tot_epochs)
# make sure we keep hopping on the widest channel spectrum
c = .1 * (state['num_hops'] / tot_channels)
# small negative reward if we don't see aps for a while
b = -.3 * (state['blind_for_epochs'] / tot_epochs)
# small negative reward if we interact with things that are not in range anymore
m = -.3 * (state['missed_interactions'] / tot_interactions)
# small negative reward for inactive epochs
i = -.2 * (state['inactive_for_epochs'] / tot_epochs)

reward = h + a + c + b + i + m
```

To explain this even better, here is a [Comic](https://hackernoon.com/intuitive-rl-intro-to-advantage-actor-critic-a2c-4ff545978752) that explains it even though I still can't understand.

As for collecting WiFi, the pwnagotchi uses bettercap that can actively or passively. Normally you need a expensive network adapter/card and a laptop, this is just a few bucks and a bonding moment that awakens your parental side and if any harm comes to them you can expect things are goind down.

As for collecting the handshakes we exploit the 4 way handshakes, the system that lets us bring a WiFi connections with a router.

![4 Way Handshake diagram](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/43b330bb-625d-40cf-85df-00dd972c8ad8)

And now I will attempt to tell you how it works even though I can barely read.

1) Router sends a ANounce, or a key to the client
2) The client encrypts it, sends it as a MIC, which will send a signal saying it's game time, as well as send a SNounce, their own key
3) Router starts developing a PTK, which needs both MAC addresses, Both keys, and a PMK, which has the **PASSWORD!!!!** The router also sends a MIC, and a GTK, which encrypts the traffic, still use a VPN though
4) The Client will send a signal saying the GTK is installed
5)  ~~Go to pirated anime sites once connected on the internet~~

>[!WARNING]
>Don't use this as a cheat sheet, it could be wrong, I'm not good at reading. Yes, I did type all this but I have a child that spews WiFi passwords, and they are enjoying life!

Once then you can take the PCAP out and use bruteforce attacks or use plugins to recover some passwords.

### Moods
Moods or the Faces are the way the pwnagotchi communicates to the user on what exactly is happening. These can be customized but here's some quick run down of just some of the faces.

-  (⇀‿‿↼) sleeping
  - Happens when hopping among WiFi Channels
- ( ⚆_⚆), (☉_☉ ) observing (neutral mood)
  - Observing what bettercap finds
- (•‿‿•) happy
  - Could be 3 things
    - AI mode is ready
    - Valid key material captured
    - In MANU mode
    - A friend is met and they have a strong bond
- (⌐■_■) cool
  - Deauth a client
- (☼‿‿☼) motivated
  - Two things
    - Pwnagotchi just scored the best reward level
    - Met a high bond friend
- (≖__≖) demotivated
  - Pwnagotchi just scored the worst reward level
- (ب__ب) lonely
  - Lost contact with
    - Friend
    - Interation with access points or client stations
-  (☓‿‿☓) broken
  - Rebooting via update or coping via bugs
- (#__#) debugging
  - Use for debugging

Now that we cover bare basic things, lets grab the bare essentials.

## Step 1) Grab Some Supplies

### Hardware
#### Raspberry Pi Zero W

![Raspberry Pi Zero W 2](https://cdn.cnx-software.com/wp-content/uploads/2021/10/Raspberry-Pi-Zero-2-W-board.jpg?lossy=0&strip=none&ssl=1)

The Respberry Pi Zero is really the bare minimum you need, realistically it's all you need along with a micro SD card (Minimum 8GB), and a micro usb cable that can deliever power and data. Interestingly you can actually run the Pwnagotchi on more modern Raspberry Pis such as

- Raspberry Pi Zero 2 W
- Raspberry Pi 3
- Raspberry Pi 4
- Raspberry Pi 5

You may also check to see if you want headers for things like a display to be powered. Simply look for the pins.
![Raspberry Pi Zero WH](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/c0a69b16-1dd8-469f-95cd-892784691eea)


>[!IMPORTANT]
> It's more ideal for using the Zeros lineup as there are more reasources based of the Zeros. With that said better or more cooler results if you do go with the newer pis.
> AND PLEASE AVOID SCALPERS

>[!NOTE]
> The Pi is all you need with everything else is just an added bonus. If you with to see the Pwnagotchi face, see the plugins sections.

#### Battery

If you want your Pwnagotchi to not be on essentially life support only for it to die when someone pulls the plug you can use batteries to keep it alive for hours. On the site there is a list of batteries however you should still look up each batteries. Here are some examples.

**[PiSugar2](https://www.amazon.com/Pisugar2-Portable-Pwnagotchi-Raspberry-Accessories/dp/B08D678XPR/ref=sr_1_2?crid=1UQ8PCL99F5UL&keywords=pisugar&qid=1697175381&s=electronics&sprefix=pisugar%2Celectronics%2C80&sr=1-2)**
![PiSugar2](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/e6936bd1-b30c-4f9f-99c0-430b5dee16af)

**[Ups-Lite battery for the Raspberry Pi Zero W](https://www.amazon.com/ElectronicMaker-Battery-Electricity-Detection-Raspberry/dp/B093FX8SDW)**
![Ups-Lite battery](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/d8c515e2-1f1c-4db0-afbe-325693ea7dd4)

#### Display

If you wish to use a display there are several version but by far the most popular version is the [Waveshare V3 paper hat](https://www.amazon.com/2-13inch-Display-HAT-Two-Color-Raspberry/dp/B07Z1WYRQH/ref=pd_bxgy_img_sccl_1/132-9253675-8321923?pd_rd_w=TOG0F&content-id=amzn1.sym.43d28dfc-aa4f-4ef6-b591-5ab7095e137f&pf_rd_p=43d28dfc-aa4f-4ef6-b591-5ab7095e137f&pf_rd_r=8JYZ9NABNVK2WMCNJHRX&pd_rd_wg=Q8MyG&pd_rd_r=f371265c-5d59-4c91-88e1-9dedf7cc84e9&pd_rd_i=B07Z1WYRQH&psc=1), a E-Ink display.
Sadly this screen is the reason why most pwnagotchi breaks as the old Pwnagotchi isn't updated to use this screen. Luckly the community came through and made some changes and everything works.

**Pros of a E-Ink Display**
- If the device loses power, you keep the last image display
- Low power usage
- Easy on the eyes
- Can be read in the sunlight

**Cons of a E-Ink Display**
- Can't read in the dark
- Low refresh rates
- Limited Colors
  - There are more colorful ones like the Waveshare V3 with the red ink, but it can get complicated with editing the files to make the red show

![Waveshare V3](https://m.media-amazon.com/images/I/71n0DedKVfL._AC_SL1500_.jpg)

#### Hardware clock

It's to update to time without internet. That's it. Not really needed. If anything makes things harder to work with.

![Hardware clock](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/a33b8076-d7de-41bf-bf29-dddf10891ad7)


#### Cases

There are tons of cases for the Pwnagotchi, or just the Raspberry Pi itself if you are going for the most minimalist. These can be 3D print or bought if you want to be scammed. All cases can be found online but depending on any hardware or modifications you need to choose a specific case. Here are some cases with parts and their modifications.

- Here are **14** cases, just carefully see what are they label as for the right [prints](https://www.thingiverse.com/thing:3920904).
![Pwnagotchi cases](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/4a1c7760-7171-4a98-a6a7-602b4b8cf66f)


- Here is a **REALLY COOL** case for a Pwnagotchi with a Raspberry Pi Zero W UPS Lite, a Pimoroni or Waveshare 2.13 Inch
![Pwnagotchi Case - UPS Lite/Pimoroni or Waveshare 2.13 Inch](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/63fe4c9f-f577-4b7c-b8e4-aec943a7b2e0)


- Here is a case for literally just the [Raspberry Pi Zero W](https://www.printables.com/model/32485-slim-raspberry-pi-zero-w-case)
![Minimal case](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/7bea3e50-9c3f-48e8-ab44-c0fd021db067)


Once you got everything you need, time for...

### Step 2) Flash the image
