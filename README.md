# How to Build and Run Chromium for Dev
If you want more information check  
[Documentation Chromium](https://chromium.googlesource.com/chromium/src/+/main/docs/linux/build_instructions.md#run-chromium)

## System Requirements

```bash
uname -m        # Check the architecture — Chromium only supports x86_64 and arm64 on Linux
free -h         # Chromium requires at least 8GB RAM; 16GB+ recommended for linking

# Add swap space — the linker (lld) can use 20GB+ of memory during compilation
sudo fallocate -l 16GB /swapfile   # Allocate 16GB of swap to avoid OOM kills during the link step
sudo chmod 600 /swapfile           # Restrict access to root only (required by mkswap)
sudo mkswap /swapfile              # Format the file as swap space
sudo swapon /swapfile              # Activate the swap immediately without rebooting

df -h           # Verify you have at least 100GB free — a full Chromium build takes ~60–80GB
clang --version # Chromium uses Clang exclusively — GCC is not supported by the build system
git --version   # depot_tools requires a recent version of git (>= 2.x)
python3 --version   # >= 3.9 required by various build and helper scripts
lscpu           # More cores = faster build; autoninja scales jobs to CPU count automatically
```

---

## 1. Installation of depot_tools

depot_tools is Google's set of tools to manage and build Chromium. It includes `gclient`, `gn`, `autoninja`, and more.

```bash
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git  # Clone the official depot_tools repo from Google
export PATH="${HOME}/ProjectFolder/depot_tools:$PATH"   # Prepend depot_tools to PATH so its commands take priority over system ones
gclient --version   # Verify gclient is found and depot_tools is correctly set up
source ~/.bashrc    # Reload the shell config to persist the PATH change in the current session
```

---

## 2. Code Retrieval

```bash
mkdir chromium && cd chromium   # Create a dedicated folder — the source tree is very large (~30GB)
fetch --nohooks chromium        # Download the Chromium source without running hooks yet (faster initial clone)
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

**Problem:** `fetch` spawns up to 12 parallel git processes by default. Google's servers rate-limit IPs that send too many simultaneous requests, returning HTTP 429.

### 2.2 Resolution

```bash
rm -rf _bad_scm     # Delete the folder containing partially downloaded/corrupted packages left by the failed fetch
gclient sync -j2    # Re-sync using only 2 parallel git jobs (-j2) instead of the default 12 — avoids hitting the 429 rate limit again
```

---

## 3. Build Dependencies

```bash
./build/install-build-deps.sh   # Install all required system packages (libgtk, libdbus, libnss, etc.) needed to compile and run Chromium
```

---

## 4. Run the Hooks

```bash
gclient runhooks   # Execute post-sync hooks: downloads the Clang toolchain, sysroot, and generates some auto-generated source files
```

---

## 5. Speed Up Git Operations (Optional but Recommended)

The Chromium repo has 300,000+ files. Standard git commands become very slow without these optimizations.

```bash
which watchman                  # Check if watchman is installed — it monitors file changes in real time so git doesn't have to scan the full tree
cd ~/chromium/src               # Move into the source directory

cp .git/hooks/fsmonitor-watchman.sample ~/bin/query-watchman   # Copy the watchman query script that git will call to detect changed files

git config core.untrackedCache true                        # Cache the list of untracked files — speeds up `git status` significantly
git config core.fsmonitor $HOME/bin/query-watchman         # Tell git to use watchman instead of scanning the filesystem manually

echo '{ "ignore_dirs": ["out"] }' > .watchmanconfig        # Exclude the build output directory from watchman — it changes constantly and would cause unnecessary overhead

sudo vim /etc/sysctl.d/99-inotify.conf   # Open sysctl config to increase inotify limits — required if you hit "inotify_init: user limit reached"
# Add the following lines:
# fs.inotify.max_user_instances = 8192    # Max number of inotify watchers per user (default is often 128)
# fs.inotify.max_user_watches = 10485760  # Max number of files that can be watched simultaneously

sudo sysctl --system   # Apply the sysctl changes immediately without rebooting

# Add to ~/.bashrc or ~/.zshrc:
alias gaa="git status --porcelain | awk '{print $2}' | xargs -r git add"   # Faster alternative to `git add -A` — avoids the slow index scan on large repos
```

---

## 6. Configure the Build

```bash
gn gen out/Default   # Generate the Ninja build files in out/Default — you can also pass flags here (e.g. is_debug=false, is_component_build=true)
```

---

## 7. Compile Chromium

```bash
autoninja -C out/Default chrome   # Build the `chrome` target — autoninja auto-tunes the number of parallel jobs based on your CPU and available RAM
```

---

## 8. Run Chromium

```bash
out/Default/chrome   # Launch the freshly compiled Chromium binary directly from the build output directory
```

if you have a problem try with 

```bash
out/Default/chrome --no-sandbox
```
