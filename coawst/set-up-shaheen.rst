##############################
Set up COAWST model on Shaheen
##############################

This page contains brief notes about compiling COAWST on a Cray XC40.

.. important::

   On Shaheen II, compilers cannot write to project space. Compilation must
   occur in scratch space.

Download the COAWST source code
===============================

First attempt with Intel programming environment
------------------------------------------------

This doesn't work.

.. code-block::

   cd /lustre/scratch/${USER}
   git clone https://github.com/jcwarner-usgs/COAWST.git
   cd COAWST/Lib/MCT/
   jinter
   module load PrgEnv-intel
   module load cray-mpich
   ./configure

.. error::

   checking for cc... cc
   checking for C compiler default output file name... b.out
   checking whether the C compiler works... configure: error: cannot run C compiled programs.
   If you meant to cross compile, use `--host'.

Even when trying to compile and execute a simple ``hello_world`` program:

.. code-block::

   #include <stdio.h>
   int main() {
      // printf() displays the string inside quotation
      printf("Hello, World!");
      return 0;
   }

Compiling on a login node doesn't work as expected:

.. code-block::

   cc hello.c -xAVX -xCORE-AVX2
   ./a.out
   Please verify that both the operating system and the processor support
   Intel(R) MOVBE, FMA, BMI, LZCN0T and AVX2 instructions.

Compiling on a compute node doesn't work as expected:

.. code-block::

   jinter
   cc hello.c -xAVX -xCORE-AVX2
   Child failed to exec: No such file or directory

Second attempt with GNU programming environment
-----------------------------------------------

Compiling and executing a simple ``hello_world`` program.

.. code-block::

   module purge
   module load PrgEnv-gnu
   cc hello.c
   No supported cpu target is set, CRAY_CPU_TARGET=x87-64 will be used.
   module load craype-haswell
   cc hello.c
   ./a.out
   Hello, World!

.. note::

   The ``craype-haswell`` module sets ``CRAY_CPU_TARGET``:

   .. code-block::

      echo $CRAY_CPU_TARGET
      haswell

Attempting to use ``./configure``:

.. code-block::

   module swap PrgEnv-intel PrgEnv-gnu
   cd /lustre/scratch/x_johnsobk/COAWST/Lib/MCT
   ./configure
   [ ... ]
   configure: creating ./config.status
   config.status: creating Makefile.conf
   Please check the Makefile.conf
   Have a nice day!
   make
   make[1]: Entering directory '/lustre/scratch/x_johnsobk/COAWST/Lib/MCT/mpeu'
   cc -c -DFORTRAN_UNDERSCORE_ -DSYSLINUX -DCPRCRAY -O  get_zeits.c
   ftn -c  -DSYSLINUX -DCPRCRAY -O2 -xCORE-AVX2 -c -em -dy -rm -Omsgs,negmsgs  m_mpif.F90
   gfortran: error: unrecognized command-line option '-rm'; did you mean '-r'?
   make[1]: *** [Makefile:66: m_mpif.o] Error 1
   make[1]: Leaving directory '/lustre/scratch/x_johnsobk/COAWST/Lib/MCT/mpeu'
   make: *** [Makefile:10: subdirs] Error 2

Edit the ``Makefile.conf`` file:

.. code-block::

   vim Makefile.conf

   Default 13 F90FLAGS        = -c -em -dy -rm -Omsgs,negmsgs
   Edited  13 F90FLAGS        = -c -em -dy -r -Omsgs,negmsgs

.. error::

   make
   make[1]: Entering directory '/lustre/scratch/x_johnsobk/COAWST/Lib/MCT/mpeu'
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_mpif.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_realkinds.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_stdio.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_mpif90.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_dropdead.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_chars.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_flow.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_ioutil.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_mpout.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_die.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_IndexBin_char.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_IndexBin_integer.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_IndexBin_logical.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_mall.F90
   ftn -c  -DSYSLINUX -DCPRCRAY  -c -em -r  m_String.F90
   m_String.F90:478:17:
     457 |   call MPI_bcast(ln,1,MP_INTEGER,root,comm,ier)
         |                 2
   ......
     478 |   call MPI_bcast(Str%c(1),ln,MP_CHARACTER,root,comm,ier)
         |                 1
   Error: Type mismatch between actual argument at (1) and actual argument at (2) (CHARACTER(1)/INTEGER(4)).
   make[1]: *** [Makefile:66: m_String.o] Error 1
   make[1]: Leaving directory '/lustre/scratch/x_johnsobk/COAWST/Lib/MCT/mpeu'
   make: *** [Makefile:10: subdirs] Error 2

Third attempt with Cray programming environment
-----------------------------------------------

Attempting to compile MCT
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block::

   module swap PrgEnv-gnu PrgEnv-cray
   cd /lustre/scratch/x_johnsobk/COAWST/Lib/MCT
   ./configure
   [ ... ]
   configure: creating ./config.status
   config.status: creating Makefile.conf
   Please check the Makefile.conf
   Have a nice day!
   make
   [ ... ]
   make[1]: Leaving directory '/lustre/scratch/x_johnsobk/COAWST/Lib/MCT/mct'
   cp mpeu/libmpeu.a mct

Then edit the ``Linux-ftn-cray.mk`` file:

.. code-block::

   169        MCT_INCDIR ?= /lustre/scratch/x_johnsobk/COAWST/Lib/MCT/mct
   170        MCT_LIBDIR ?= /lustre/scratch/x_johnsobk/COAWST/Lib/MCT/mct

Building the COAWST executable
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block::

   cd /lustre/scratch/x_johnsobk/COAWST
   cp coawst.bash Projects/Inlet_test
   cd Projects/Inlet_test
   vim coawst.bash
   [ ... ]
   132 export   MY_ROOT_DIR=/lustre/scratch/x_johnsobk/COAWST
   [ ...  ]
   :wq
   jinter
   chmod 777 coawst.bash
   ./coaswt.bash
   make: nf-config: Command not found

   .. error::

   cp -f /include/netcdf.mod ./Build
   cp: cannot stat '/include/netcdf.mod': No such file or directory
   make: *** No rule to make target 'Build/netcdf.mod', needed by 'Build/MakeDepend'.  Stop.

