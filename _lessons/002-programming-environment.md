---
layout: post
title:  HPC3 Programming Environment
---

Please use `git clone` to download this repository, if you want to try out compiling codes yourself:

```
git clone https://github.com/nesi/hpc_training.git
```

Then go to the `code` directory:

```
cd hpc_training/code
```

There are further subdirectories with the different examples that will be shown in this session.

### Targeting a CPU

A CPU target should be loaded before building any code. This means that you tell the compiler to use instructions that are understood by this particular CPU (or a younger CPU in the same family, such as Intel's x86 CPUs). Otherwise you will see a warning message from the Cray compiler driver whenever you try to compile code,

```
No supported cpu target is set, CRAY_CPU_TARGET=x86-64 will be used.
Load a valid targeting module or set CRAY_CPU_TARGET
```

Although the compilers will still work, the resulting code is generic code for x86-64 processors and not optimised for the Skylake processor. So be sure to load
```
module load craype-x86-skylake
```
The resulting executable will not run on older processors, such as the Broadwell family which is used for the upcoming HPC1. Codes will either crash with an "illegal instruction" error, or display an error message if built with an Intel compiler.

### Compiler Wrappers

The Cray environment provides compiler wrappers, which should be used to compile Fortran, C, and C++ code:

```
ftn -o simpleMpiF90 simpleMpi.f90 # compile Fortran code
cc -o simpleMpiC simpleMpi.c      # compile C code
CC -o simpleMpiCxx simpleMpi.cxx  # compile C++ code
```

Note that these wrappers are always the same, regardless of your choice of compiler (Cray, Intel, or GNU). The wrappers will ensure correct linking of your code with compiler runtime libraries, and with Cray-supported libraries (such as Cray's `libsci` scientific library, or netCDF). We will get to that topic later in this session.

There is also no need to invoke special compilers for MPI code or to apply special compiler options for code with OpenMP directives - use ftn, cc, and CC in all cases. 

### Programming environments

The default programming environment provides Cray's CCE (Cray Compiler Environment) for Fortran, C, and C++. However, other compilers can also be selected using

```
module swap PrgEnv-cray PrgEnv-intel
```
to use the Intel compilers, or
```
module swap PrgEnv-cray PrgEnv-gnu
```
to use the GNU compilers.

You should always invoke the ```ftn```, ```cc``` and ```CC``` compiler wrappers to ensure correct linking, regardless of the programming environment.

Note that swapping programming environments will automatically swap Cray-provided libraries - but this will **not** be the case for libraries provided by NeSI or NIWA.

### Common compiler options

Although the compiler drivers have a few options of their own, they will pass through any compiler options you set - this means that you simply set the exact same options that you would have used with the underlying compiler. For example, if you are using the `gfortran` compiler and wanted to activate compiler warnings and compiler optimisation, you would use the following command:

```
module swap PrgEnv-cray PrgEnv-gnu
ftn -Wall -O2 -o simpleMpiF90 simpleMpi.f90
```

To see the compiler options that are specific to a compiler, type
```
man crayftn
man icc
man g++
```
for instance - don't forget to to load the corresponding programming environment first.

The wrappers provide their own options, and a few options that are common accross programming environments. You can look at them using
```
man ftn
man cc
man CC
```


### Linking against an external library

Three different cases need to be considered for linking your code against an external library:

1. The library is provided by Cray

In this case, it is sufficient to simply load the library module before building your code,
```
module load cray-netcdf
ftn -o simple_xy_wr simple_xy_wr.f90
```
for instance. The compiler drivers ensure that include files and libraries are automatically found, no paths or library flags need to be specified. Moreover, correct libraries will automatically be used when a different compiler is selected (e.g., Intel or GNU).

Note that you may occasionally see a warning message of the kind:
```
/opt/cray/pe/hdf5/1.10.1.1/INTEL/16.0/lib/libhdf5.a(H5PL.o): In function `H5PL_load':
H5PL.c:(.text+0x612): warning: Using 'dlopen' in statically linked applications requires at runtime the shared libraries from the glibc version used for linking
```
This simply means that the library must be accessible at runtime despite fully static linking; this can usually be ignored.

Let's have a look at how the compiler driver does this:
```
ftn -o simple_xy_wr simple_xy_wr.f90 -craype-verbose
```
This will produce lots of output, and we will find that the driver added a lot of compiler flags for us.

Let

Run
```
module avail
```
and look for modules that start with "cray-" to get a full list of Cray-provided libraries. Use Cray's libsci or Intel's MKL if you need BLAS, LAPACK, or FFT.

2. The library is provided via NeSI or NIWA

NeSI and NIWA use EasyBuild to provide additional, commonly used libraries. In many cases, loading the library module before building your code will suffice to make it visible to the compiler driver:
```
module swap PrgEnv-cray PrgEnv-gnu
module load GSL/2.4-CrayGNU-2017.06
cc -o gsl_statistics_example gsl_statistics_example.c -lgsl
```
Note that we had to add `-lgsl` to avoid a linker error, but we did not set an additional include path using `-I/some/path/to/gsl/include`, nor a library search path using `-L/some/path/to/gsl/lib`. The GSL module sets the ```CPATH```, ```LIBRARY_PATH```, and ```LD_LIBRARY_PATH``` variables for us, which are picked up by the compiler driver:
```
echo $CPATH
echo $LIBRARY_PATH
echo $LD_LIBRARY_PATH
```
This automatic setup may not always work, in which case you will need to help things along a little bit by specifying these paths manually using EasyBuild's automatically defined module variables. These can be found by running the `module show` command:
```
module show GSL/2.4-CrayGNU-2017.06
cc -I$EBROOTGSL/include -o gsl_statistics_example gsl_statistics_example.c -L$EBROOTGSL/lib -lgsl
```
EasyBuild's convention for the variable names is `EBROOT<package name>`.

**Note:** Make sure that you choose the correct variant of a library, depending on your choice of compiler. This does *not* happen automatically for libraries that are provided by NeSI and NIWA, and you will also need to swap between different library variants if you swap compilers.

EasyBuild uses the following module naming conventions ("toolchain names"):
* "CrayCCE" for libraries and tools built with the Cray compilers ("PrgEnv-cray")
* "CrayIntel" for libraries and tools built with the Intel compilers ("PrgEnv-intel")
* "CrayGNU" for libraries and tools built with the GNU compilers ("PrgEnv-gnu")

3. You need to link against your own library

In this case, you will need to specify the include or Fortran module files with the ```-I``` option. Use ```-L```
to specify the location of the libraries and ```-l``` the name of the libraries. Libraries typically have the prefix ```lib``` and either ```.so``` or ```.a``` as suffix.  Do not include the lib prefix or the suffix when passing a library name to the ```-l``` option. For instance, library "A" might be called ```libA.a``` and to link against "A" you would use ```-lA```. 

You can have multiple ```-L``` and ```-l``` options. When specifying libraries with ```-l```, the order matters. Symbols are resolved from left to right; that is if library "A" depends on "B" then "A" should precede "B" (```-lA -lB```). Library "A" depends on "B" if "A" calls functions or invokes symbols that are defined in "B".

### Static and dynamic linking

The compilers drivers default to static linking where possible, and the resulting executables will be completely self-contained if all dependencies are available as static libraries. Let's have a look at the previous example:
```
cc -o gsl_statistics_example gsl_statistics_example.c -lgsl
ldd gsl_statistics_example
```
This should result in the message `not a dynamic executable`. Static linking is recommended on XC systems for performance reasons. To force dynamic linking, run the following command:
```
cc -o gsl_statistics_example gsl_statistics_example.c -lgsl -dynamic
ldd gsl_statistics_example
```
You should now see a number of libraries that are dynamically linked. Note that not all libraries are available as both static and dyamic libraries - in this case, the linker will simple use what it can get.