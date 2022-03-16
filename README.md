# Fortan-Reducer

## About

Fortran-Reducer is a tool for extracting reproducers from Fortran codes that takes a Fortran file that has a property of interest and automatically builds a hierarchy of used modules in that file and generates a compile line for all files.

*NOTE:* This tool can be used on both *Linux* and *Windows*.

## Usage tool

**Build and Run:**  

Cloning a repository:  

                git clone https://github.com/IlyaShein/Fortran-reducer.git
                cd Fortran-reducer

To run, you need to enter the following command in the command line: python3.x Script.py `<name_file>` –source-dir `<path_to_module>` -i `<additional_path_to_module>` -x `<excluded_path>` –o `<name_output_file>` -out-dir `<path_output>` -print-module-only-deps=on -intrinsics:extended -verbose -check_include_file

The file name can be specified both through the full path to this file, and through name.f90 if it is located in the same directory as Script.py

**Basic options:**

- -source-dir:
  
>The `-source-dir` allows to specify the path to the source code search directory and another codes, which contains needed modules.  
>*NOTE: If source files are located in the same directory as Script.py, then this option can be omitted.*

- -i:
  
>The `-i` is an addition to -source-dir and allows to add the additional search path.

- -x:
  
>The `-x` removes the specified path from consideration for finding modules.

- -log:off:

>The `log:off` allows you not to create a file log.txt with the output of the entire hierarchy.

- -o:
  
>The `-o` allows to create the merged output source file with containing all dependences. On the output we have Fortran file which contains all dependences modules arranged in the order of nesting.

- -out-dir:
  
>The `-out-dir` specifies the path where a folder Result for output files will be created.

- -print-module-only-deps=on:
  
>The `-print-module-only-deps=on` allows to print the used internal entities/objects  from “USE statement” only-list.

- -intrinsics:
  
>The `-intrinsics` has three behavior modes (default, off and extended). This options allow to ignore missing intrinsics modules such as iso_c_binding, mpi, omp_lib and others. In the default mode the modules are found as a result of the script execution and compared with the default list of system modules. In the extended mode the behavior is similar, but the comparison occurs with a larger list. And in the third mode the comparison doesn’t happen.

- -verbose:
  
>The `-verbose` enables additional debug output( it’s a file which is currently being viewed, additional execution steps).

- -check_include_file:
  
>The `-check_include_file` allows you to search for #include preprocessor directives in addition to "use statement" and display them in the final compilation line.

*NOTE: The path containing spaces must be enclosed in quotation marks.*  
*If you use -check_include_file option, so please add -Iinclide(Linux) or /Iinclude(Windows) to compilation line*
<br/><br/>

Example start for Windows: `python Script.py hp_calc_chi.f90 -source-dir C:\sr\q-e-qe-6.7.0 -out-dir C:\doc\dtest\dtest1 -i C:\doc\dfil -i C:\doc\project -verbose -check_include_file -log:off -o repro.f90 -x C:\doc\project\test`

Example start for Linux: `python Script.py hp_calc_chi.f90 -source-dir /src/dtest -out-dir /src/dtest -i /src/doc/dfil -i /src/doc/project -verbose -check_include_file -log:off -o repro.f90 -x /src/doc/project/test`  

As a result of execution, will be received 2 text files(`log.txt` and `summary.txt`) and 1 Fortran file `repro.f90`. The first contains the hierarchy of modules, and the second contains the modules that were not found and the compilation line.

## Prerequisites

- [Git](https://git-scm.com/)
- [Python](https://www.python.org/) (version 3.x)

## Usage example

Let's consider an example of executing the tool on the [ELPA](https://gitlab.mpcdf.mpg.de/elpa/elpa) open source project.

There is a Fortran file which includes the code below:

        #include "config-f90.h"

        module elpa1_impl
                use, intrinsic :: iso_c_binding
                use elpa_utilities
                use elpa1_auxiliary_impl
        #ifdef HAVE_LIKWID
                use likwid
        #endif

                implicit none
        ...

Run the tool on this file: `python Script.py elpa1.F90 -source-dir /home/elpa/src -check_include_file -o repro.f90`

As a result of execution, the **Result** folder will be created, in which all files containing the necessary modules will be placed. In addition, a separate **include** folder will be created in Result, where all "#include" files will be placed.  

The files with the module hierarchy (`log.txt`) and with the compilation line (`summary.txt`) and the `repro.f90` file will be located in the Result folder.

The file **log.txt**:

        module elpa_utilities:<path>
        --------module iso_fortran_env:intrinsic module
        --------module elpa_gpu:<path>
        ----------------module precision:<path>
        ----------------module iso_c_binding:intrinsic module
        ----------------module mkl_offload:<path>
        ------------------------module iso_c_binding:intrinsic module
        ----------------module cuda_functions:<path>
        ------------------------module precision:<path>
        ----------------module hip_functions:<path>
        ------------------------module precision:<path>
        ...

The file **summary.txt**:

At the beginning of the entry, the paths of all modules that are used.  
The rest of the findings follow:  

- Modules without use statement:  

        mod_precision.F90 elpa_constants.F90 elpa_generated_fortran_interfaces.F90 ftimings_type.F90 timer_dummy.F90 mod_scalapack_interfaces.F90

- Compilation line:  
  
       mod_precision.F90 mod_hip.F90 mod_cuda.F90 mod_time_c.F90 mod_mkl_offload.F90 elpa_constants.F90 ftimings_type.F90 mod_vendor_agnostic_layer.F90 mod_mpi_stubs.F90 ftimings_value.F90 elpa_utilities.F90 timer_dummy.F90 matrix_plot.F90 <!module PRECISION_MODULE!> ftimings.F90 mod_blas_interfaces.F90 mod_mpi.F90 elpa_generated_fortran_interfaces.F90 mod_omp.F90 elpa_api.F90 elpa_abstract_impl.F90 mod_v_add_s.F90 mod_solve_secular_equation.F90 mod_add_tmp.F90 mod_check_monotony.F90 mod_transform_columns.F90 mod_resort_ev.F90 mod_distribute_global_column.F90 mod_global_gather.F90 mod_global_product.F90 mod_cholesky_cuda.F90 mod_invert_trm_cuda.F90 mod_merge_systems.F90 mod_merge_recursive.F90 check_for_gpu.F90 mod_solve_tridi.F90 elpa1_compute_private.F90 mod_thread_affinity.F90 mod_scalapack_interfaces.F90 <!module likwid!> mod_elpa_invert_trm.F90 mod_elpa_cholesky.F90 elpa1_auxiliary.F90 elpa1.F90 

- Modules and includes not found:

        Missing modules:
        1 - module PRECISION_MODULE
        2 - module likwid


        Missing include:
        1 - config-f90.h
        2 - elpa_generated_public_fortran_interfaces.h
        3 - fortran_constants.F90
        4 - elpa_generated_fortran_interfaces.h
        5 - module PRECISION_MODULE
