NRFIN_00063: Battle of the generics
===================================
Once upon a time, a CGC challenge binary (CB)
author inserted a carefully constructed heap overflow
vulnerability into a service, and built a reference POV 
that corrupts selected data structures within the heap.
But then the author discovered the service had another,
more easily exploited buffer overflow that overwrites
a function pointer.  Fortunately for the telling
of this saga, the CB author responded by declaring 
that the service now had two vulnerabilities,
and developed a reference POV for the second.  This second vulnerability
led to interesting interaction between CRS's during CFE.

Many CFE vulnerabilities are buffer overflows resulting
in a corrupt function return address.  This type of vulnerability
can be mitigated using stack cookies, and that defensive technique was
commonly used during CFE.  However, corruption of a function pointer 
is not mitigated by stack cookies, allowing this service to illustrate 
the effectiveness of other kinds of generic defenses.

CodeJitsu Develops a POV
========================
The NRFIN\_00063 service was introduced in round 49, and in
round 50 CodeJitsu had a working Type 2 POV which it successfully
deployed against all competitors.  This POV sends the service
a payload that, when executed, causes content of protected memory to be exfiltrated
using the DECREE transmit call. This payload is located in the
process stack, and the overwritten function pointer 
transfers execution to the payload. As shown below, the protected
memory address, (0x4347C000) is obfuscated within the payload.

    BAAAAEE1 xor     eax, eax
    BAAAAEE3 xor     ebx, ebx
    BAAAAEE5 xor     edx, edx
    BAAAAEE7 mov     al, 2
    BAAAAEE9 mov     bl, 1
    BAAAAEEB mov     cx, 32D4h
    BAAAAEEF xor     cx, 7193h
    BAAAAEF4 shl     ecx, 8
    BAAAAEF7 mov     cl, 0C0h ; ''
    BAAAAEF9 shl     ecx, 8 ;  ecx is now 0x4347C000, start of protected memory
    BAAAAEFC mov     dh, 10h ; edx is now 0x1000, i.e., a full page
    BAAAAEFE xor     esi, esi
    BAAAAF00 int     80h  

The "int 80h" instruction invokes the DECREE transmit function, which takes its
buffer address in the ecx register, which by this time contains the deobfuscated
address of protected memory.  Obfuscation of protected memory addresses within 
POV payloads prevents simple IDS filters from blocking the payloads by scanning
for such addresses.  Two teams in CFE frequently deployed such IDS filters, however
neither were deployed in round 50 when CodeJitsu first scored with this POV.

Another interesting aspect of this POV payload is its behavior subsequent to exfiltrating
the protected memory content.  One advantage of Type 2 POVs is that, unlike Type 1
POVs, they do not need to crash.  If the payload terminates or otherwise avoids
crashing, then competitors would receive less indication that their services
have been scored on.  Following each round, the CGC infrastructure provides a CRS 
with feedback indicating if services have crashed.
Additionally, the CRS is provided with a network tap of all traffic destined for
its services.  This traffic can be fed to instrumented instances of services to detect 
crashes.  

