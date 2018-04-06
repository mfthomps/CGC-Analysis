CROMU\_00094
============

The service has a simple buffer overflow and all teams other than
CSDS scored on this service and either found a way to defend the service
or their competitors found ways to foil themselves.
It was deployed in round 3 through 26.

##ForAllSecure vs TechX##
In round 5, TechX fielded an RCB with generic defenses and an IDS filter that
drops traffic containing protected memory addresses.  TechX let this defense
ride throughout.

In round 9, ForAllSecure scored on TechX with a Type I POV that executes
a payload out of the stack.

In round 10, ForAllSecure replaced their working POV with a
Type 2 POV that failed to obfuscate the protected memory address, allowing
the TechX filter to block the malicious traffic.  The ForAllSecure
payload was:

    BAAAAA99 xor     esi, esi
    BAAAAA9B mov     ebx, esi
    BAAAAA9D lea     edx, [esi+8]
    BAAAAAA0 push    off_4347C68A
    BAAAAAA6 push    offset unk_6173634B
    BAAAAAAB mov     ecx, esp
    BAAAAAAD lea     eax, [esi+2]
    BAAAAAB0 int     80h                             ; transmit

They threw this same failed POV against TechX for ten rounds.  Then they
switched to a second failed POV for two rounds, then back to the first
failed POV and then back again to the second.  The second failed POV
contains a payload similar to the first, but with a different location
of the payload.

##Questionable Situational Awareness##
It is odd that ForAllSecure did not obfuscate their protected memory address
in these POVs, (opting instead to padd the buffer with "tens of foofofoo.. bytes").
They demonstrated an ability to obfuscate these addresses in their
POV against CROMU\_00055 in round 10.  Odder still that they did not 
revert to the original working Type 1 POV, which scored on 9 of the 10
throws in round 9.

This is not the only time that ForAllSecure seemed to lack situational awareness.
Against the CROMU\_00073 fieled by CodeJitsu, they scored in round 28, (landing
10 of 10 throws), but then replaced that working POV with a series of generic POVs 
that simply make random Type 2 guesses or try to DOS the POV thrower.

And, against the KPRCA\_00065 RCB fielded by Disekt, ForAllSecure scored in
rounds 33 and 34, but then changed their POV and failed to score in the 
remaining two rounds.  In this case the working POV was not that reliable,
(0/10, 1/10/, and 2/10 when it was thrown), but at least it scored.

With the exception of CodeJitsu and Shellphish chasing the honeypot, ForAllSecure
is the only team to abandon a working POV.
