Attempting to Compile (Try 1)
===
_Note: I was decidedly doing *the wrong thing* for a portion of Try 1. 
If you're trying to make this work for yourself, it's best to skip
over this section. I keep it here only for historical reference._

First, for whatever reason, I had broken packages on my system,
so I had to do a
    sudo apt --fix-broken install
then, I installed cmake with
		sudo apt install cmake

Trying to run the cmake command listed in the Readme.md resulted
in errors because the -i option is no longer used. I opted to 
try to install cmake-gui, but later decided that this wasn't
worth it. Just leave off the -i option.
Leaving off that option, there were still errors when I tried
to compile.

		(base) karl@peter:~/gitSpace/CLEED$ cmake -G "Unix Makefiles" --build build/
		CMake Error: File /home/karl/gitSpace/CLEED/src/src/include/caoi_leed.h.in does not exist.
		CMake Error at aoi_leed/CMakeLists.txt:24 (CONFIGURE_FILE):
			CONFIGURE_FILE Problem configuring file
	
	
		CMake Error: File /home/karl/gitSpace/CLEED/src/src/include/caoi_rfac.h.in does not exist.
		CMake Error at aoi_rfac/CMakeLists.txt:24 (CONFIGURE_FILE):
			CONFIGURE_FILE Problem configuring file
	
	
		CMake Error: File /home/karl/gitSpace/CLEED/src/src/include/ftsmooth.h.in does not exist.
		CMake Error at ftsmooth/CMakeLists.txt:24 (CONFIGURE_FILE):
			CONFIGURE_FILE Problem configuring file
	
	
		CMake Error: File /home/karl/gitSpace/CLEED/src/src/include/leed_ver.h.in does not exist.
		CMake Error at leed_nsym/CMakeLists.txt:24 (CONFIGURE_FILE):
			CONFIGURE_FILE Prob

I noticed that the ".../src/src/..." piece seemed like an error, 
so I edited each of the files it noted in the output.
I concisely listed them with 

		cmake -G "Unix Makefiles" --build build/ |& grep CMakeLists.txt

You may have seen the "|" operator before, called a pipe, which
sends the output of one program to another. When I 
first tried to do this command, I used the standard "|", but
there was no output to the grep program. Since the output
was a collection of error messages, it was being sent to 
stderr, which is also the screen. Using the "|&" operator
pipes both the output and the error to the next program.
I should also note that I discovered this by using the man
pages. I typed

		man bash

Then searched (by typing "/" followed by my query, "|") for the 
pipe in the documentation to come across that gem.

I then edited the group of files in a single vi terminal,
allowing me to quickly repeat commands that were the same in 
each file. (searching for \/src and deleting the two instances
in which it appeared). 

Now I got all of the /src/src errors eliminated, the remaining 
error was:

		(cleed) karl@peter:~/gitSpace/CLEED$ cmake -G "Unix Makefiles" --build build/
		CMake Error at /usr/share/cmake-3.10/Modules/FindQt4.cmake:1320 (message):
		  Found unsuitable Qt version "5.9.7" from /home/karl/anaconda3/bin/qmake,
		  this code requires Qt 4.x
		Call Stack (most recent call first):
		  latt-gui/CMakeLists.txt:1 (FIND_PACKAGE)
		
		
		CMake Warning (dev) in CMakeLists.txt:
		  No cmake_minimum_required command is present.  A line of code such as
		
		    cmake_minimum_required(VERSION 3.10)
		
		  should be added at the top of the file.  The version specified may be lower
		  if you wish to support older CMake versions for this project.  For more
		  information run "cmake --help-policy CMP0000".
		This warning is for project developers.  Use -Wno-dev to suppress it.
		
		-- Configuring incomplete, errors occurred!
		See also "/home/karl/gitSpace/CLEED/build/CMakeFiles/CMakeOutput.log".

I was convined this was an anaconda problem, so spent a few
hours trying to figure out how to uninstall Qt5. Side note:
is there a way to force anaconda to downgrade your installation 
to accomodate a certain package that you want to install?
This seems like a very convenient feature that is absent.

In any case, I eventually had the realization that I didn't 
need to install Qt4 in Anaconda, but on my operating system.
So I ran the following command to get that installed.

		sudo apt install qt4-dev-tools

Then cmake successfully ran! Now onto the next step. Navigate
to the build directory and run 

		make install

