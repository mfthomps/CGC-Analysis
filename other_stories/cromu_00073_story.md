CROMU_00073
===========

The most interesting thing about this service is that
all three competitors who scored on it, threw at least one
POV that uses a return address overwrite to transfer execution 
to the very same code block.

The three competitors employed six different techniques
to prove vulnerabilities subsequent to gaining control of EIP.

## Summary of CROMU_00073 in CFE ##
All POVs followed the very same unintended path to 
exploitation of the vulnerability.  
Three teams landed against the reference service, and
all POVs were extremely unreliable.
No team was able to score against any RCB.
All of the RCBs seem to have deployed some kind of generic
defenses.  


## Variations in strategies after gaining EIP ##
All POVs from the three competitors that scored on this service 
overwrote the return address of the "handle_double" function,
which exits with:

    ooo 
    080482BB add     esp, 10h
    080482BE mov     ecx, [ebp+var_C]
    080482C1 mov     esp, ecx
    080482C3 mov     esp, ebp
    080482C5 pop     ebp
    080482C6 retn
    080482C6 handle_double endp


### CodeJitsu ###

* #### Round 23 vs CSDS, TechX and DeepRed (throw 3): ####
    
  + Used the corrupt handle_double return address
to transfer control to 8050313 within this trampoline code block,
which ends with the equivalent of call [ecx]

            0805030B ; ---------------------------------------------------------------------------
            0805030B mov     edx, [esp+4]
            0805030F mov     eax, [esp+8]
            08050313 mov     ecx, [edx]
            08050315 mov     ebx, [edx+4]
            08050318 mov     esp, [edx+8]
            0805031B mov     ebp, [edx+0Ch]
            0805031E mov     esi, [edx+10h]
            08050321 mov     edi, [edx+14h]
            08050324 test    eax, eax
            08050326 jnz     short loc_8050329
            08050328 inc     eax
            08050329
            08050329 loc_8050329:                            ; CODE XREF: .text:08050326j
            08050329 mov     [esp], ecx
            0805032C retn
        
  + The service SEGVs after the retn at 805032C: Type 1 POV eip:0x822cc6e6 ebx:0x8442aa28
    
    
* #### Round 28 vs DeepRed (throw 10) (Note use of ret-to-libc) ####
  + Uses the trampoline to return to the tx function at 8048d80,
to send protected memory content directly from the magic page.
(No stealth advantage because the service crashes after the xmit.)
    
* #### Round 29 vs DeepRed ####
  + Back to the old Type 1 POV
    

### Disekt ###

* ####Round 29 vs DeepRed (throw 1)####
  + Used the same trampoline code as CodeJitsu above,
and also entered at 8050313
The retn at 805032C transfers to a payload in the stack, which
transmits from the magic page.  
(No stealth advantage because the service crashes after the xmit.)

### ForAllSecure ###

* #### Round 22 vs CSDS (throw 1) ####
  + Used the same trampoline code as CodeJitsu above,
but entered at 805030B.
The service SEGVs after the retn at 805032C: POV Type 1  eip:0x7742df18 esi:0x3c3a22f0 

* #### Round 22 vs DeepRed and TechX (throw 1) ####
  + Used the corrupt handle\_double return address to transfer
control to a payload in the stack.  This payload immediatly
transfers to a gadget that does four pops and a retn,
leading to a SEGV: POV Type 1  eip:0xdc15f652 esi:0x5cd5320a 


* #### Round 23 vs CSDS and TechX (throw 1) ####
  + Same as vs DeepRed round 22

* #### Round 23 vs DeepRed (throw 1) ####
  + Used the corrupt handle\_double return address to transfer
execution to a payload in the stack:  

        BAAAAEF6 mov     eax, offset unk\_5CD5320A
        BAAAAEFB mov     ecx, offset unk\_DC15F652
        BAAAAF00 jmp     ecx

  + This simple payload led to a SEGV: POV Type 1  eip:0xdc15f652 eax:0x5cd5320a 

* #### Round 28 vs CodeJitsu (throw 1) ####
  + Used the corrupt handle\_double return address to immediatly
SEGV: POV Type 1  eip:0xa68d5503 ecx:0x6a9c0c3a

* #### Round 28 vs DeepRed (throw 1) ####
  + Same as round 22 vs DeepRed (pop-pop-pop-pop-retn)




