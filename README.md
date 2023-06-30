﻿# `MonteCarloMultiGPU` Sample

The `MonteCarloMultiGPU` sample evaluates fair call price for a given set of European options using the Monte Carlo approach. MonteCarlo simulation is one of the most important algorithms in quantitative finance. This sample uses a single CPU Thread to control multiple GPUs.

| Area                   | Description
|:---                    |:---
| What you will learn    | How to begin migrating CUDA to SYCL
| Time to complete       | 15 minutes
| Category               | Code Optimization

>**Note**: This sample is migrated from NVIDIA CUDA sample. See the [MonteCarloMultiGPU](https://github.com/NVIDIA/cuda-samples/tree/master/Samples/5_Domain_Specific/MonteCarloMultiGPU) sample in the NVIDIA/cuda-samples GitHub.


## Purpose
The MonteCarlo Method provides a way to compute expected values by generating random scenarios and then averaging them. The execution of these random scenarios can be parallelized very efficiently. 

With the help of a GPU, we reduce and speed up the problem by parallelizing each path. That is, we can assign each path to a single thread, simulating thousands of them in parallel, with massive savings in computational power and time.

> **Note**: The sample used the open-source SYCLomatic tool that assists developers in porting CUDA code to SYCL code. To finish the process, you must complete the rest of the coding manually and then tune to the desired level of performance for the target architecture. You can also use the Intel® DPC++ Compatibility Tool available to augment Base Toolkit.

This sample contains two versions in the following folders:

| Folder Name                          | Description
|:---                                  |:---
| 01_dpct_output                       | Contains output of SYCLomatic Tool used to migrate SYCL-compliant code from CUDA code. This SYCL code has some unmigrated code that has to be manually fixed to get full functionality.
| 02_sycl_migrated                     | Contains manually migrated SYCL code from CUDA code.

## Prerequisites

| Optimized for         | Description
|:---                   |:---
| OS                    | Ubuntu* 20.04
| Hardware              | Intel® Gen9 <br> Gen11 <br> Xeon CPU
| Software              | SYCLomatic <br> Intel® oneAPI Base Toolkit (Base Kit)

## Key Implementation Details

This sample demonstrates the migration of the following prominent CUDA feature: 
- Random Number Generator

Calls to cuRAND function APIs are being translated to equivalent Intel® oneAPI Math Kernel Library (oneMKL) function calls. 

The `MonteCarloOneBlockPerOption()` kernel is the main computation kernel. It calculates the integral over all paths using a single thread block per option. 

The `rngSetupStates()` kernel initializes the random number generator states for each thread. 

The `initMonteCarloGPU()` function allocates memory on the GPU, sets up the random number generator states, and initializes other variables. 

The `closeMonteCarloGPU()` function computes statistics and deallocates memory on the GPU. 

Finally, the `MonteCarloGPU()` function performs the main computations by copying data to the GPU, launching the computation kernel, and copying the results back to the host.

>**Note**: Refer to [Workflow for a CUDA* to SYCL* Migration](https://www.intel.com/content/www/us/en/developer/tools/oneapi/training/cuda-sycl-migration-workflow.html) for general information about the migration workflow.

### CUDA source code evaluation

Pricing of European Options can be done by applying the Black-Scholes formula and with MonteCarlo approach.

MonteCarlo Method first generates a random number based on a probability distribution. The random number then uses the additional inputs of volatility and time to expiration to generate a stock price. The generated stock price at the time of expiration is then used to calculate the value of the option. The model then calculates results over and over, each time using a different set of random values from the probability functions

The first stage of the computation is the generation of a normally distributed N(0, 1)number sequence, which comes down to uniformly distributed sequence generation. Once we’ve generated the desired number of samples, we use them to compute an expected value and confidence width for the underlying option. 

The Black-Scholes model relies on fixed inputs (current stock price, strike price, time until expiration, volatility, risk free rates, and dividend yield). The model is based on geometric Brownian motion with constant drift and volatility. We can calculate the price of the European put and call options explicitly using the Black–Scholes formula.

The price of a call option C in terms of the Black–Scholes parameters is

C=N(d1)×S−N(d2)×PV(K)

where:

•	d1=1σ√T[log(SK)+(r+σ22)T]

•	d2=d1−σ√T

•	PV(K)=Kexp(−rT)

After repeatedly computing appropriate averages, the estimated price of options can be obtained, which is consistent with the analytical results from Black-Scholes model.

For information on how to use SYCLomatic, refer to the materials at *[Migrate from CUDA* to C++ with SYCL*](https://www.intel.com/content/www/us/en/developer/tools/oneapi/training/migrate-from-cuda-to-cpp-with-sycl.html)*.

## Set Environment Variables

When working with the command-line interface (CLI), you should configure the oneAPI toolkits using environment variables. Set up your CLI environment by sourcing the `setvars` script every time you open a new terminal window. This practice ensures that your compiler, libraries, and tools are ready for development.

## Migrate the Code Using SYCLomatic

For this sample, the SYCLomatic tool automatically migrates 100% of the CUDA code to SYCL. Follow these steps to generate the SYCL code using the compatibility tool.

1. Clone the required GitHub repository to your local environment.
   ```
   git clone https://github.com/NVIDIA/cuda-samples.git
   ```
2. Change to the MonteCarloMultiGPU sample directory.
   ```
   cd cuda-samples/Samples/5_Domain_Specific/MonteCarloMultiGPU/
   ```
3. Generate a compilation database with intercept-build.
   ```
   intercept-build make
   ```
   This step creates a JSON file named compile_commands.json with all the compiler invocations and stores the names of the input files and the compiler options.

4. Pass the JSON file as input to the Intel® SYCLomatic Compatibility Tool. The result is written to a folder named dpct_output. The `--in-root` specifies path to the root of the source tree to be migrated. The `--use-custom-helper` option will make a copy of dpct header files/functions used in migrated code into the dpct_output folder as include folder.

   ```
   c2s -p compile_commands.json --in-root ../../.. --use-custom-helper=api
   ```

## Build the `MonteCarloMultiGPU` Sample for CPU and GPU

> **Note**: If you have not already done so, set up your CLI
> environment by sourcing  the `setvars` script in the root of your oneAPI installation.
>
> Linux*:
> - For system wide installations: `. /opt/intel/oneapi/setvars.sh`
> - For private installations: ` . ~/intel/oneapi/setvars.sh`
> - For non-POSIX shells, like csh, use the following command: `bash -c 'source <install-dir>/setvars.sh ; exec csh'`
>
> Windows*:
> - `C:\Program Files (x86)\Intel\oneAPI\setvars.bat`
> - Windows PowerShell*, use the following command: `cmd.exe "/K" '"C:\Program Files (x86)\Intel\oneAPI\setvars.bat" && powershell'`
>
> For more information on configuring environment variables, see *[Use the setvars Script with Linux* or macOS*](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos.html)*.

### On Linux*

1. Change to the sample directory.
2. Build the program.
   ```
   $ mkdir build
   $ cd build
   $ cmake ..
   $ make
   ```

   By default, this command sequence will build the `01_dpct_output`, `02_sycl_migrated` version of the program.

3. Run `02_sycl_migrated` for CPU and GPU.
     ```
    make run_sm_cpu
    make run_sm_gpu (runs on Level-Zero Backend)
    make run_sm_gpu_opencl (runs on OpenCL Backend)
    ```

#### Troubleshooting

If an error occurs, you can get more details by running `make` with
the `VERBOSE=1` argument:
```
make VERBOSE=1
```
If you receive an error message, troubleshoot the problem using the **Diagnostics Utility for Intel® oneAPI Toolkits**. The diagnostic utility provides configuration and system checks to help find missing dependencies, permissions errors, and other issues. See the [Diagnostics Utility for Intel® oneAPI Toolkits User Guide](https://www.intel.com/content/www/us/en/develop/documentation/diagnostic-utility-user-guide/top.html) for more information on using the utility.

### Example Output
The following example is for `02_sycl_migrated` for GPU on Intel(R) UHD Graphics P630 [0x3e96] with Level Zero Backend.
```
./a.out Starting...

MonteCarloMultiGPU
==================
Parallelization method  = streamed
Problem scaling         = weak
Number of GPUs          = 1
Total number of options = 12
Number of paths         = 262144
main(): generating input data...
main(): starting 1 host threads...
main(): GPU statistics, streamed
GPU Device #0: Intel(R) UHD Graphics P630 [0x3e96]
Options         : 12
Simulation paths: 262144

Total time (ms.): 3.139000
        Note: This is elapsed time for all to compute.
Options per sec.: 3822.873601
main(): comparing Monte Carlo and Black-Scholes results...
Shutting down...
Test Summary...
L1 norm        : 6.504269E-04
Average reserve: 2.790815

```
## License
Code samples are licensed under the MIT license. See
[License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third party program licenses are at [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt).