Which resulted in the following output.

		(base) karl@peter:~/gitSpace/CLEED/build$ make install
		Scanning dependencies of target caoi_leed
		[  0%] Building C object aoi_leed/CMakeFiles/caoi_leed.dir/caoi_leed_bsr.o
		/home/karl/gitSpace/CLEED/src/aoi_leed/caoi_leed_bsr.c:16:10: fatal error: caoi_leed.h: No such file or directory
		 #include "caoi_leed.h"
		          ^~~~~~~~~~~~~
		compilation terminated.
		aoi_leed/CMakeFiles/caoi_leed.dir/build.make:62: recipe for target 'aoi_leed/CMakeFiles/caoi_leed.dir/caoi_leed_bsr.o' failed
		make[2]: *** [aoi_leed/CMakeFi

After some searching, I found the .h files in the src/include folder.
I knew that if these .h files were in the same directory 
as the .c file that is trying to compile, the compilier would
no longer complain that it couldn't find them. I ran the following 
commands to accomplish that and again attempt to compile:

		(base) karl@peter:~/gitSpace/CLEED/build$ cp ../src/include/caoi_leed.h ../src/aoi_leed/
		(base) karl@peter:~/gitSpace/CLEED/build$ make install
		[  0%] Building C object aoi_leed/CMakeFiles/caoi_leed.dir/caoi_leed_bsr.o
		/home/karl/gitSpace/CLEED/src/aoi_leed/caoi_leed_bsr.c: In function ‘bsrinp’:
		/home/karl/gitSpace/CLEED/src/aoi_leed/caoi_leed_bsr.c:55:11: warning: implicit declaration of function ‘strlen’ [-Wimplicit-function-declaration]
		  length = strlen(filebsr) - 4;
		           ^~~~~~
		/home/karl/gitSpace/CLEED/src/aoi_leed/caoi_leed_bsr.c:55:11: warning: incompatible implicit declaration of built-in function ‘strlen’
		/home/karl/gitSpace/CLEED/src/aoi_leed/caoi_leed_bsr.c:55:11: note: include ‘<string.h>’ or provide a declaration of ‘strlen’
		
We no longer see the error! There are a few warning messages, 
which might be cause for concern later, but likely won't 
cause any issues because this was a working program at one time after
all.

Further down in the ouput, there was another fatal error, this
time for a different .h file. Using a similar process of copying
the files to the appropriate places, I gradually made it further 
and further into the compile process. I eventually got to 
8% complete, when a new error showed up.

		[  7%] Building C object latt/CMakeFiles/latt.dir/latt_help.c.o
		[  7%] Building C object latt/CMakeFiles/latt.dir/latt_parse_args.c.o
		[  8%] Linking C executable latt
		/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/Scrt1.o: In function `_start':
		(.text+0x20): undefined reference to `main'
		CMakeFiles/latt.dir/lattice.c.o: In function `lattice_read':
		lattice.c:(.text+0x1da1): undefined reference to `inf_stream'
		lattice.c:(.text+0x1dc7): undefined reference to `ctr_stream'
		lattice.c:(.text+0x1e7f): undefined reference to `inf_stream'
		collect2: error: ld returned 1 exit status
		latt/CMakeFiles/latt.dir/build.make:198: recipe for target 'latt/latt' failed
		make[2]: *** [latt/latt] Error 1
		CMakeFiles/Makefile2:383: recipe for target 'latt/CMakeFiles/latt.dir/all' failed

This one felt familiar, as I have seen the "undefined reference to 'main'"
line before. That fix is easy enough, we just added an empty
main() function to the lattice.c program and that error no longer 
appeared. The next two lines mentioning undefined references to 
_stream variables was a bit more tricky. I did notive that Liam Deacon
had noted that he made adjustments to the lattice file, and that 
encouraged me that he might have not finished changing things.
Comparing latt.c and lattice.c, I found similar variables, so 
I decided to give them the same values in each program and hoped
that worked- it did!

Getting past that error resulted in a new error- again with the
.h files.

All of these errors led me to believe there must be a more straightforward way.
I abandoned this attempt and decided to start from scratch.

Attempting to Compile (Try 2)
===
As is the custom, I make a lot of mistakes while I'm trying to install something the first time, 
then the next time I install things go a bit easier. I learned that 
the python related errors I got on my first try were caused by conflicts between 
my existing python installation and packages automatically installed by the installer
by creating a new anaconda environment, I was able to completely bypass those errors.

There are some files that are included in the GitHub repository that are
configuration files that will necessarily differ between systems but throw errors 
when you are trying to compile.
These should be removed from the repository, and I have done this.

Refined the syntax of the cmake command according to the updated program 
and added this information to the README file.

The biggest break seems to be in the pattern, patt, and lattice files, which 
it appears Liam was in the middle of updating when he initially committed the 
repository. He seemed to have got things working at some point though, so I'm 
still hopeful that if I pick the right commit, I should be able to get things 
up and running with minimal modifications to the code. 

Next step: Attempt to install from scratch on the "Happy 2015 CLEED" commit.
This will be my last attempt at picking a random commit to see if things work. 
After this, I'm just going to focus on resolving all of the compile errors on
whichever version gets me to the highest compile percentage.

UPDATE (2022-02-21): 
Still getting compile errors, the same old song and dance. I thought I had 
resolved the python errors by using the correct version of python, but I still 
have to go in and manually correct the curexec lines. I guess I've never 
explictily documented this. Here's the output:

```
/home/karl/gitSpace/CLEED6/CLEED/build/src/phaseshifts-0.1.3-dev/phaseshifts/cython/phsh.cpp:10459:13: error: ‘PyThreadState {aka struct _ts}’ has no member named ‘exc_type’; did you mean ‘curexc_type’?
     tstate->exc_type = type;
             ^~~~~~~~
             curexc_type
```

I attempted rolling python back to earlier versions, but then I got another
error. I'm officially in error purgatory. I tried to recreate this error
and instead got another error. I need to abandon this project for the time 
being.


Attempting to Compile (Try 3)
===
I decided to again start a new attempt COMPLETELY from scratch on a new laptop.
In doing this, I learned that Ubuntu 20.04 dropped Qt4 support, which this program
relied on. It then seems important that I'm able to get the program running so that
I can attempt to port Qt4 over to Qt5 so that the program can still run on modern
systems. I'm canceling my attempt on Try 3 and going back to Try 2, until I'm
successful there. When I have the program running, then I'll see about converting
it over to Qt5.
