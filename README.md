# How to build and Run Chromuim for Dev 
## System Requirement
``` bash
uname -m # check the architecture X86_64
free -h  # check memory and swap memory
# add a size of swap file 
    sudo fallocate -l 16GB /swapfile  # create  afile with 16GB
    sudo chmod /swapfile  # add the correct autorisation
    sudo mkswap /swapfile
    sudo swapon /swapfile
df -h  # verify a disk size
clang --version
git --version
python3 --version  # >=3.9
lscpu # check the number of cpu 
```
### 1. Instalation de depotools 
```bash
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git # clone the repot
 export PATH = "${HOME}/ProjectFolder/depot_tools:$PATH" # add to the path
gclient --version # test if the instalation is correct 
```
###  2.Code Recuperation
```bash
mkdir chromium && cd chromium
fetch --nohooks chromium 
```
