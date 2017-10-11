Automated Analysis of POVs
==========================

Each successful POV from CFE was processed by the CGC Monitor
automated analysis function to identify the location of
interesting events in the course of exploitation of a service.
Location information is captured in the form of EIP addresses and
an instruction number derived from an execution trace of the
service.  The results are in [auto_analysis/)](auto_analysis/).

Interesting events include:

    * Overwrite of a return address
    * Execution of an address from the stack area
    * Return to an instruction that does not follow a call
    * Reading of protected memory in the course of a Type 2 POV
    * Transmission of protected memory values

In most cases, the event information also includes data flow
back-traces that identify instructions that move tracked data between
registers and memory.  For example, such a back-trace might illustrate
the flow of a corrupt return address value back to where it was
received via a syscall.  Back-traces of data are included for the
following:

    * Corrupted return addresses
    * Corrupt values of call registers
    * Executable payloads
    * General register values negotiated in Type 1 POVs
    * The source of protected memory addresses

The results of this analysis is captured in a set of JSON files 
having a naming convention as follows:

       CSID-thrower-defender-round-throw.json

where thrower, defender, round and throw are integers representing
the team that submitted the POV, the team running the service, the
round number and the throw number within that round.  The data set contains
only one throw number per CSID/thrower/defender/round tuple.

Each of these json files has a corresponding .txt file that is simply
a human readable display of the json file with explanatory text.  An example follows:


    Automated analysis results for CSID: CROMU_00046 throw_team: 6  defend_team: 2  round: 66  throw_number: 1 first_syscall: 8070
    POV Type 1  eip:0x634aa17b eax:0xbbbd8486  instruction_number: 81442
    Execution control corruption eip:0x804866b  return to address:0xbaaaaf2d  esp:0xbaaaaf48  instruction_num:81439
            Reverse data track from eip:0x804866b tracking source of memory:0xbaaaaf48 value:0xbaaaaf2d
                    0x8048338   mov dword ptr [ebp+edx*4-0x58],eax      ; 79443
                    0x8048ee2   mov eax,dword ptr [eax]                 ; 79437
                    0x8048edd   mov dword ptr [ecx],eax                 ; 79435
                    0x8048ed8   or eax,ecx                              ; 79433
    NOX at eip:0x00000000baaaaf2d  "mov eax,0xbbbd8486" instruction_num:81440  -- source of payload:
            Reverse data track from eip:0xbaaaaf2d tracking source of memory:0xbaaaaf2d value:0xbd8486b8
                    0x8049734   mov byte ptr [ecx],bl                   ; 79749
                    0x804972e   mov bl,byte ptr [ebp-0x16]              ; 79747
                    0x80498e9   receive syscall (return)                ; 79742


The data traces are in reverse time order, as reflected by the trace instruction numbers to the right of each disassembled instruction.
The analysis tool follows data flow until it encounters a data transformation operation, (other than a simple offset adjustment),
such as the "or eax,exc" instruction at trace line 79433 in the trace of the return address overwrite value.

The [analysisDump.py](analysisDump.py) script creates human readable dump of a json file, and [dumpAll.py](dumpAll.py) processes
all of the json files.

## Limitations ##
A few Type 2 POVs do not make it to the point of negotiating the protected memory value, due to TCP-related divergence between the
CGC Monitor execution and the CFE infrastructure execution.  Analysis of these POVS will still show execution control path
corruption where it occurs, and may guess as to which protected memory reference was part of the POV.  The json files for such
conditions will lack a "proof=True" value in the POV entry.

And, a few POVs entirely fail to occur in the CGC Monitor, due to the POV transmiting and exiting, leading the service to see
a TCP recet prior to reading the full POV payload.  The correspondig json file include a "no\_event" entry.
