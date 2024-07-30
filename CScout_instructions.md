# Instructions for running CScout on Component ID #7 of Decomposition Engine Output#
1. #### Make the project buildable as CScout needs to run the csmake script to track the building process
    1. This step involves running the installation script + getting python requirements satisfied
2. #### Create CScout temporary directory (e.g., *tmp_dir_xtensa*) and set CSMAKEFLAGS environment variable accordingly (export CSMAKEFLAGS='-T </path/to/tmp_dir_xtensa> -d --')
3. #### Make *csmake* script combatible with custom toolchains (whose direct path is hardcoded in the Makefiles) and the CMake Makefile generator  
    1. Edit the csmake script (by default in */usr/local/bin/csmake* after the installation of CScout) and get the values on the *$real* variables changed to those of the custom toolchains (see csmake of current repository as an example)
    2. Edit the CMake files in the CMakeFiles/3.25.1 directory of the component on which CScout will run to give them the values of the *tmp_dir_xtensa*
