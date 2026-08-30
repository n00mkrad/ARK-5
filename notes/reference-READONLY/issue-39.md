# Possible downclock bug in sctrlHENSetSpeed

Cursor is telling me that there's a bug in ARK5 when I'm trying to use sctrlHENSetSpeed to reset the clock back to 333Mhz from 407Mhz.  Is the below logical?  It is saying that cancelOverclock() will do nothing and therefore origSetClockFrequency won't then work.

> Root cause is an ARK-5 bug, not just our apply logic.
> 
> When you call sctrlHENSetSpeed(333, …) after running at 407 MHz, ARK’s handler does this:
> 
> 
> current_frequency = cpu;   // set to 333 first
> if (cpu > 333) doOverclock();
> else {
>     cancelOverclock();   // uses current_frequency — already 333, not 407!
>     origSetClockFrequency(cpu, bus);
> }
> cancelOverclock() tries to ramp the PLL down using current_frequency, but that was already overwritten with the target (333), not the previous speed (407). The cancel loop does almost nothing, the PLL stays overclocked, and FPS stays high. The overlay showed 333 because we display option_clock_mhz for ≤333 MHz — menu value, not hardware.
