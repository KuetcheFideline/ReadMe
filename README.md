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
source ~/.bashrc
```
###  2.Code Recuperation
```bash
mkdir chromium && cd chromium
fetch --nohooks chromium 
```
#### 2.1 Erreur possible 
```bash
 RESOURCE_EXHAUSTED: Resource has been exhausted (e.g. check quota)
[2:32:21] remote: [type.googleapis.com/google.rpc.QuotaFailure]
[2:32:21] remote: violations {
[2:32:21] remote:   subject: "ip/195.220.84.41"
[2:32:21] remote:   description: "Short term server-time rate limit exceeded"
[2:32:21] remote: }
[2:32:21] remote: 
[2:32:21] remote: [type.googleapis.com/google.rpc.RequestInfo]
[2:32:21] remote: request_id: "e5ce80a5772a481995eecb0b9d57fdd5"
[2:32:21] fatal: unable to access 'https://chromium.googlesource.com/chromiumos/third_party/libdrm.git/': The requested URL returned error: 429
[2:32:24] remote: RESOURCE_EXHAUSTED: Resource has been exhausted (e.g. check quota)
[2:32:24] remote: [type.googleapis.com/google.rpc.QuotaFailure]
[2:32:24] remote: violations {
[2:32:24] remote:   subject: "ip/195.220.84.41"
[2:32:24] remote:   description: "Short term server-time rate limit exceeded"
[2:32:24] remote: }
[2:32:24] remote: 
[2:32:24] remote: [type.googleapis.com/google.rpc.RequestInfo]
[2:32:24] remote: request_id: "4e0c73f85ce942e59884e42726743140"
[2:32:24] fatal: unable to access 'https://chromium.googlesource.com/chromiumos/third_party/libdrm.git/': The requested URL returned error: 429
```
Probleme Trop de requette envoye au serveur de de google chrome so your ip adress is blocked 
#### 2.2 Resolution 
```bash
    rm -rf _bad_scm  # supression du dossier qui conient les paquest casser
     gclient sync -j2 # on synchronise avec Gclient on ne lance plus le fetch tiret -j2 c'est pour dire que a la place de 12 process git on lance juste 2 pour eviter la requette 429 qu'on a eu la premiere fois  
```
### 3.Buil de Chromium 
```bash
./build/install-build-deps.sh
 
``
