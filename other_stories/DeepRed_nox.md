Deep Red Random NX Stack Defense
================================

The Deep Red generic defense usually included three components:

1) A honeypot;
2) allocation of NX memory for their stack, (at a randomized location).  
3) A simple IDS filter that blocks values that might reference the magic page

The NX defense proved effective against several POVs as summarized in
the list below.  By its nature, the defense proved effective against POVs
designed to transfer execution to a payload received into the stack.  This
POV technique was employed by several teams during CFE.   

The Disekt POVs that were foiled by this defense were also blocked by
the IDS filter because Disekt turned execution control to a Type 2 POV
which made no attempt to obfuscate protected memory addresses in transmissions.

ForAllSecure was too busy chasing the honeypot to be blocked by the NX stack.
However, had they not been fooled by the honeypot, the NX stack would have
defeated the POVs they threw against other teams.  


##POVs that were (or would have been) defeated by the DeepRed NX stack##

    CROMU_00046 from Shellphish in round 61-66
        The NX clearly defeated the Shellphish POV
    CROMU_00051 from Codejitsu (in 68-72 Codejitus ate honey, but the old POV had EIP).
    CROMU_00055 from FAS, rounds... no, they were eating honey.
    CROMU_00064 from Disekt (FAS ate honey)    4-21
        However, the Disekt POV was a type 2, with 4347c easily blocked by the DeepRed filter.
    CROMU_00094 from Codejitsu (FAS ate honey)  5-21 (after that that fielded a brick FNAR)
    CROMU_00095 from FAS, rounds 9-19, except FAS was eating honey. Then DR fielded a brick. The FAS POV would/may have failed.
    KPRCA_00065 from Disekt  32-36 (FAS was eating honey)
        However, the filter would have blocked the Disekt Type 2
