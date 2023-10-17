
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

The problem with the handshake is that the first 2 handshakes are what spawn the PMK, which is great because realistcally we only need half the handshakes, perfect for our weak signal.

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

## Step 2) Flash the image

There are a few images out there, with the one I use is admittibly old. [Version 1.5.5](https://github.com/wpa-2/pwnagotchi/releases) created by github user @wpa-2, but github user @jayofelony has been contantly been updating the project, thus earning Chad Status of being a parent to every script kiddies's favorite child, with the newest img file they have uploaded at the time of writing is [2.4.6](https://github.com/jayofelony/pwnagotchi/releases/tag/v2.4.6), with Raspberry Pi 4 compatibility. I have provided both files mention in the repo. 

Next we need the software to go to Google and search for 'Balena Etcher' **AND PLEASE NOTE THE LINK, BECAUSE THERES OFTEN SOMEONE PUTTING A SCAM LINK, AND GOOGLE IS HAPPY TO RECOMMMEND IT OVER THE ACTUAL LINK**

>[!WARNING]
> I HAVE FALLEN FOR THIS PLEASE BECAREFUL, USUALLY THEY ARE MARKED AS ADS, READ UNLIKE ME

Once installed the software, this should be the software:
![Balena Etcher](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/977d919e-6ef9-42d2-91d2-0590753d72ba)

Simply select the .img file you have choosen, then **PICK THE RIGHT STORAGE, AKA THE LITTLE SD CARD NOT YOUR MAIN TB EXPENSIVE HAVE IMPORTANT THINGS STORAGE!!!!!**

>[!WARNING]
> PLEASE CHOOSE THE RIGHT ONE, CHOOSING THE WRONG ONE WILL WIPE EVERYTHING OF SAID STORAGE AND REPLACE IT WITH THE .img FILE I PROMISE YOU DO NOT WANNA LOSE YOUR MINECRAFT SERVER

Then once everything is selected **Correctly**, hit flash, and wait. You should have a little micro SD card ready, but you can't put it in just yet.

## Step 3) Add somer personality

You can use your default OS or use a software like Filezilla. What you need to do regardless is create the file *config.toml*. How to do that?

1. cd or move into the following directories */etc/pwnagotchi/* and create the *config.toml* file. This is the main source of all customizations, or at least activating them. Once created copy and paste the following in your file:
```
main.name = "pwnagotchi"
main.lang = "en"
main.whitelist = [
  "EXAMPLE_NETWORK",
  "ANOTHER_EXAMPLE_NETWORK",
  "fo:od:ba:be:fo:od",
  "fo:od:ba"
]

main.plugins.grid.enabled = true
main.plugins.grid.report = true
main.plugins.grid.exclude = [
  "YourHomeNetworkHere"
]

ui.display.enabled = true
ui.display.type = "waveshare_2"
ui.display.color = "black"

personality.advertise = true
personality.deauth = true
```
This will be the skeleton of everything, you can customize the name, the language \(Japanese requires *ui.font.name = "fonts-japanese-gothic"* to be added, simply add it to the bottom of the list\), to the type of display color if possible. Note the *main.whitelist* and the *main.plugins.grid.exclude* lists. You want to put your SSID, or the WiFi name you don't want pwned by your pwnagotchi. You can also use your ESSID but I have no idea how to find that.

>[!NOTE]
> See the pwnagotchi website to set your *ui.display.type* to the display you end up using
>[!WARNING]
>Deauthing devices is a legal grey area, so it's best to change *personality.deauth = true* to false. Don't worry, the software it use, bettercap, can just listen for handshakes, and it's just as good as deauthing.

2. Once you wrote your *config.toml* file simply take it out, and put it in your raspberry. On the Raspberry Pi zero W there are two micro usb ports, the one closer to the end is your power port, but the port you need is the the one next to it, the data port

![Ports](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/6619ca7f-e351-4380-abea-29cba24bd63f)


Plug it in, and **DO NOT REMOVE IT**. This is generating RSA keys, and removal of the cabel will corrupt everything. After 7-10 minutes, you need to assign it with the following
   - IP) 10.0.0.1
   - Netmask) 255.255.255.0
   - Gateway) 10.0.0.1
   - DNS) 8.8.8.8

![Windows config](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/309d4b34-2a21-445b-8b33-7481d0ae141c)


>[!NOTE]
> If nothing happens for a few minutes, check to see if your cable can even transfer data. If you can't then find one. 

If you are on step two and can not configure or see the ethernet option on your windows machine, you may have not have the right drivers, [This website](https://cyberspacemanmike.com/product/rdnis-drivers-ethernet-over-usb-for-make-pwnagotchi-work/) sells it for free, and I know, sketchy but it works for me, and if you don't need it, don't install it. Simple. File can also be found in the repo.


3. Use Filezillia or PuTTY to ssh into the pi.
   - Host Name) pi@10.0.0.2
   - Port) 22
   - Password) raspberry

Congrats on getting in your Pwnagotchi, now go change those passwords, you know basic linux lmao.

>[!NOTE]
> A good command to remember is *sudo systemctl restart pwnagotchi*. This will restart your pi, thus updating things, including changes you made

## Step 4) Seeing your childs face

There are a few ways to see the UI, so let's start with a few

### Hardware

Take out the usb and if you have a battery, carefully install the battery as to not damage both the battery or the pi. Usually there are instructions, but if not, Youtube.....

After that, install the display. Usually it's a 'hat', or something you can place on the pins, but be sure to not bend the pins other wise you have to see a therapist instead when you see this...

![Bent pins](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/7b10ac3a-c5cf-4d7b-b2de-034d0fbc2521)

Once you installed everything you can either turn on the battery or plug the usb in either ports. Power port will make the image more clear but for the tutorial, it's best to keep using the data port.

### Bluetooth

If you wish, you can connect to your phone via bluetooth, this is one way to use a Pwnagotchi without a screen but it will require you to add things in your *config.toml*.

```
main.plugins.bt-tether.enabled = false

main.plugins.bt-tether.devices.android-phone.enabled = false          # the name of this entry is android-phone
main.plugins.bt-tether.devices.android-phone.search_order = 1         # in which order the devices should
                                                                      ## be searched. E.g. this is #1
main.plugins.bt-tether.devices.android-phone.mac = ""                 # you need to put your phones
                                                                      ## bt-mac here (settings > status)
main.plugins.bt-tether.devices.android-phone.ip = "192.168.44.44"     # this is the static ip of your pwnagotchi
                                                                      ## adjust this to your phones pan-network
                                                                      ## (run "ifconfig bt-pan" on your phone)
                                                                      ## if you feel lucky,
                                                                      ## try: 192.168.44.44 (Android) or
                                                                      ## 172.20.10.6 (iOS)
                                                                      ## 44 is just an example, you can choose
                                                                      ## between 2-254 (if netmask is 24)
main.plugins.bt-tether.devices.android-phone.netmask = 24             # netmask of the PAN
main.plugins.bt-tether.devices.android-phone.interval = 1             # in minutes, how often should
                                                                      ## the device be searched
main.plugins.bt-tether.devices.android-phone.scantime = 10            # in seconds, how long should be searched
                                                                      ## on each interval
main.plugins.bt-tether.devices.android-phone.max_tries = 10           # how many times it should try to find the
                                                                      ## phone (0 = endless)
main.plugins.bt-tether.devices.android-phone.share_internet = false   # set to true if you want to have
                                                                      ## internet via bluetooth
main.plugins.bt-tether.devices.android-phone.priority = 1             # the device with the highest
                                                                      ## priority wins (1 = highest)

main.plugins.bt-tether.devices.ios-phone.enabled = false          # the name of this entry is android-phone
main.plugins.bt-tether.devices.ios-phone.search_order = 1         # in which order the devices should
                                                                      ## be searched. E.g. this is #1
main.plugins.bt-tether.devices.ios-phone.mac = ""                 # you need to put your phones
                                                                      ## bt-mac here (settings > status)
main.plugins.bt-tether.devices.ios-phone.ip = "172.20.10.6"     # this is the static ip of your pwnagotchi
                                                                      ## adjust this to your phones pan-network
                                                                      ## (run "ifconfig bt-pan" on your phone)
                                                                      ## if you feel lucky,
                                                                      ## try: 192.168.44.44 (Android) or
                                                                      ## 172.20.10.6 (iOS)
                                                                      ## 44 is just an example, you can choose
                                                                      ## between 2-254 (if netmask is 24)
main.plugins.bt-tether.devices.ios-phone.netmask = 24             # netmask of the PAN
main.plugins.bt-tether.devices.android-phone.interval = 1             # in minutes, how often should
                                                                      ## the device be searched
main.plugins.bt-tether.devices.ios-phone.scantime = 10            # in seconds, how long should be searched
                                                                      ## on each interval
main.plugins.bt-tether.devices.ios-phone.max_tries = 10           # how many times it should try to find the
                                                                      ## phone (0 = endless)
main.plugins.bt-tether.devices.ios-phone.share_internet = false   # set to true if you want to have
                                                                      ## internet via bluetooth
main.plugins.bt-tether.devices.ios-phone.priority = 1             # the device with the highest
                                                                      ## priority wins (1 = highest)
```

Simply copy and edit the code above in the *config.toml* and once a connection between your phone and pi is made, head to the ip in your browser to see it's face. This is also a good way to hide your pwnagotchi while it collects handshakes.

### Browser/Computer

To see your pwnagotchi on a browser in general you can go in your browser and enter 10.0.0.2:8080 to see it. The username and password for the website is *changeme*. If you wish to change them either paste the following in the *config.toml*:

```
ui.web.username = "my_new_username"
ui.web.password = "my_new_password"
```

Or head into the plugins tabs and enable *webcfg.py* and restart your Pwnagotchi. Now you can go back to the website, head into the plugin tab, click on the *webcfg.py* newly made hyperlink, and now you can customize your built-in plugins.

But what's a plugin?

## Step 5) Plugins

The pwnagotchi at it's bare minimum is nice but there are some built in plugins. If you recalled earlier eariler there was a plugin to edit the *config.toml* file, named *webcfg.py*. There was a plugin that allows bluetooth connection, otherwise known as *bt-tether.py*. The *memtemp.py* shows the usages of the memory and temputures.

However there are more elaborate plugins such as *grid.py*, which is how:

> signals the unit’s cryptographic identity and (optionally) a list of pwned networks to PwnGRID at api.pwnagotchi.ai.

And you can see it in the offical [pwnagotchi](https://pwnagotchi.ai/map/) website, where they have a map updating on new friends.

To add a little bit of *custom* personality, we can actually import plugins. If you go into your *config.toml* file and add:

```
main.custom_plugins = "/usr/local/share/pwnagotchi/custom-plugins/"
```
and
```
cd /usr/local/share/pwnagotchi/ && mkdir custom-plugins
```

>[!NOTE]
> Some plugins requires certain installments of other things to run, and may even require an entire plugin, or API keys from 3rd party tools online, so read the documentations to be sure

we can git clone community made plugins into the *custom-plugins* directory, and simple call it from the *config.toml* and restart the pwnagotchi to update it. Here is a set a plugins like:
- [A plugin that will add a battery capacity and charging indicator for the Waveshare UPS HAT (C)](https://github.com/hannadiamond/pwnagotchi-plugins)
- [A plugin that adds age and strength stats based on the number of epochs and the number of epochs trained](https://github.com/hannadiamond/pwnagotchi-plugins)
- [XP bar for the pwnagotchi](https://github.com/GaelicThunder/Experience-Plugin-Pwnagotchi)

![custom plugins](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/b6b872a5-8a95-4fd2-992f-728866c1cad7)

And if you think this is not that impressive, there are literal face lifts for pwnagotchis, where you get more impressive looks, such as Pikachu on your pwnagotchi. Now you can pwn WiFi while being sereved with cease and desist from Nintendo.

[**WHO DOESN'T WANT TO HAVE A PIKACHU**](https://github.com/roodriiigooo/PWNAGOTCHI-CUSTOM-FACES-MOD)

![Pikachu](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/c1847c5f-1aff-49f6-8d59-3930aef0f72b)


## Things to know

So you got a plugins, a fancy case, and a new friend, what now. Well first there are some files to remember.

- */etc/pwnagotchi/config.toml* where you put your custom configurations
- */root/handshakes/* is where all your handshakes go

It's also good to remember how to read at least the basic UI:

![Basic UI](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/d8121806-80b4-4a46-8f9d-bd868742c092)


- CH: This displays the current channel the unit is operating on.
  - When the unit is performing recon and hopping on all channels, it will display * instead of a number. It is gathering the number of APs on each channel when it is conducting recon. Recon signals the start of a new epoch.
- APS: Number of access points on the current channel.
  - The total visible access points across all channels (according to the last recon) is displayed in parentheses.
- UP: The uptime of the unit, since its last reboot. It is displayed in hh:mm:ss format.
- PWND: Number of handshakes captured during this current session.
  - The number of unique networks your Pwnagotchi has eaten at least one handshake of, from the beginning of its life, is displayed in parentheses.
  - The SSID of the latest network handshake your Pwnagotchi has acquired is displayed in brackets.
- MODE: Mode indicates how Pwnagotchi is currently functioning. See above for more info about modes.
  - MANU: This appears when the unit is running in MANUAL mode, which is triggered when you start up your unit with the USB network cable connected.
    - This mode is good for updating and backing up your unit and using bettercap’s web UI.
    - Pwnagotchi does NOT sniff or capture handshakes when it is in MANUAL mode.
    - Stuck in MANUAL mode? Turn on the unit without the USB network cable connected.
  - AUTO: This indicates that the Pwnagotchi algorithm is running in AUTOMATIC mode, with AI disabled (or still loading).
    - Pwnagotchi will still sniff and capture handshakes in this mode; it is mostly functional—the primary difference between AUTO and AI mode is its actions are being determined by a static algorithm instead of the AI deciding what the Pwnagotchi should do for optimal pwnage.
    - This disappears once the AI dependencies have been bootstrapped and the neural network has finished loading. (On a RPi0W, this process takes about 20–30 minutes.)
    - If you are running your Pwnagotchi without the AI enabled, this is the mode you’ll stay in.
  - AI: AI mode appears once the AI dependencies have finished loading and the neural network is functional.
    - Once this appears, your Pwnagotchi is all ready to begin learning from its pwnage!
- FRIEND DETECTED!: If another unit is nearby, its presence will be indicated between the bottom stats bar and your Pwnagotchi’s status face.
>[!NOTE]
> If more than one unit is nearby, only one—whichever has the stronger signal strength—will be displayed here.

Another cool things about the Pwnagotchi is that it also servers a Pager. The *PwnMAIL* is as the website says:
> crypto-pager!

AND

> Each message is encrypted on your Raspberry with the recipient RSA public key before being sent, therefore we only have access to encrypted data and we have absolutely no way to see the cleartext as it can only be done by the original recipient via his private key

Just run *sudo pwngrid -whoami* to see your key.

You can also check your webUI, there is a tab there just for checking your inbox
```
sudo pwngrid -inbox
# and for all other pages if more than one
sudo pwngrid -inbox -page 2
```

To fetch a message given its id (123 in this example), verify the sender signature and decryp it:
```
sudo pwngrid -inbox -id 123
# in case you want to save the decrypted message body to a file
sudo pwngrid -inbox -id 123 -output picture.jpg
```

This will automatically mark the message as read, to mark it back as unread:
```
sudo pwngrid -inbox -id 123 -unread
```

To delete it:
```
sudo pwngrid -inbox -id 123 -delete
```

And to send an encrypted message (max size is 512KB) to another unit having its fingerprint (ca1225b86dc35fef90922d83421d2fc9c824e95b864cfa62da7bea64ffb05aea in this example):
```
sudo pwngrid -send ca1225b86dc35fef90922d83421d2fc9c824e95b864cfa62da7bea64ffb05aea -message "hi there, how are you doing?"
# in case you want to send a file instead
sudo pwngrid -send ca1225b86dc35fef90922d83421d2fc9c824e95b864cfa62da7bea64ffb05aea -message @/path/to/file.jpg
```
>[!NOTE]
> People use to send messages via just numbers, you can look them up, and if you use Goroawase, you are based

![Sailor Moon being base](https://github.com/Cold-Syntax/Dumb-Down-Pwnagotchi-Guide/assets/85046345/7853118d-7072-45fe-b139-7cca8999345c)


For SD card protection, the worst thing possible, in the *config.toml* put/enable
```
fs.memory.enabled = true
fs.memory.mounts.log.enabled = true
fs.memory.mounts.data.enabled = true
```

or if you are smarter than me
```
fs.memory.mounts.log.enabled = true     # switch
fs.memory.mounts.log.mount = "/var/log" # which directory to map into memory
fs.memory.mounts.log.size = "50M"       # max size to put into memory
fs.memory.mounts.log.sync = 60          # interval in seconds to sync back onto disk
fs.memory.mounts.log.zram = true        # use zram for compression (recommended)
fs.memory.mounts.log.rsync = true       # use rsync to copy only the difference (recommended)
```

And lastly to Backup your Pwnagotchi
```
usage: ./scripts/backup.sh HOSTNAME backup.zip
```
