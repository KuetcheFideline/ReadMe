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