Thus, there can be advantages to a Type 2 POV that does not crash.  However, this CodeJitsu
POV does in fact crash shortly after exfiltrating the protected memory, providing
their competitors with indications that a service vulnerability may be currently exploited.
By round 53, three teams had deployed relacement CBs (RCBs).  Two of those teams, Shellphish and
TechX were prolific patchers, making it very difficult to draw conclusions about what led 
them to patch, or the timing of their patches.  The third team, DeepRed, appears likely
to have patched their service as a result of learning that the service crashed <sup>[1](#foot1)</sup>


CodeJitsu vs Deep Red
=====================
In round 53 a DeepRed RCB came online which included 
a re-allocated stack.  This generic technique was common to
DeepRed RCBs, and was achieved by using DECREE to allocate
a small buffer of random size, followed by a larger buffer that
is then treated as the process stack.  This second buffer is allocated
with the "no execute" (NX) bit enabled, preventing execution of
payloads within the stack.   Until now, the
CodeJitsu POV had transferred execution into a stack buffer,
which was now not possible against the DeepRed RCB. The
CodeJitsu CRS would have seen the DeepRed RCB as part of 
consensus evaluation in the previous round, and could have determined
that the POV thrown in round 53 would fail.

By round 54 CodeJitsu had reformulated the POV to use Return Oriented 
Programming (ROP) to exfiltrate protected memory content. The POV
overwrites the function pointer with the address of a library function
that is a wrapper for the DECREE transmit function.
This technique does not require execution of a payload from the
stack, and allowed CodeJitsu to score agaist the DeepRed NX defense.
However, unlike the original CodeJitus POV, the new CodeJitsu 
POV did not obfuscate the protected memory address <sup>[2](#foot2)</sup>
Curiously, after exfiltrating the protected memory content, the new POV
uses ROP to invoke the DECREE terminate function, and thus avoids crashing.
But by this time, any CRS that was paying attention has seen that someone
was scoring on this service.

Prior to the start of round 55, CodeJitsu
again changed their POV, but this time it was 
reformulated to attack a honeypot that was embedded in the
initial DeepRed RCB.  This honeypot contains a simple buffer overflow 
vulnerability that cannot be reached while the binary 
executes on CGC game infrastructure hardware, but may appear 
exploitable to dynamic analysis hosted on other platforms 
(reference to rop\_in\_cfe.md).  The game infrastrucure provides a CRS 
with feedback on the success of their POVs.  The CodeJitsu CRS 
may not have optimally respond to this feedback, and 
they continued to attack the honeypot until the service was 
retired after round 62.  

At the time it fielded the POV to attack the honeypot, the CodeJitsu
CRS had no indication that the DeepRed defense would change.  CodeJitsu
abandoned what, by all indications, was a working POV to chase the
honeypot.  However, it turns out that prior to round 55, DeepRed
submitted a filter whose sole purpose is blocking traffic containing 
protected memory addresses.  This filter came online in round 56, 
(after the consensus evaluation round), and it would have defended against the previous 
CodeJitsu POV.  As described below in the discussion of Shellphish, CodeJitsu 
could have converted their POV to a Type 1 POV that does not rely 
on either a payload or on the transmission of protected memory addresses.  

If CodeJitsu relied on their own instrumentation to assess the success
of their POV against the honeypot, the CRS would have observed successful attacks.
The honeypot buffer overflow enables a very simple Type 1 POV by loading
the EAX register with the stack value at EBP shortly before the retn instruction.  
CodeJitsu threw just such a POV, and thus the DeepRed IDS filter had no 
effect on the "success" of the POV because their was no protected memory address
to filter.


Shellphish Attacks
==================
In round 58, Shellphish deployed a POV that scored 
against the reference service, which was still being run by 
ForAllSecure and CSDS, and being owned by Codejitsu. 
The Shellphish POV uses ROP to perform a Type 1 POV.  
This POV failed against CodeJitsu and DeepRed RCBs,
because the first ROP gadget targeted by the 
corrupt function pointer was not present at
the expected address in the reformulated binaries.  

In round 59, Shellphish changed the POV thrown 
against CodeJitsu to cause the service to SEGV when
referencing the address of the function pointer, 
i.e., the "call edx".  This Type 1 POV 
succeeded against CodeJitsu.  And, it would also have 
scored against DeepRed, however in round 60, Shellphish started
targeting the DeepRed honeypot.  

Shellphish Defense
==================
The Shellphish generic defenses includes function
pointer validation as described below.  This protected
their service from the CodeJitsu POVs, and would have
protected the service from their own POVs.

In the reference service do\_update function, (part
of which is listed below), the "call edx" at 0x804EF07
is where the corrupt function
pointer leads to a POV.  The value of edx comes from
an overflowed buffer, (which occurs earlier in the process).
The value of esi also derives from that same buffer, and
the successful Shellphish Type 1 POVs negotated esi
as the general register, with a value eight greater than
the value received into the buffer.

    0804EECA     mov     eax, 80h
    0804EECF     lea     ecx, [ebp+var_A4]
    0804EED5     mov     edx, [ebp+var_14]
    0804EED8     sub     edx, 1
    0804EEDE     mov     [ebp+edx+var_A4], 0
    0804EEE6     mov     edx, [ebp+var_1C]
    0804EEE9     mov     esi, [ebp+var_18]
    0804EEEC     add     esi, 8
    0804EEF2     mov     [esp], esi
    0804EEF5     mov     [esp+4], ecx
    0804EEF9     mov     dword ptr [esp+8], 80h
    0804EF01     mov     [ebp+var_E4], eax
    0804EF07     call    edx
    0804EF09     mov     [ebp+var_E8], eax
    0804EF0F     call    get_next_update_serial
   
Shellphish wrapped some address validation around this  
as follows.
    
    0804EFED mov     eax, 80h ; ''
    0804EFF2 lea     ecx, [ebp-0A4h]
    0804EFF8 mov     edx, [ebp-14h]
    0804EFFB sub     edx, 1
    0804EFFE mov     byte ptr [ebp+edx-0A4h], 0
    0804F006 mov     edx, [ebp-1Ch]
    0804F009 mov     esi, [ebp-18h]
    0804F00C add     esi, 8
    0804F00F mov     [esp], esi
    0804F012 mov     [esp+4], ecx
    0804F016 mov     dword ptr [esp+8], 80h ; ''
    0804F01E mov     [ebp-0E4h], eax
    0804F024 push    edx
    0804F025 mov     edx, edx
    0804F027 mov     dl, [edx]
    0804F029 cmp     dl, 58h ; 'X'
    0804F02C jb      short loc_804F037
    0804F02E cmp     dl, 5Fh ; '_'
    0804F031 jbe     near ptr crash_and_burn
    0804F037
    0804F037 loc_804F037:                       
    0804F037 cmp     edx, offset off_4347C000
    0804F03D jnb     near ptr crash_and_burn_2
    0804F043 pop     edx
    0804F044 call    edx
    0804F046 mov     [ebp-0E8h], eax
    0804F04C call    near ptr unk_804F7C0

When exposed to their own POV, the process SEGV's at
0x804f027, which serves to validate the address that
would be reached by the "call edx".  By design, the POV 
sets this to an invalid address, leading to the SEGV.
A SEGV on a dereferenced general purpose register cannot
be leveraged into a Type 1 POV because EIP is not controlled.

When exposed to the CodeJitsu POV from round 54, the service
deliberately crashes at 0x804F031, which is a jump to null bytes.
That jump occurs because the value of dl is less than or equal
to 0x5F.  The value of edx following execution of 804F006 is 
804E969, which is the following code:

    0804E969 pop     eax
    0804E96A jb      short loc_804E975
    0804E96C cmp     dl, 5Fh ; '_'
    0804E96F jbe     near ptr unk_8047332

The op code for the "pop" instruction at that address is indeed less than 0x5F. 
Here, the generic defense is preventing calls to any code that
begins with a "pop" instruction, which is often the start of a
ROP gadget intended to adjust the stack. <sup>[3](#foot3)</sup>
In this case, since the Shellphish code layout
does not match that targeted by the POV, it just so happens that the
target of the call would have been a pop, which coincidentally is part
of a generic defense validation of a different function pointer invocation.

TechX Defense
=============
TechX also deployed an RCB that validated function pointers.
This RCB came online in round 53
along with a filter similar to that deployed by DeepRed.
These were never scored on.
When exposed to the CodeJitsu POV from round 54 (against DeepRed), the
TechX filter blocks the traffic containing the protected memory address.

When exposed to the Shellphish POV from round 59, generic defenses catch the
attempt to "call edx" with a corrupt address.  The CRS recompiled the service 
such that the code block containing the "call edx" is replaced with:

    080504F5 loc_80504F5:
    080504F5 sub     edx, 1
    080504FB mov     [ebp+edx+var_A4], 0
    08050503 mov     edx, [ebp+var_1C]
    08050506 mov     esi, [ebp+var_18]
    08050509 add     esi, 8
    0805050F mov     [esp], esi
    08050512 mov     [esp+4], ecx
    08050516 mov     dword ptr [esp+8], 80h ; ''
    0805051E mov     [ebp+var_E4], eax
    08050524 call    sub_80504D5
    08050529 nop

The "call edx" is replaced with a call to 0x80504D5:

    080504D5 sub_80504D5 proc near
    080504D5 push    edx
    080504D7 jmp     loc_804FD4C
    080504D7 sub_80504D

    ooo
    0804FD4C loc_804FD4C:
    0804FD4C pop     ecx
    0804FD4D cmp     byte ptr [ecx-1], 0F4h ; ''
    0804FD51 jnz     short _terminate
    0804FD53 and     ecx, 7FFFFFFFh
    0804FD59 jmp     ecx

which validates the edx register.
Since its value is an invalid address, the process SEGV's
at 080FD4D.  

The "0F4h" value is a control flow cookie that the CRS
compiled into memory just preceeding the target of function
pointers.  The test
blocks most attempts at ROP using the "call edx".  This is 
exactly what occurs when the RCB is exposed to the
first CodeJitsu POV which attempts to call into its payload, 
leading to a DECREE terminate call.

Also, note the "nop" at 8040529.  The TechX CRS compiled a "nop" to
follow every "call", and they replaced the corresponding "retn" instructions
with code to check that the retn is to a nop, thus defeating attempts to
ROP using corrupt function return values.

CodeJitsu Defense
=================
CodeJitsu fielded an RCB in round 51, (submitted in the same round they
submitted their initial POV).  This RCB contained a NX stack similar
to that used by the DeepRed RCB, and it contained stack cookies which were
of no value defending this service.  The NX stack would have defended against
their own initial POV.  When exposed to their on ROP POV, the service crashes
because the gadget is missing from the expected location.  Their
RCB did not protect against ROP attacks. And of course it was scored on by
the Shellphish POV that caused a SEGV when referencing the function pointer.

Other Competitors
=================
Disekt deployed a brick in round 53 and they let it ride. It was never scored on.

ForAllSecure was having technical difficulties during these rounds.

CSDS did not deploy an RCB.


* <a name="foot1">1</a>:  Of 34 services for which DeepRed fielded RCBs, in 14 cases the RCB came online within
three rounds of being scored upon.   In 18 cases, (22% of all services), they deployed one or more RCBs for no apparent reason.
The always brought an RCB online within three rounds of being scored except for one Type 2 POV that did not
crash, and two cases where the service was withdrawn before they could have responded.  One of the 14 cases in which 
they deployed an RCB was a silent Type 2 POV -- however in that case, DeepRed had developed their own POV, which could 
explain why they would deploy a defense.  

* <a name="foot2">2</a>: Obscuring the protected memory address without use of a stack-based payload 
could have been achieved by one of two techniques:  

    (1) Place obscured data within the ROP stack frame and find a ROP gadget
that happens to perform a desired operation, e.g., sum two values and put the results
on the stack; or,

    (2)  use ROP to invoke DECREE to allocate a memory buffer with the execute bit enabled,
and then use ROP to invoke the DECREE receive function to read in a payload similar to that
employed by the initial CodeJitsu POV.  (reference rop\_in\_cfe.md)


* <a name="foot3">3</a>:  Cyber Grand Shellphish, Phrack, January 2017.  phrack.org/papers/cyber\_grand\_shellphish.html
