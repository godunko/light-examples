# Getting Started with Ada and `light` runtime on Arduino Due

## Prerequsites

You need have installed:
  * Alire package manager (see https://alire.ada.dev/)
  * BOSSAC utility
  * Debug probe and OpenOCD utility (optional)

### Create and configure project with `Alire`

Initial structure of the project can be created by the Alire tool. You need to run


    $ alr init --bin --no-skel arduino_due_blink


For the first time `Alire` will ask few questions. Answer all of them.


    Alire needs some user information to initialize the crate author and maintainer,
    for eventual submission to the Alire community index. This information will be 
    interactively requested now.
    
    You can edit this information at any time with 'alr config'

    Please enter your GitHub login: (default: 'github-username')
    > godunko
    Please enter your full name: (default: 'Your Name')
    > Vadim Godunko
    Please enter your email address: (default: 'example@example.com')
    > vgodunko@gmail.com
    ✓ arduino_due_blink initialized successfully.
    

It creates `arduino_due_blink` directory and `alire.toml` file inside it. It is
minimal set of files to use `Alire` to manage project (`--no-skel switch was
used to don't fill non-Alire files).

Enter into the created directory and use `Alire` to add dependency from the
`light_arduino_due` crate, which provides startup files and linker script to
build application to run on Arduino Due board using `light` GNAT runtime.

    $ alr with light_arduino_due


`Alire` download index information and will ask question about compiler:

    Cloning into '/home/godunko/.config/alire/indexes/community/repo'...
    remote: Enumerating objects: 9467, done.
    remote: Counting objects: 100% (1888/1888), done.
    remote: Compressing objects: 100% (363/363), done.
    remote: Total 9467 (delta 1627), reused 1593 (delta 1512), pack-reused 7579
    Receiving objects: 100% (9467/9467), 1.64 MiB | 1.20 MiB/s, done.
    Resolving deltas: 100% (5334/5334), done.
    Welcome to the toolchain selection assistant

    In this assistant you can set up the default toolchain to be used with any crate
    that does not specify its own top-level dependency on a version of gnat or 
    gprbuild.

    If you choose "None", Alire will use whatever version is found in the 
    environment.

    ⓘ gnat is currently not configured. (alr will use the version found in the environment.)

    Please select the gnat version for use with this configuration
      1. gnat_native=13.2.1
      2. None
      3. gnat_arm_elf=13.2.1
      4. gnat_avr_elf=13.2.1
      5. gnat_riscv64_elf=13.2.1
      6. gnat_arm_elf=13.1.0
      7. gnat_avr_elf=13.1.0
      8. gnat_native=13.1.0
      9. gnat_riscv64_elf=13.1.0
      0. gnat_arm_elf=12.2.1
      a. (See more choices...)
    Enter your choice index (first is default): 
    >


Look for `gnat_arm_elf` of latest version (`13.2.1`) and enter its number,
press Enter. `Alire` will ask for version of the `gprbuild` utility to use:

    > 3
    ⓘ Selected tool version gnat_arm_elf=13.2.1

    ⓘ Choices for the following tool are narrowed down to releases compatible with just selected gnat_arm_elf=13.2.1

    ⓘ gprbuild is currently not configured. (alr will use the version found in the environment.)

    Please select the gprbuild version for use with this configuration
      1. gprbuild=22.0.1
      2. None
      3. gprbuild=21.0.2
      4. gprbuild=21.0.1
    Enter your choice index (first is default): 
    > 

Look for latest version, enter its number and press Enter. Now `Alire` will
download and install compiler and `gprbuild` utility. It might take time.

    > 1
    ⓘ Selected tool version gprbuild=22.0.1
    ⓘ Deploying gprbuild=22.0.1...                   
    ############################################################################################################## 100,0%
    ⓘ gprbuild=22.0.1 installed successfully.
    ⓘ Deploying gnat_arm_elf=13.2.1...
    ############################################################################################################## 100,0%
    ⓘ gnat_arm_elf=13.2.1 installed successfully.


Next, `Alire` resolves all dependencies of the `light_arduino_due` crate,
report resolved dependencies and ask for confirmation:

    ⓘ Synchronizing workspace...
    Nothing to update.          

    Requested changes:                                            

       ✓ light_arduino_due ~0.1.0 (add)

    Changes to dependency solution:

       +♼ gnat_arm_elf      13.2.1 (new,installed,indirect)
       +  light_arduino_due 0.1.0  (new)                   

    Do you want to proceed?
    [Y] Yes  [N] No  (default is Yes) 

Type `Y` and press `Enter`.

    [Y] Yes  [N] No  (default is Yes) y
    ⓘ Deploying light_arduino_due=0.1.0...
    Cloning into '/home/godunko/Hexapod/light-examples/arduino_due_blink/alire/cache/dependencies/alr-gwxs.tmp'...
    remote: Enumerating objects: 249, done.
    remote: Counting objects: 100% (249/249), done.
    remote: Compressing objects: 100% (156/156), done.
    remote: Total 249 (delta 108), reused 209 (delta 71), pack-reused 0
    Receiving objects: 100% (249/249), 90.54 KiB | 648.00 KiB/s, done.
    Resolving deltas: 100% (108/108), done.

Now development environment is ready to start work on the project. Don't forget
to install other necessary tools (BOSSAC/OpenOCD).


### Project file and dummy main

Lets create project files to build and link example application. Project file
should have same name as project with `.gpr` extension
(`arduino_due_blink.gpr`). Here is mandatory content of the file:

    with "arduino_due_startup";

    project Arduino_Due_Blink is

       for Target use Arduino_Due_Startup'Target;
       for Runtime use Arduino_due_Startup'Runtime;

       ...

       package Linker is
          for Switches ("Ada") use Arduino_Due_Startup.Linker_Switches;
       end Linker;

    end Arduino_Due_Blink;

We also want to generate debuger information, use of Ada 2022 features,
minimize size of the application in flash memory. So, lets add a `Compiler`
package and modify `Linker` package.

    project Arduino_Due_Blink is

       ...

       package Compiler is
          for Switches ("Ada") use
            ("-g",                   --  Debug information
             "-gnat2022",            --  Ada2022 features
             "-ffunction-sections",  --  Place each function in its own section
             "-fdata-sections");     --  Place each data item in its own section
       end Compiler;
    
       package Linker is
          for Switches ("Ada") use
            Arduino_Due_Startup.Linker_Switches
    	       & ("-Wl,--gc-sections");  --  Perform a garbage collection of code and data never referenced
       end Linker;

    end Arduino_Due_Blink;

Last missing lines in the project files is a name of the main subprogram and
name of the executable in ELF format:


    project Arduino_Due_Blink is

       ...

       for Main use ("main.adb");

       package Builder is
          for Executable ("main.adb") use "main.elf";
       end Builder;

       ...

    end Arduino_Due_Blink;

Here is full content of the `arduino_due_blink.gpr` file:

    with "arduino_due_startup";

    project Arduino_Due_Blink is

       for Target use Arduino_Due_Startup'Target;
       for Runtime use Arduino_due_Startup'Runtime;

       for Main use ("main.adb");

       package Builder is
          for Executable ("main.adb") use "main.elf";
       end Builder;

       package Compiler is
          for Switches ("Ada") use
            ("-g",                   --  Debug information
             "-gnat2022",            --  Ada2022 features
             "-ffunction-sections",  --  Place each function in its own section
             "-fdata-sections");     --  Place each data item in its own section
       end Compiler;

       package Linker is
          for Switches ("Ada") use
            Arduino_Due_Startup.Linker_Switches
    	       & ("-Wl,--gc-sections");  --  Perform a garbage collection of code and data never referenced
       end Linker;

    end Arduino_Due_Blink;


Now lets create dummy `main.adb` file:

    procedure Main is
    begin
       null;
    end Main;

And it is time to build application!

    $ alr build

    ⓘ Building arduino_due_blink/arduino_due_blink.gpr...
    Compile
       [Ada]          main.adb
       [Ada]          system_armv7m-startup_utilities.adb
       [Ada]          system_armv7m-mpu.ads
       [Ada]          system_armv7m-scb.ads
       [Ada]          system_armv7m.adb
       [Ada]          system_armv7m-cmsis.adb
       [Ada]          system_types.ads
       [Ada]          system_armv7m-systick.ads
       [Ada]          atsam3x8e-efc.ads
       [Ada]          system_atsam3x8e-startup_utilities.adb
       [Ada]          atsam3x8e-pmc.ads
       [Ada]          system_atsam3x8e.adb
       [Ada]          atsam3x8e.ads
       [Ada]          system_arduino_due.adb
    Build Libraries
       [gprlib]       arduinodue.lexch
       [archive]      libarduinodue.a
       [index]        libarduinodue.a
    Bind
       [gprbind]      main.bexch
       [Ada]          main.ali
    Link
       [link]         main.adb
    Build finished successfully in 0.54 seconds.

That's it! Executable file in ELF format can be found in `main.elf` file.
