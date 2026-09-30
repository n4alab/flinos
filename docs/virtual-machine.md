# Create a FLiNOS virtual machine

> **Pre-release production profile.** This document specifies the intended
> production deployment model. It is not available until a FLiNOS release
> publishes the complete signed bundle described below.

FLiNOS production appliances are designed to run as UEFI virtual machines with
Secure Boot enabled. A QCOW image on its own is not a complete deployment
artifact: the release firmware, NVRAM template, and signed manifest are part of
the boot trust chain.

## Host requirements

The host running the virtual machine must provide:

- An `amd64` CPU and QEMU/KVM suitable for the release.
- A `q35` virtual machine type.
- OVMF firmware with UEFI Secure Boot support.
- Access to the release-provided OVMF `CODE` and `VARS` files.
- Sufficient memory, CPU, storage, and network connectivity for the appliance
  and its attached interfaces.

The production profile does not support legacy BIOS or an insecure UEFI boot.

## Release bundle

For a release named `<release>`, obtain all artifacts from the same signed
release bundle:

| Artifact | Purpose |
| --- | --- |
| `flinos-<release>.qcow2` | UEFI appliance disk |
| `flinos-<release>.json` | Release metadata and artifact inventory |
| `flinos-<release>.version` | Release identifier |
| `flinos-<release>.manifest` and `.sig` | Signed manifest used to verify the release contents |
| `flinos-<release>-OVMF_CODE.fd` | Read-only Secure Boot firmware code |
| `flinos-<release>-OVMF_VARS.fd` | NVRAM template containing the required trust configuration |

Verify the manifest and signature according to the release instructions before
creating a VM. Do not mix the QCOW, firmware, or manifest from different
releases.

## Create and start the VM

The OVMF `CODE` file is immutable and may be shared by VMs of the same release.
The `VARS` file is writable NVRAM state and must be copied once per VM. Never
boot a VM directly against the distributed `VARS` template.

```bash
release_dir=/path/to/flinos-<release>
vm_state=/var/lib/flinos-vm/lab-router-1

install -d -m 0700 "$vm_state"
cp "$release_dir/flinos-<release>-OVMF_VARS.fd" "$vm_state/OVMF_VARS.fd"

qemu-system-x86_64 \
  -enable-kvm \
  -machine q35,smm=on \
  -cpu host \
  -m 2048 \
  -smp 2 \
  -drive if=pflash,format=raw,readonly=on,file="$release_dir/flinos-<release>-OVMF_CODE.fd" \
  -drive if=pflash,format=raw,file="$vm_state/OVMF_VARS.fd" \
  -drive if=virtio,format=qcow2,file="$release_dir/flinos-<release>.qcow2" \
  -serial mon:stdio
```

Attach management and data interfaces using the network backend required by
your virtualization platform. The exact QEMU network arguments are deployment
specific and are intentionally not part of this minimal VM creation example.

## Boot enforcement and validation

The supported production path is fail-closed:

- The VM must boot with the release OVMF Secure Boot chain and NVRAM template.
- A legacy BIOS launch is unsupported.
- A UEFI launch with Secure Boot disabled is unsupported; the FLiNOS boot
  environment must refuse to mount or start the appliance services.
- Missing, unreadable, or modified `VARS` state must be treated as an
  unsuccessful deployment, not as a reason to fall back to BIOS.

After boot, confirm the UEFI Secure Boot state from the guest using the command
provided by the release. A successful boot must also make the FLiNOS console
available. If either check fails, stop the VM and correct the firmware, NVRAM,
or release bundle rather than disabling Secure Boot.

## Persistence

The QCOW disk retains FLiNOS configuration and operational state. The
per-instance `OVMF_VARS.fd` retains UEFI NVRAM state for that VM. Back up both
files together when preserving a lab node, and never share one writable `VARS`
file among multiple concurrent VMs.

## Security boundary

Secure Boot, signed release artifacts, and runtime integrity checks establish
that the standard boot path is using approved artifacts. They help detect
corruption and unauthorized modification of a VM deployment.

They do not protect FLiNOS from an administrator who controls the hypervisor,
QEMU process, firmware files, or VM storage. Such an administrator can replace
the trust chain itself. Treat this profile as integrity protection, not as a
confidential-computing boundary.

## dNLab

FLiNOS is intended to be compatible with [dNLab](https://github.com/n4alab/dnlab),
which manages Containerlab-based virtual labs across one or more worker nodes.
This guide does not define dNLab deployment, image import, or topology
workflows; use the dNLab documentation for those operations. Compatibility is
planned until a FLiNOS release explicitly publishes its validation results.
