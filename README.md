# pod-qemu-guest-agent

The `qemu-guest-agent` candy of the OpenCharly candy library, as a standalone
repo (the candy de-submodule cutover, kind-prefixed naming). It installs the QEMU
guest agent — the full host↔guest control channel with an
application-consistent `fsfreeze` snapshot hook.

## What it provides

Installs the `qemu-guest-agent` package (identical name on Arch/CachyOS and
Fedora), enables the system service, and writes `/etc/qemu/qemu-ga.conf` pinning
the full RPC surface (`guest-exec`, `guest-file-*`, `guest-fsfreeze-*`,
`guest-set-*` — the package default blocks none). The `fsfreeze` hook dispatcher
runs every executable in `/etc/qemu/fsfreeze-hook.d/` on freeze/thaw, so the host
can take application-consistent snapshots.

| Property | Value |
|---|---|
| Package | `qemu-guest-agent` (arch + fedora) |
| Service | `qemu-guest-agent` (`use_packaged: qemu-guest-agent.service`, system scope) |
| Config | `/etc/qemu/qemu-ga.conf` (`fsfreeze-hook=/etc/qemu/fsfreeze-hook`) |
| Hook | `/etc/qemu/fsfreeze-hook` (dispatcher) + `/etc/qemu/fsfreeze-hook.d/` (drop-ins) |

The guest-side virtio-serial channel (`org.qemu.guest_agent.0`) is declared on the
VM entity, not this candy — see the vm-libvirt deploy's own
`libvirt.devices.channels:`.

## How to use it

```yaml
my-vm-image:
  bootc: true
  candy:
    - '@github.com/opencharly/pod-qemu-guest-agent:<tag>'
```

or applied to a VM guest at deploy time:

```bash
charly fleet add vm:<name> qemu-guest-agent
```

Drop per-application scripts into `/etc/qemu/fsfreeze-hook.d/` to run them on
snapshot freeze/thaw.

## Layout

- `charly.yml` — the `qemu-guest-agent:` candy entity (description, `package`,
  `service`, `plan`) plus its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-distros:qemu-guest-agent` — the candy properties, the
  full RPC surface, the `fsfreeze` hook, and the virtio-serial channel.
- `/charly-vm:vm` — VM lifecycle and the QEMU-user-net caveat.
- `/charly-internals:libvirt-renderer` — the renderer that injects the channel.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
