Effects of Deep Red's Honeypot on CFE
=====================================
The CGC issue of IEEE S&P includes two articles on this honeypot:
One from Deep Red: <http://ieeexplore.ieee.org/document/8328978/>
And one from yours truely: <http://ieeexplore.ieee.org/document/8328967/>

My article "Effects of a Honeypot on the Cyber Grand Challenge Final Event" 
incorrectly states that a logic error in Rubeus's deployment of patches 
containing the honeypot led to service crashes.  A subsequent exchange with 
a Rubeus developer has clarified that the bug was with the overall patcher 
itself, and not with the honeypot logic.  The patcher bug manifested
when multiple patches were part of the same RCB.  Rubeus evaluated individual 
patches prior to deployment, but did not evaluate the overall RCB, and thus 
did not notice that their RCB would crash. The choice to not evaluate 
RCBs was driven by Deep Red's plan to use RCBs to execute malicious programs 
to attack competitor CRSes, which they abandoned shortly before CFE upon 
being reminded that is a violation of CGC rules.  Evaluating the full 
RCB would have required additional effort to protect their own CRS from 
that malicious software.

