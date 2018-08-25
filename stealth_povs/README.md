Did "Silent" Type 2 POVs Delay Defensive Responses in CFE?
==========================================================
This note explores whether the use of "stealth" offensive
techniques provided any advantage in CFE.

The CRS received feedback indicating service crashes.
Additionally, network tap traffice could be fed to instrumented
instances of services to detect crashes, e.g., a SEGV during
a Type 1 POV.

A Type 2 POV does not necessarily include a crash of the service.
For example, after gaining control of EIP, a POV can exfiltrate
the protected memory and then exit, spin or wait using fdwait.

Thus, in theory, deploying "silent" Type 2 POVs might delay
defensive responses from your opponents.   While it is the case
that teams tended to deploy RCBs in response to attacks, this analysis finds
very little evidence of differences between responses to silent POVs
and POVs that lead to crashes.

The attached files, one per team, include entries for each 
service for which the team was scored upon prior to deploying a RCB.
The nature of the POVs that scored on the service are indicated
as being either "silent", or "SEGV".  And the number of rounds
between failing to defend and deploying an RCB are noted.

## Other advantages ##
Another theoretical advantage of a silent POV is that competitors
could not easily learn from the POV and use it to construct their
own POV for a service.  However, there does not appear to be
evidence of any CRS doing this, complicating investigation of
whether stealth prevented it.

## Who threw silent POVs? ##
Some Type 2 POVs were silent due to the nature of the vulnerability,
e.g., an over-read that leaks the protected memory page, without execution
control ever being gained by the POV.  Other Type 2 POVs 
required effort to keep silent because they included
control of EIP.   After execution control is gained, and
protected memory is leaked, then what?  Many POVs cared not, which
led to crashes.  There were 472 Type 2 POVs in which control of
execution was gained. About half of those did not lead to crashes,
and are referred to below as "controlled silent POVs".  
DeepRed and ForAllSecure accounted for most of the controlled silent POVs.

ForAllSecure threw 106 controlled silent POVs. Their  payloads execute EBFE, 
(jump to EIP), after writing protected memory. 
This puts the service into a tight loop, potentially affecting the availability
of other services executing on the opponent's defended host.
They deployed silent POVs for two services, CROMU_00055, CROMU_00064, 
CROMU_00096, CROMU_00097

DeepRed used a controlled silent POV against : CROMU_00064, CROMU_00094 and YAN01_00016.
Their 83 silent POVs generally used the DECREE *terminate* system 
call to exit after transmitting content of
protected memory.  This sometimes affected the reliability of their POVs
because a process exit leads to a TCP reset being received by the kernel
of the system that is executing the POV.  This can cause the kernel to close
the TCB port prior to receipt of all the data sent by the service, leaving the
POV without the data necessary to negotiate the POV.  This does not appear to
have prevented DeepRed from scoring on at least one of the ten throws they were 
permitted for a given round against a give competitor service.  

Disekt threw 21 controlled silent POVs.
Disekt POV payloads used the DECREE terminate system call to exit after 
transmitting protected memory content.  

Last, but not least, Codejitsu threw 16 controlled silent POVs, some of which
use ROP to "return" to the *terminate* function, resulting in an exit via
the DECREE system call.  

## Hemming and Hawing ##
Factors that complicate the ability to draw conclusions from any
of this include:

* A CRS may deploy a defense based on vulnerabilities that it discovered
* About 75% of POVs were not silent, (about half of all POVs were Type 2.)
* The analysis ignores cases where the first failure to defend occurs 
just before withdrawal of the service.  
* Teams deployed RCBs for some services for which they were never scored on, 
(and never deployed a POV).
* The two YAN01 services (20% of the samples) cannot easily be 
defended without failing availability, and thus a CRS may have chosen to not deploy an RCB.
* A CRS may have instrumented services allowing it to detect "silent" Type 2 POVs.
