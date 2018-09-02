Some PoVs are missing from the CGC Archive
==========================================
The archive at https://www.lungetech.com/cgc-corpus is missing links to
some of the PoVs submitted by competitors.  However, all of the PoVs
do exist at the GitHub corpus at https://github.com/lungetech/cgc-cfe-submission-corpus,
albeit without information about who threw what at whom and when. Information
about how to find the missing files is included at the end of this note.

## Example missing files ##
In round 56, none of the CodeJitus PoVs against Deep Red appear in the PoV
sections of the archive.  For example, this POV was thrown by CodeJitsu against Deep Red's  
NRFIN_00063 service in round 56:
    2347155893-310730c3b3d08e1bc972633c23a7f808f2cb6d838c156b8f78e3fb2a663b67c1.pov
but there is no link to it at https://www.lungetech.com/cgc-corpus/challenges/NRFIN_00063/
This PoV can be found at
https://github.com/lungetech/cgc-cfe-submission-corpus/raw/master/NRFIN_00063/2347155893-310730c3b3d08e1bc972633c23a7f808f2cb6d838c156b8f78e3fb2a663b67c1.pov

## OMG are the scores wrong? ##
No.  None of the missing PoV's scored.  But that does not mean they are not interesting.  For example,
some of the PoVs attack the honeypot that Deep Red deployed.  Such is the case for the PoV referenced above.
That PoV played a key role in the saga described here:
https://github.com/mfthomps/CGC-Analysis/blob/master/battleOfGenerics.md
That interchange between Deep Red and CodeJitsu has been characterized as the most interesting
interaction within CGC.  And it may have been.  However, the official telling of that tale ends with
Deep Red deploying a filter, suggesting that filter foils CodeJitsu's latest PoV. In fact, before CodeJitsu
could have known about the upcoming filter, they switched out their working PoV and chased the honeypot
with the PoV referenced above.  That behavior runs counter to the narrative of autonomous systems
adapting to changes in offense and defense in real time.

## The PoV contextual information ## is available at:
https://raw.githubusercontent.com/mfthomps/cgc-monitor/master/cgc-monitor/scoreUtils/povs_by_round.csv
That csv includes round,defender,csid,thrower,num_throws,file_name
Append the common name of the service, e.g., NRFIN_00063, and the filename to this URL:
https://github.com/lungetech/cgc-cfe-submission-corpus/raw/master/
The team map is at https://github.com/mfthomps/cgc-monitor/blob/master/cgc-monitor/scoreUtils/TeamMap.txt
And the CSID/common name map is at https://github.com/mfthomps/cgc-monitor/blob/master/cgc-monitor/scoreUtils/cbmap.txt




