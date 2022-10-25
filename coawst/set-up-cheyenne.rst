###############################
Set up COAWST model on Cheyenne
###############################

This page contains brief notes about compiling COAWST on NCAR's Cheyenne
machine.

.. note::

   This was much easier to do on Cheyenne than it was on Shaheen so the 
   instructions are simpler.

.. code-block::

   cd /glade/work/johnsonb
   git clone https://github.com/jcwarner-usgs/COAWST.git
   cd COAWST
   module load openmpi
   cd /glade/work/johnsonb/COAWST/Lib/MCT
   ./configure
   [ ... ]
   Please check the Makefile.conf
   Have a nice day!
   cp mpeu/libmpeu.a mct/
   cd ../../Compilers
   vim Linux-ifort.m
   # Set environmental variables as shown here:
   227        MCT_INCDIR ?= /glade/work/johnsonb/COAWST/Lib/MCT/mct
   228        MCT_LIBDIR ?= /glade/work/johnsonb/COAWST/Lib/MCT/mct
   cp coawst.bash Projects/Inlet_test/
   cd Projects/Inlet_test
   vim coawst.bash
   # Set environmental variable as shown here:
   132 export   MY_ROOT_DIR=/glade/work/johnsonb/COAWST
   chmod 777 coawst.bash
   ./coawst.bash
   [ ...  ]
   rm -f -r /glade/u/home/johnsonb/make_macros.m

