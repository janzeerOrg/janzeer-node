# Janzeer node

Official release downloads of the **Janzeer node**: the single program that follows the chain, serves the explorer
and the API, and — once registered on-chain — validates. This repository holds the releases only; the node is
distributed in binary form (see [LICENSE.txt](LICENSE.txt)).

- Website: <https://janzeer.org> · Explorer: <https://explorer.janzeer.org>
- Announcements (releases and protocol upgrades): <https://t.me/janzeerorg>
- Documentation (Node Operator Guide, EN/AR): <https://github.com/janzeerorg/janzeer-docs> and <https://janzeer.org/docs>
- SDKs: [TypeScript](https://github.com/janzeerorg/janzeer-sdk-ts) · [Dart](https://github.com/janzeerorg/janzeer-sdk-dart) · [Kotlin](https://github.com/janzeerorg/janzeer-sdk-kotlin)

## Download

Take the archive for your system from [Releases](../../releases) and **verify it before you run it**:

```bash
sha256sum -c SHA256SUMS            # macOS: shasum -a 256 -c SHA256SUMS
```

The same SHA-256 values are printed on <https://janzeer.org/validators>. If they differ, do not run the file.

## Run

```bash
tar -xzf janzeer-node-<version>-mainnet-linux-x64.tar.gz
cd janzeer-node-<version>-mainnet-linux-x64
./start.sh
curl -s http://localhost:7019/api/v1/info        # "syncStatus":"SYNCHRONIZED" when caught up
```

No Java, no database server and no configuration are needed: the bundle is one native executable, its runtime
libraries, the network's public `genesis.json` and a launcher. Requirements: 64-bit Linux, 1 GB of free RAM, 20 GB of
disk and an NTP-synchronised clock. `README.txt` inside the bundle covers ports, PostgreSQL, the systemd unit and
upgrades.

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
