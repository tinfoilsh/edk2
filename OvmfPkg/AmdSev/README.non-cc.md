# Non-CC firmware

`NON_CC_BUILD=TRUE` builds the AMD firmware with the null kernel, initrd, and command-line blob verifier. This firmware is for nonconfidential guests and must not be used for confidential guests. The default is `FALSE`, which retains SEV hash verification.

Versioned releases publish `OVMF.fd`, `OVMF-noncc.fd`, and `SHA256SUMS`. Each variant builds in a separate job. Only the confidential `OVMF.fd` receives the `https://tinfoil.sh/predicate/component-artifact/v1` attestation; the non-CC artifact is excluded from that endorsement.

From the repository root, use the release build settings with the explicit non-CC flag:

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

The full firmware image is `Build/AmdSev/DEBUG_GCC5/FV/OVMF.fd`. Copy it to `OVMF-noncc.fd` before building another variant. Use a clean firmware build directory for each variant. For a local launch, select it with `tinctl dev-launch --bios` and `--disable-cc-mode`.
