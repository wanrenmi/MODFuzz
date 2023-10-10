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
### 2.1 Set parameters for static analysis
Set the path of the compiled vmlinux in important_global_data.py, and adjust each function set according to needs.
### 2.2 Run the cfg_construct_store.py script
This step will save the experimental results (CFG and symbol table) in pickle form.
### 2.3 Run the vulnerability_score.py script
Get the vulnerability score set. The script results will also be saved in pickle form. Although the use of pickle has the risk of deserialization vulnerabilities, static analysis takes a long time, and saving intermediate results is beneficial to debugging and customization.
### 2.4 Run function write_result_to_afl() in gadgets.py
Generate the final result of static analysis addr_score_afl.txt
## 3. Multi-objective directed fuzzing
Since MODFuzz uses the framework of USBFuzz, the environment preparation is similar.
### 3.1 Preparing a Linux userspace image
scripts/create-image.sh -f full
### 3.2 Build the fuzzer and qemu
sudo apt-get update
sudo apt-get build-dep qemu
./build.sh
After the guest system is up, copy OSes/linux-target/user-mode-agent to the guest system and run install.sh in the guest system.
If you encounter environment or configuration problems (some versions of gcc may be incompatible) and cause compilation to fail, you can use the compiled compressed file MODFuzz_compiled.zip.
### 3.3 Begin fuzzing
./USBFuzz --seeddir seeds --kernel_image PATH_TO_YOUR_BZIMAGE --os_image PATH_TO_YOUR_USERSPACE_IMAGE
### 3.4 Reproduce or debug a crash
./usbfuzz-afl/qemu_mode/qemu-build/x86_64-softmmu/qemu-system-x86_64 -M q35 -device qemu-xhci,id=xhci  -object memory-backend-shm,id=shm -device ivshmem-plain,id=ivshmem,memdev=shm  -m 4G -enable-kvm -kernel  PATH_TO_YOUR_BZIMAGE -hda PATH_TO_YOUR_USERSPACE_IMAGE -append 'root=/dev/sda console=ttyS0; nokaslr' -usbDescFile PATH_TO_YOUR_CRASH -serial stdio (if you want to debug a crash, add "-S -s" at the end of the command, and use gdb to connect to port 1234 of qemu process)

