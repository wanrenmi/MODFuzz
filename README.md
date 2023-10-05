# A multi-objective directed fuzzer for USB kernel drivers
MODFuzz is a multi-objective directed fuzzer for USB kernel drivers. It is mainly divided into three parts: kernel source code preprocessing, static vulnerability analysis, and multi-objective directed fuzzing.
## 1. Kernel source code preprocessing
### 1.1 Download the kernel source code of the target version. 
Next we take v6.4.10 as an example.
### 1.2 Patch the kernel source code by patch files in OSes/linux-target/kernel-patches/
There are four files in this directory, of which 1.patch and 2.patch are required and are used to add the IVSHMEM driver and customize the kcov instrumentation solution respectively. 3-limit_kcov_USB.py is used to customize the range of instrumentation, and the range selected by 3.patch is the same as USBFuzz. For simplicity, you can use 1.patch, 2.patch and 3.patch directly.
### 1.3 Compile the kernel
Use OSes/linux-target/.config_6.4.10 as the kernel compilation configuration file, and then run the "make" command to compile the kernel. If you have customized compilation requirements, you can change the configuration options according to the version and requirements, but be sure to keep "CONFIG_KCOV=y".
## 2. Static vulnerability analysis
