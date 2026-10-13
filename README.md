# Janzeer node

Official release downloads of the **Janzeer node**: the single program that follows the chain, serves the explorer
and the API, and — once registered on-chain — validates. This repository holds the releases only; the node is
distributed in binary form (see [LICENSE.txt](LICENSE.txt)).

- Website: <https://janzeer.org> · Explorer: <https://explorer.janzeer.org>
- Announcements (releases and protocol upgrades): <https://t.me/janzeerorg>
- Documentation (Node Operator Guide, EN/AR): <https://github.com/janzeerorg/janzeer-docs> and <https://janzeer.org/docs>
- SDKs: [TypeScript](https://github.com/janzeerorg/janzeer-sdk-ts) · [Dart](https://github.com/janzeerorg/janzeer-sdk-dart) · [Kotlin](https://github.com/janzeerorg/janzeer-sdk-kotlin) · [Python](https://github.com/janzeerorg/janzeer-sdk-python) · [Go](https://github.com/janzeerorg/janzeer-sdk-go)

## Download

Take the archive for your system from [Releases](../../releases) and **verify it before you run it**:

```bash
sha256sum -c SHA256SUMS            # macOS: shasum -a 256 -c SHA256SUMS
# Windows (PowerShell): Get-FileHash janzeer-node-<version>-mainnet-windows-x64.zip -Algorithm SHA256
```

The same SHA-256 values are printed on <https://janzeer.org/validators>. If they differ, do not run the file.

From node 0.1.3 every release's `SHA256SUMS` is signed with the project's [minisign](https://jedisct1.github.io/minisign/)
key, and the bundle's `update.sh` refuses an unsigned or tampered release. Public key (key id `9F798583B345AE01`):

```
RWQBrkWzg4V5n5wUBa9GNPpa95Z7WFT2EY5QrabMv/DP+KG+gJsPIgdr
```

```bash
minisign -Vm SHA256SUMS -P RWQBrkWzg4V5n5wUBa9GNPpa95Z7WFT2EY5QrabMv/DP+KG+gJsPIgdr
```

## Run

```bash
tar -xzf janzeer-node-<version>-mainnet-linux-x64.tar.gz
cd janzeer-node-<version>-mainnet-linux-x64
./start.sh
curl -s http://localhost:7019/api/v1/info        # "syncStatus":"SYNCHRONIZED" when caught up
```

No Java, no database server and no configuration are needed: the bundle is one native executable, its runtime
libraries, the network's public `genesis.json` and a launcher. Requirements: 64-bit Linux (x86-64 or arm64) or Windows 10/11, 1 GB of free RAM,
20 GB of disk and an NTP-synchronised clock. `README.txt` inside the bundle covers ports, PostgreSQL, the systemd unit and
upgrades.

### Raspberry Pi (Linux arm64)

The same node for 64-bit ARM: `janzeer-node-<version>-mainnet-linux-arm64.tar.gz`, built for the Cortex-A53 and up,
so it runs on a Raspberry Pi 3, 4 or 5 under a 64-bit Raspberry Pi OS or Debian 12+ (`uname -m` must say `aarch64`).
Install and run exactly as the Linux x86-64 bundle above; the systemd unit inside has the install lines in its header.
A Pi with 1 GB of RAM runs the node, 2 GB or more is comfortable.

### Windows

Unzip `janzeer-node-<version>-mainnet-windows-x64.zip`, open the folder and run `start.cmd` (a plain Command Prompt is
enough). The ledger and your node identity go to `data\` next to it; `README.txt` explains ports, PostgreSQL and how to
run it as a service. Keep the same `data\` folder when you upgrade.

### Arch Linux

The same binary as a pacman package: it installs under `/opt/janzeer-node`, adds a `janzeer` system user, keeps the
ledger and your node identity in `/var/lib/janzeer-node`, and ships the systemd unit.

```bash
sudo pacman -U janzeer-node-bin-<version>-1-x86_64.pkg.tar.zst     # from Releases; or build it: cd packaging/arch && makepkg -si
sudo systemctl enable --now janzeer-node
journalctl -fu janzeer-node
```

`janzeer-node` on the command line runs it by hand as your own user (data in `~/.local/share/janzeer-node`). Ports or
PostgreSQL: `systemctl edit janzeer-node` and set the variables listed in `/opt/janzeer-node/README.txt`.
The package wraps the published Linux tarball byte for byte; its `PKGBUILD` is in [`packaging/arch/`](packaging/arch/).

## Mainnet

| | |
|---|---|
| Network id | `janzeer` |
| Genesis hash | `3a716783ee58bc56335aaede1d489eeda22cd7442c358f085a83213028b7782c` |
| Protocol | 3.4.0 |

A node whose `genesis.json` describes another chain is refused at the handshake, and stops at start with one
`GENESIS MISMATCH` line.

## Becoming a validator

A started node is watch-only. To validate: open its P2P port, read its key
(`curl -s http://localhost:7019/api/v1/info` → `nodeKey`), and register that key from a Janzeer wallet
(deposit 2000 JNZ, non-refundable, plus a 3 JNZ fee). The node joins the producer set at an epoch boundary.
Run exactly one node per key, and back up `data/config.json` — it is the node's identity.

### Validator grant — the first deposit is on the project

The deposit can only come from the game or from the project, so the **first 10 outside operators** who run a reachable,
synced mainnet node for 7 days get the 2,000 JNZ deposit (plus the 3 JNZ fee) from the ecosystem wallet, in the open on
the explorer. One grant per operator, until 2027-01-01. Rules: https://janzeer.org/validators#grant — apply with the
[Validator grant](../../issues/new?template=validator-grant.yml) issue (the queue is public), or write the same three
lines to info@janzeer.org.

## Security

Found a vulnerability? Please write to **security@janzeer.org** before disclosing it publicly.
