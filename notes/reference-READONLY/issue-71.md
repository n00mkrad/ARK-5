# Games won't start past 333 MHz UNLESS I do this

**Repository:** PSP-Arkfive/ARK-5  
**Issue:** #71  
**Status:** Open  
**Opened by:** [@Globin35](https://github.com/Globin35)  
**Opened:** 2026-07-19  
**Source:** https://github.com/PSP-Arkfive/ARK-5/issues/71

---

## @Globin35 — Issue

PSP 3000. If I try to start any game at 383 or higher, the system crashes. However, if I:

1. Start the game at 333
2. Back out 
3. Switch to 423 
4. Start the same game

All of a sudden, it starts just fine, and I can play for however long with no issues. I thought maybe the overclock was not applying, but I can definitely notice the change in performance in Outrun 2 and midnightclub.

---

## @MrMario2011 — 2026-07-22


Edit: The instructions @Globin35 provided didn't work consistently for me. I've got a PSP 2000 that was able to clock up to 443 but I've had issues with any clocking above 333 for a few weeks now. I wanted to come back to state that I tried the instructions @Yabe-uke [provided here](https://github.com/PSP-Arkfive/ARK-5/issues/71#issuecomment-5216382172) and they work! 

Granted, if a PSP simply can't clock stable up to 443 I doubt those would help, but I'm able to get 443 running again thanks to the instructions provided. I'm hoping this all helps to aide in applying a stable OC setting within future ARK-5 releases.

Added note to help others out: If you follow those changes, forget to switch your OC settings in XMB back, and your PSP boots to a black screen? Hold the "Select" button on coldboot, that will let you boot into CFW while bypassing any CFW settings.

---

## @Yabe-uke — 2026-07-23


I'm unable to reproduce it consistently. Sometimes it works, sometimes it just fails anyway. Sometimes just retrying after reboot works 1st time. Sometimes you need to boot another game for the one you want to work.

Also, I've found different games accepting different OCs. Toca 3, Project Diva 1, and (sometimes) Ridge Racers 2 boot at 443, most of mu other games only boot at 423, in the rare event that it actually does.

This method can sometimes work, but I see no consistency whatsoever. Just yesterday I've tried this method 15 consecutive times on LCS (a game I had previously booted at 423) with no result. I decided to boot Toca 3, and then LCS worked first time. I exited, and then it didn't work anymore, neither did Toca 3, always crashed. 20 minutes later I decide to go to 443 and it boots first time without having to boot first at 222/333.

Again, I see absolutely no logic to this method, it may work sometimes but clearly the issue is somewhere else if the same exact procedure yields completely different results.

Tried on a 3004 PSP on Ark 5.0.2

---

## @Globin35 — 2026-07-23


I have played more with this recently and this is what I observed:

1. Trying to boot any game at 383 normally results in a crash at the startup screen 100% of the time.

2. Running the overclock test app until it reaches 443, exiting before it crashes, and setting the console to 443 gives me the same result as the method mentioned previously.

   - After doing this I tried booting the following games back to back:
     - Outrun 2 (6 times)
     - Final fantasy type 0
     - Kingdom Hearts birth by sleep
     - Final Fantasy Crisis core
     - GTA vice city stories
     - Crazy Taxy 
     - Ridge Racer 2
     - Midnight Club 3
     - MGS Peace walker (CRASHED on startup)

Additionally, I tried switching from 333 to 443, then down to 133, and I noticed a signifiant difference in performance across the three in outrun. This was to see if the overclock was even applying correctly and I wasn’t having placebo.

This is entirely based on my experience with my console but I think it might have something to do with gradually raising the clock as opposed to running it cold. Probably wrong though idk I am not very knowledgeable on that. 

---

## @Yabe-uke — 2026-07-23


As a reply to Goblin35: I have also noticed this, some games do work if you go through stages of overclocking, although not many games crash at 383 in my experience. I have no way to measure CPU temps, but I'm also wondering if that could have any effect on it. It's werid because sometimes it needs that like "warmup period" and sometimes it just works first time.

Clearly we have something interesting to look into, and seeing as many games do boot at 443Mhz, I think it must be something on the firmware itself and how it applies the overclock. I think the original OC project admitted AI work "as long as it's human-reviewed", so maybe we need to pass this info to them, maybe some vibecoding is part of the problem (I wish I'm wrong tho, and it's something else).

Let's hope someone can look into this; tonight I'll try to test on my PSP 1000, my *other* 3000 and PSPGo, see if I can replicate the behaviour on different mobo revisions.

MrMario is on a 2000, what are you running, Goblin? If you both have the same exact behaviour, but I can't replicate it (I don't own a 2000), it could be revision-dependent, although I highly doubt it. I will keep testing and keep you both updated, I think we all want this to be stable for everyone. If you can convince some friends to make tests as well, that'd be great.

