# SnapRAID for OpenWrt

Prebuilt **[SnapRAID](https://www.snapraid.it/) 14.9** OpenWrt packages,
cross-compiled for **every** OpenWrt architecture (release 25.12.5) and
published automatically as an **apk package repository**.

- ⚙️ Built with the official OpenWrt SDK (musl libc).
- 🗂️ apk format (OpenWrt **25.1+**); one directory per architecture.
- 🤖 Auto-built & re-published on every new upstream SnapRAID release (daily).
- 📄 Includes the upstream English man pages (`man snapraid`).

---

## Add the repo & trust key (OpenWrt 25.1+, apk)

The repository is served from the `gh-pages` branch:
`https://aseracorp.github.io/snapraid-openwrt/<arch>/packages.adb`

`<arch>` is your device's OpenWrt package architecture (e.g. `aarch64_cortex-a72`,
`x86_64`, `mipsel_24kc`). Find it with `apk arch` or from the fw-selector.

**One-time setup** (replace `<arch>` with the real value):

```sh
# 1. Install the repository verification public key (required for trust)
mkdir -p /etc/apk/keys
wget -O /etc/apk/keys/dc721b66b4af2424.pub \
  https://raw.githubusercontent.com/aseracorp/snapraid-openwrt/main/keys/dc721b66b4af2424.pub

# 2. Add the repository (URL must end in /packages.adb and match YOUR arch)
echo "https://aseracorp.github.io/snapraid-openwrt/aarch64_cortex-a72/packages.adb" \
  > /etc/apk/repositories.d/customfeeds.list

# 3. Refresh the index and install
apk update
apk add snapraid
```

> **Important:** include the trailing `/packages.adb` and use your exact
> architecture. If you point apk at a bare directory it falls back to
> Alpine's `APKINDEX.tar.gz` layout against the wrong `/<arch>/` path and
> fails. The URL must name the index directly — this matches how OpenWrt's
> own feeds are configured (`.../packages/<arch>/base/packages.adb`).

---

## All published architectures

Each folder below is present at the repo root `https://aseracorp.github.io/snapraid-openwrt/`:

```
aarch64_cortex-a53   aarch64_cortex-a72   aarch64_cortex-a76
aarch64_generic      arm_arm1176jzf-s_vfp  arm_arm926ej-s
arm_cortex-a15_neon-vfpv4  arm_cortex-a5_vfpv4  arm_cortex-a7
arm_cortex-a7_neon-vfpv4   arm_cortex-a7_vfpv4  arm_cortex-a8_vfpv3
arm_cortex-a9        arm_cortex-a9_neon   arm_cortex-a9_vfpv3-d16
arm_fa526            arm_xscale           armeb_xscale
i386_pentium-mmx     i386_pentium4        loongarch64_generic
mips64_mips64r2      mips64_octeonplus    mips64el_mips64r2
mips_24kc            mips_mips32          mipsel_24kc
mipsel_24kc_24kf     mipsel_74kc          mipsel_mips32
powerpc64_e5500      powerpc_464fp        powerpc_8548
riscv64_generic      x86_64
```

Each folder contains:
- `snapraid-<ver>.apk`  – the package
- `packages.adb`        – the apk package index (OpenWrt v3 / adb format)
- `packages.adb.sig`    – index signature
- `<keyid>.pub`         – the repo public key

---

## Usage

```sh
snapraid --version
man snapraid
```

Start with the sample config:

```sh
cp /usr/share/doc/snapraid/snapraid.conf.example /etc/snapraid.conf
nano /etc/snapraid.conf
snapraid sync
snapraid status
```

---

## If the package is unsigned

If `apk` reports `UNTRUSTED signature`, the publishing signing key wasn't
configured. Install with trust disabled:

```sh
apk add --allow-untrusted --repository \
  https://aseracorp.github.io/snapraid-openwrt/<arch>/packages.adb snapraid
```

---

## Build it yourself

Via `feeds.conf`:

```
src-git snapraid https://github.com/aseracorp/snapraid-openwrt.git
```

```sh
./scripts/feeds update -a
./scripts/feeds install snapraid
make menuconfig       # select Utilities > snapraid
make package/snapraid/compile
```

Or drop `package/snapraid/Makefile` into an SDK and build:

```sh
mkdir -p package/snapraid
cp package/snapraid/Makefile package/snapraid/
echo "CONFIG_PACKAGE_snapraid=y" >> .config
make defconfig
make package/snapraid/compile
```

---

## CI / release workflow

`.github/workflows/build.yml`:
- Runs **daily**, on **manual dispatch**, and on pushes to the package.
- Builds SnapRAID with the matching **OpenWrt 25.12.5** SDK for **every**
  architecture, signs the index, and pushes the assembled repo to `gh-pages`.

### Setting up package signing

```sh
usign -G -s private.key -p keys/<KEYID>.pub   # create a keypair once
base64 < private.key                          # value for the repo secret
```
1. Commit `keys/<KEYID>.pub`.
2. Add the base64 private key as the repo secret **`APK_SIGNING_KEY`**.
3. Set `KEYID` in the workflow `env`.

Until a signing key is configured, packages are published unsigned.

---

## License

- SnapRAID: GPL-3.0-or-later (© Andrea Mazzoleni, https://www.snapraid.it/)
- This OpenWrt feed packaging: GPL-2.0 (OpenWrt standard)
