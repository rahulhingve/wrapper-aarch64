# wrapper

A tool to decrypt Apple Music's music. An active subscription is still needed.

Only supports Linux x86_64 and arm64.

---

## ARM64 Linux — Quick Start (Pre-built Image)

If you're on an `aarch64` server, the easiest way is to pull the pre-built Docker image — no building required:

```bash
docker pull rahulhingve1/wrapper
```

Skip to the [Docker (ARM64)](#docker-arm64) section below.

---

## Install

Get the pre-built version from this project's [Actions](../../actions) or [Releases](../../releases).

Or refer to the Actions configuration file for manual compilation.

> ⚠️ **ARM64 users:** Always clone or download from the `arm64` branch. The release zip may be missing `entrypoint.sh` due to a CI bug — use `git clone -b arm64` to be safe.

---

## Docker (x86_64)

**Build:**

```bash
docker build --tag wrapper .
```

**Login:**

```bash
docker run -it -v ./rootfs/data:/app/rootfs/data -p 10020:10020 \
  -e args="-L username:password -F -H 0.0.0.0" wrapper
```

**Run:**

```bash
docker run -d -v ./rootfs/data:/app/rootfs/data \
  -p 10020:10020 -p 20020:20020 -p 30020:30020 \
  -e args="-H 0.0.0.0" wrapper
```

---

## Docker (ARM64)

### Option A — Pull Pre-built Image (Recommended)

No QEMU, no build tools needed:

```bash
docker pull rahulhingve1/wrapper
```

### Option B — Build from Source on ARM64

**Prerequisites:**

```bash
# Install QEMU (needed only for the amd64 build stage)
sudo apt-get install -y qemu-user-static binfmt-support
docker run --privileged --rm tonistiigi/binfmt --install amd64

# Clone the arm64 branch (do NOT use release zip — entrypoint.sh may be missing)
git clone https://github.com/WorldObservationLog/wrapper -b arm64
cd wrapper

# Fix Debian base image (debian:13 has GPG signature bug inside Docker)
sed -i 's|debian:13.2|debian:bookworm|g' Dockerfile

# Build
docker build --no-cache --tag wrapper .
```

### Login

The `arm64` branch uses separate environment variables instead of the `-L` flag:

```bash
docker run -it --rm \
  -v ./rootfs/data:/app/rootfs/data \
  -p 10020:10020 \
  -e USERNAME="your.apple.id@email.com" \
  -e PASSWORD="YourApplePassword" \
  -e args="-F -H 0.0.0.0" \
  rahulhingve1/wrapper:arm64
```

#### 2FA — If Two-Factor Authentication is Enabled

When the container pauses for OTP, open a **second terminal in the same directory** and run:

```bash
echo -n yourOTPcode > rootfs/data/data/com.apple.android.music/files/2fa.txt
```

Example:

```bash
echo -n 987811 > rootfs/data/data/com.apple.android.music/files/2fa.txt
```

The container reads the file automatically.

### Run

After login, the token is cached in `./rootfs/data`. Run without credentials:

```bash
docker run -d \
  --name wrapper \
  --restart unless-stopped \
  -v ./rootfs/data:/app/rootfs/data \
  -p 10020:10020 -p 20020:20020 -p 30020:30020 \
  -e args="-H 0.0.0.0" \
  rahulhingve1/wrapper:arm64
```

---

## ARM64 Troubleshooting

| Error                                 | Cause                                  | Fix                                                                     |
| ------------------------------------- | -------------------------------------- | ----------------------------------------------------------------------- |
| `exec format error`                 | Running amd64 image on ARM64           | Use `rahulhingve1/wrapper` or build with `tonistiigi/binfmt`  |
| `/entrypoint.sh: not found`         | Broken release zip                     | Use `git clone -b arm64` instead of downloading zip                   |
| `No good signature` during build    | Debian 13 GPG bug                      | Run `sed -i 's\|debian:13.2\|debian:bookworm\|g' Dockerfile`             |
| `USERNAME and PASSWORD must be set` | Old `-L email:pass` format           | Use `-e USERNAME=` and `-e PASSWORD=` env vars separately           |
| `input device is not a TTY`         | Using `-it` with `-d` or `nohup` | Remove `-it` for background runs                                      |
| 2FA login stuck                       | OTP not provided                       | Write OTP to `rootfs/data/data/com.apple.android.music/files/2fa.txt` |

---

## Usage

```
Usage: wrapper [OPTION]...

  -h, --help              Print help and exit
  -V, --version           Print version and exit
  -H, --host=STRING         (default=`127.0.0.1')
  -D, --decrypt-port=INT    (default=`10020')
  -M, --m3u8-port=INT       (default=`20020')
  -P, --proxy=STRING        (default=`')
  -L, --login=STRING        (username:password)  [x86_64 only]
  -F, --code-from-file      (default=off)
```

---

## Special Thanks

- Anonymous, for providing the original version of this project and the legacy Frida decryption method.
- chocomint, for providing support for arm64 arch.
