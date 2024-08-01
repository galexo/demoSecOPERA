# Instructions for running CScout on Component ID #7 of Decomposition Engine Output
## CScout produced a call graph in .../esp-idf-v4.4.7/examples/custom_bootloader/bootloader_hooks subcomponent
1. #### Make the project buildable as CScout needs to run the csmake script to track the building process
    1. This step involves running the installation script and getting the Python requirements satisfied.
2. #### Create CScout temporary directory (e.g., *tmp_dir_xtensa*) and set CSMAKEFLAGS environment variable accordingly (export CSMAKEFLAGS='-T </path/to/tmp_dir_xtensa> -d --')
3. #### Make *csmake* script combatible with custom toolchains (whose direct path is hardcoded in the Makefiles) and the CMake Makefile generator  
    1. Edit the csmake script (by default in */usr/local/bin/csmake* after the installation of CScout) and get the values on the *$real* variables changed to those of the custom toolchains (see csmake of current repository as an example)
    2. Edit the CMake files in the CMakeFiles/3.25.1 directory of the component on which CScout will run to give them the values of the *tmp_dir_xtensa* scripts (gcc, ar, ld). (see files in custom_cmake of current repository as an example)
    3. Create a build diretory to do an *out of source* build (optional) and run CMake to generate the necessary Makefiles.
4. #### Run csmake which generates the make.cs file (csmake)
5. #### Run CScout to generate the call graph (cscout make.cs -R cgraph.txt?\<flags\>)
    1. The parser that CScout generates through *btyacc* (https://github.com/ChrisDodd/btyacc) is not compatible with all C constructs used in this component. The problems that may arise are various and need different handling per case.
        1. issue with *#include_next* directive not being processed needs to be manually fixed by including the absolute path instead (with *#include*).
        2. keywords such as **_Atomic** and **\_\_VA_OPT\_\_** are not supported by the CScout parser and thus need to be fixed by editing a local copy of the *host_defs.h* file located by default in */usr/local/include/cscout* and adding a few additional macro definitions that neutralize unknown compiler extensions. 

CScout Github repository: https://github.com/dspinellis/cscout/tree/master  
CScout User Manual: https://www.spinellis.gr/cscout/doc.html