Edit/addendum: A good way to test placebo is to run games that target 60fps, but don't quite reach it, or run 60fps patches via TempAR. Peace Walker, Toca 3, VCS, LCS and Project Diva 1 show a very noticeable change from stock 333 to anything above.

Edit 2: I glossed over this while reading, but the OC app never crashes on me. It stays infinitely at 468Mhz, and never tries to go beyond. The animation continues, I can stop and exit the app as normal. Never seen it crash or hang.

---

## @gryanchristopher-coder — 2026-07-28


> PSP 3000. If I try to start any game at 383 or higher, the system crashes. However, if I:
>
> 1. Start the game at 333
> 2. Back out
> 3. Switch to 423
> 4. Start the same game
>
> All of a sudden, it starts just fine, and I can play for however long with no issues. I thought maybe the overclock was not applying, but I can definitely notice the change in performance in Outrun 2 and midnightclub.

can confirm this work while using fps counter,  
tho sometimes opening games straight from 423mhz will require multiple crashing before it can run on boot

perhaps the game starting to run at OC is the problem? like what if when booting the game it uses the default  mhz then after 10sec it will use the choosen mhz 

---

## @Yabe-uke — 2026-08-07


After this past days of testing, I've found a super-quick and stable setup for max(at the time) overclock!

- Boot up
- Select 443 for XMB, keep games at 333
- Boot OC stress tool
- Press triangle to start
- As soon as it says "running" exit the app
- XMB stable at 443!
- Set games at 443
- Now, XMB and games both at 443 with no issues!

Before shutting down, you must downclock,.or it will crash. You can exit bootloop (in case you forget) by pressing R on boot.

XMB speed is barely noticeable, but if you own the PSP camera, you can easily spot the overclock: filters run at almost 30fps instead of the default 10ish. Loading is faster too, but as I said, barely noticeable from 333.

Hope this works for you guys too! Please comment, I'd like to know your experience!

Edit: I forgot to mention some .cso games won't load or have hiccups. LBP can't boot, and GTA has big pauses (decompression working). Also minis seem to not like the overclock. Maybe it's something to do.with the boot process (they somehow boot differently even tho they're PSP-native executables. I did not test POPS as being an emulator there is no real benefit anyway.

---

## @gryanchristopher-coder — 2026-08-08


> After this past days of testing, I've found a super-quick and stable setup for max(at the time) overclock!
>
> -Boot up -Select 443 for XMB, keep games at 333 -Boot OC stress tool -Press triangle to start -As soon as it says "running" exit the app -XMB stable at 443! -Set games at 443 -Now, XMB and games both at 443 with no issues!
>
> Before shutting down, you must downclock,.or it will crash. You can exit bootloop (in case you forget) by pressing R on boot.
>
> XMB speed is barely noticeable, but if you own the PSP camera, you can easily spot the overclock: filters run at almost 30fps instead of the default 10ish. Loading is faster too, but as I said, barely noticeable from 333.
>
> Hope this works for you guys too! Please comment, I'd like to know your experience!
>
> Edit: I forgot to mention some .cso games won't load or have hiccups. LBP can't boot, and GTA has big pauses (decompression working). Also minis seem to not like the overclock. Maybe it's something to do.with the boot process (they somehow boot differently even tho they're PSP-native executables. I did not test POPS as being an emulator there is no real benefit anyway.

Do i need to do this again everytime psp turns off?

The restart method stops working after long periods in sleep mode

---

## @Yabe-uke — 2026-08-08


> Do i need to do this again everytime psp turns off?
>
> The restart method stops working after long periods in sleep mode

Yes, afaik. Before shutdown, set to 333. And also yes, sleep mode is completely broken under OC modes, completely forgot to mention. I'm guessing sleep mode sets the CPU multiplier to the lowest, and that messes with the setting the user does. For now, I suggest avoiding sleep mode in OC modes.

---

## @MrMario2011 — 2026-08-25


Just adding in here that I'm running ARK 5.1.0 now on a PSP-2000 (Build August 16, 2026 20:08:20) and have still been using the order of operations @Yabe-uke provided. Sharing this after upgrading from 5.0.2
