# How to Build and Run Chromium for Dev
If you want more information check  
[Documentation Chromium](https://chromium.googlesource.com/chromium/src/+/main/docs/linux/build_instructions.md#run-chromium)


## System Requirements

```bash
uname -m        # Check the architecture (x86_64)
free -h         # Check memory and swap memory

# Add swap space
sudo fallocate -l 16GB /swapfile   # Create a file of 16GB
sudo chmod 600 /swapfile           # Set correct permissions
sudo mkswap /swapfile
sudo swapon /swapfile

df -h           # Verify disk size
clang --version
git --version
python3 --version   # >= 3.9
lscpu           # Check the number of CPUs
```

---

## 1. Installation of depot_tools

```bash
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="${HOME}/ProjectFolder/depot_tools:$PATH"
gclient --version
source ~/.bashrc
```

---

## 2. Code Retrieval

```bash
mkdir chromium && cd chromium
fetch --nohooks chromium
```

### 2.1 Possible Error

```
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
```

**Problem:** Too many requests sent to Google's servers — your IP address has been rate-limited (HTTP 429).

### 2.2 Resolution

```bash
rm -rf _bad_scm
gclient sync -j2
```

---

## 3. Build Dependencies

```bash
./build/install-build-deps.sh
```

---

## 4. Run the Hooks

```bash
gclient runhooks
```

---

## 5. Speed Up Git Operations (Optional but Recommended)

```bash
which watchman
cd ~/chromium/src

cp .git/hooks/fsmonitor-watchman.sample ~/bin/query-watchman

git config core.untrackedCache true
git config core.fsmonitor $HOME/bin/query-watchman

echo '{ "ignore_dirs": ["out"] }' > .watchmanconfig

sudo vim /etc/sysctl.d/99-inotify.conf
# Add the following lines:
# fs.inotify.max_user_instances = 8192
# fs.inotify.max_user_watches = 10485760

sudo sysctl --system

# Add to ~/.bashrc or ~/.zshrc:
alias gaa="git status --porcelain | awk '{print $2}' | xargs -r git add"
```

---

## 6. Configure the Build

```bash
gn gen out/Default
```

---

## 7. Compile Chromium

```bash
autoninja -C out/Default chrome
```

---

## 8. Run Chromium

```bash
out/Default/chrome
```
