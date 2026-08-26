# SnapRAID for OpenWrt

Prebuilt **[SnapRAID](https://www.snapraid.it/) 14.9** OpenWrt packages, cross-compiled
for every major OpenWrt architecture and published automatically as an **apk package
repository**.

- ⚙️ Built with the official OpenWrt SDK (musl libc) — small binaries.
- 🗂️ Packaged as an **apk** repo (OpenWrt **25.1x+**); one directory per architecture.
- 🤖 Auto-built & re-published on every new upstream SnapRAID release (daily check).
- 📄 Includes the upstream English man pages (`man snapraid`).

---

## Supported architectures

| OpenWrt arch dir | CPU | Devices |
|---|---|---|
| `x86_64` | AMD64/Intel 64 | PCs, NAS (the natural SnapRAID home) |
| `i386_pentium4` | x86 32-bit | older/32-bit x86 PCs |
| `aarch64_generic` | ARMv8-A ARM64 | armvirt, many ARM64 devices |
| `aarch64_cortex-a53` | ARMv8-A (Cortex-A53) | MediaTek Filogic, many router SoCs |
| `arm_cortex-a15_neon-vfpv4` | ARMv7/32-bit ARM hard-float | many ARM routers/NAS |
| `mipsel_24kc` | MIPS32r2 little-endian | MediaTek/Lantiq routers |
| `mips64_octeonplus` | MIPS64 little-endian | Octeon routers |
| `powerpc_8548` | PowerPC (e500) | some NAS/CPE platforms |

> SnapRAID is a **backup/parity** tool for disk arrays — most useful on devices
> with several real disks (x86 / NAS-class). On tiny router hardware it compiles
> but you rarely have the disks for it to be worthwhile.
>
> Each folder on the published repo is OpenWrt's **package architecture** for a
> target. New architectures can be added by bumping the matrix in
> `.github/workflows/build.yml`.

---

## Add the repo & trust key (OpenWrt 25.1+, apk)

The repository is served from the `gh-pages` branch as
`https://aseracorp.github.io/snapraid-openwrt/<arch>/`.

**One-time setup** (replace `<arch>` with your architecture from the table):

```sh
# 1. Install the repository verification public key (required for trust)
mkdir -p /etc/apk/keys
wget -O /etc/apk/keys/0e9e520c9ec791cf.pub \
  https://raw.githubusercontent.com/aseracorp/snapraid-openwrt/main/keys/0e9e520c9ec791cf.pub

# 2. Add the repository for your architecture, e.g. x86_64
echo "https://aseracorp.github.io/snapraid-openwrt/x86_64" >> /etc/apk/repositories

# 3. Refresh the index and install
apk update
apk add snapraid
```

Common architecture URLs:

| Device arch | Repo URL suffix |
|---|---|
| x86_64 PC/NAS | `.../snapraid-openwrt/x86_64` |
| i686/x86 32-bit PC | `.../snapraid-openwrt/i386_pentium4` |
| aarch64 (ARM64) | `.../snapraid-openwrt/aarch64_generic` |
| ARM 32-bit hard-float | `.../snapraid-openwrt/arm_cortex-a15_neon-vfpv4` |
| MediaTek Filogic | `.../snapraid-openwrt/aarch64_cortex-a53` |
| mips32r2 (mt7621) | `.../snapraid-openwrt/mipsel_24kc` |
| Octeon (MIPS64) | `.../snapraid-openwrt/mips64_octeonplus` |
| PowerPC e500 | `.../snapraid-openwrt/powerpc_8548` |

### If the package is unsigned

If `apk` reports `UNTRUSTED signature`, the publishing signing key wasn't
configured for this release. Install with trust disabled:

```sh
apk add --allow-untrusted --repository \
  https://aseracorp.github.io/snapraid-openwrt/<arch> snapraid
```

---

## Usage

```sh
snapraid --version
man snapraid          # full man page included
```

Start with the sample config:

```sh
cp /usr/share/doc/snapraid/snapraid.conf.example /etc/snapraid.conf
nano /etc/snapraid.conf
snapraid sync
snapraid status
```

---

## OpenWrt ≤ 23.x (opkg)

Releases older than 25.x use **opkg**, which needs a `Packages` index, while
this repo publishes the newer apk `.adb` index. For opkg devices either use
OpenWrt 25.1+ (recommended) or build from source (below).

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

Or drop the `Makefile` straight into an SDK and build:

```sh
mkdir -p package/snapraid
cp package/snapraid/Makefile package/snapraid/   # from a checkout of this repo
echo "CONFIG_PACKAGE_snapraid=y" >> .config
make defconfig
make package/snapraid/compile
```

---

## CI / release workflow

`.github/workflows/build.yml`:
- Runs **daily**, on **manual dispatch**, and on pushes to the package.
- Builds SnapRAID with the matching **OpenWrt 25.12.5** SDK for each arch,
  signs the index, and pushes the assembled repo to `gh-pages`.

### Setting up package signing

```sh
usign -G -s private.key -p keys/<KEYID>.pub   # create a keypair once
base64 < private.key                          # value for the repo secret
```
1. Commit `keys/<KEYID>.pub` to this repository.
2. Add the base64 private key as the repo secret **`APK_SIGNING_KEY`**.
3. Set `KEYID` in the workflow `env`.

Until a signing key is configured, packages are published unsigned (see
"if unsigned" above).

---

## License

- SnapRAID: GPL-3.0-or-later (© Andrea Mazzoleni, https://www.snapraid.it/)
- This OpenWrt feed packaging: GPL-2.0 (OpenWrt standard)
