An Explanation of analysis tool extracts
========================================
The post event analysis discussions often include extracts from the CGC Monitor analysis tool.
The syntax of these extracts can be understood by considering an example:

    added Type 1 POV, eip: 554187, ebp: 76b99cb : c1ca6e4
    added backtrack START:0x804b358:ret track addr: 0xbaaaaf48, value 0x554187 : c1ca6e4
    added backtrack:0x804f527:mov byte ptr [eax],dl : c1ca670
    added backtrack:0x804f517:mov dl,byte ptr [eax] : c1ca66b
    added backtrack:0x804fd22: follows kernel write of 0x4187 to 0xb7fff0f4 : c0cb695
    
The first line indicates that a Type 1 POV occurred with EIP value 0x445187, the general register
negotiated was ebp, and its value is 0x76b99cb at the time of the segmentation fault.
The final field in that line is the cycle count of the emulated processor, relative to when
the challenge binary began running.  This field occurs in every line, and inspection shows
that the lines are moving backward in time.

The second line is the start of a reverse taint tracking operation initiated by an analyst.
This second line can be read as:  The operation starts while the simulation is at EIP 0x804b378,
a "ret" instruction.  The analyst has requested a trace of the content of address 0xbaaaaf48, whose
content is 0x554187.

The next two lines reflect movement from dl into the function return address, proceeded by
the movement from the address named by eax into dl.
Note that the CGC Monitor reverse taint tracking generally watches a single byte, it does not branch
to track the full word.  

The final line identifies a "receive" system call that brought the tracked value into memory.  The EIP
of 0x804fd22 is the instruction that follows the "int 80h" of the system call.
