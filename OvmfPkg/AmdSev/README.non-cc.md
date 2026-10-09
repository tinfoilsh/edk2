# Non-CC test firmware

`NON_CC_BUILD=TRUE` builds the AMD firmware with the null kernel, initrd, and command-line blob verifier. This artifact is for non-CC testing only and must not be used for confidential guests. The default is `FALSE`, which retains SEV hash verification.

From the repository root at the pinned v0.0.4 source (commit `6488a73a3e65b941db69d0854131b3c3115f89b1`), use the release build settings with the explicit non-CC flag:

```sh
sudo apt-get update
sudo apt-get install -y build-essential uuid-dev acpica-tools nasm gcc python3 python3-venv
git submodule update --init --recursive
python3 -m venv .venv
. .venv/bin/activate
touch OvmfPkg/AmdSev/Grub/grub.efi
make -C BaseTools
. ./edksetup.sh --reconfig
build -q --cmd-len=64436 -DDEBUG_ON_SERIAL_PORT=TRUE -DNON_CC_BUILD=TRUE -n 32 -t GCC5 -a X64 -b DEBUG -p OvmfPkg/AmdSev/AmdSevX64.dsc
```

The full firmware image is `Build/AmdSev/DEBUG_GCC5/FV/OVMF.fd`. Copy it to a separately named non-CC test artifact before building another variant, and select it only for the test VM through `tinctl dev-launch --bios`. Keep the configured confidential firmware unchanged. Use a clean firmware build directory for each variant.

Both variants compiled with the release settings on October 9, 2026. The library reports selected `BlobVerifierLibSevHashes` by default and `BlobVerifierLibNull` with the explicit flag. On inf6, the non-CC variant booted CVM 0.14.13 with 4 vCPUs, 16 GiB RAM, and one H200: QEMU started in 12 seconds and the VM reached Ready in 69 seconds. SSH and the HTTPS debug endpoint worked. The pinned confidential firmware had stopped at `BdsDxe: No bootable option was found` in the earlier non-CC launch.
