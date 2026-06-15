<p align="center">
  <img src="src/qt/res/icons/universalmolecule.png" alt="UniversalMolecule" width="95">
</p>

# UniversalMolecule Core 0.25.2

UniversalMolecule Core 0.25.2 is the UniversalMolecule v25.2 codebase
with AuxPoW merged-mining support, descriptor wallet support, legacy
Berkeley DB wallet support, SQLite wallet support, ZMQ, and Linux USDT
tracepoints for hardened release builds.
The repository ships build and release packaging for native Ubuntu `20.04`,
`22.04`, `24.04`, `26.04`, Windows, native macOS, macOS cross-builds, and
Ubuntu `22.04+` AppImage builds.

## Mainnet Consensus Changes In 0.25.2

UniversalMolecule 0.25.2 follows the UniversalMolecule 0.15.21 chain and
inherits SegWit as already-active mainnet history. Miners and pools should use
the daemon-provided block template version and must not rewrite version bits.

| Rule set | Mainnet policy in UniversalMolecule 0.25.2 |
|---|---|
| SegWit (`BIP141` / `BIP143` / `BIP147`) | Already active from 0.15.21; buried at height `4108645`. No new SegWit signaling window in 0.25.2. |
| `BIP34` coinbase height | Height activation at `4141188`; `BIP34Hash = uint256{}`. |
| `BIP65` / CLTV | Height activation at `4141188`; required for standard CLTV atomic-swap refunds. |
| `BIP66` / strict DER | Height activation at `4141188`. |
| Taproot (`BIP340` / `BIP341` / `BIP342`) | BIP9 deployment bit `2`, start `1782871200` (`2026-07-01 02:00:00 UTC`), timeout `1814407200` (`2027-07-01 02:00:00 UTC`), minimum activation height `4146228`. |

Only Taproot is a future BIP9-signaled deployment in 0.25.2. `BIP34`,
`BIP65`, `BIP66`, and buried SegWit are height rules. Pools and solo miners
should request templates with the normal daemon RPCs; UniversalMolecule Core
computes the correct top bits, AuxPoW chain bits, and Taproot bit `2` during
the BIP9 `started` and `locked_in` states.

## About UniversalMolecule

UniversalMolecule is an AuxPoW merged-mined Blake-256 coin in the BlakeStream
family.

- Uses the Blake-256 hashing algorithm, 8 rounds
- Based on the upstream v25.2 Core codebase
- Supports AuxPoW merged mining with chain ID `0x000F`
- Supports legacy Berkeley DB wallets and descriptor SQLite wallets
- Keeps UniversalMolecule txids on single SHA-256
- Uses HASH256/double SHA-256 for witness-v0 BIP143 signing
- Keeps BIP340/BIP341/BIP342 Taproot tagged hashes byte-compatible with upstream vectors

| Network Info | Value |
|---|---|
| Algorithm | Blake-256, 8 rounds |
| Block time | 2 minutes |
| Difficulty retarget | Every 5 blocks |
| Coinbase maturity | 120 blocks |
| Mainnet P2P port | 24785 |
| Mainnet RPC port | 5921 |
| Mainnet genesis | `00000059f24d9e85501bd3873fac0cd6e8a43fd8c20eee856082dbdcc09a8e64` |
| Mainnet Bech32 HRP | `umo` |
| Testnet Bech32 HRP | `tumo` |
| Regtest Bech32 HRP | `rumo` |

## Block Subsidy

UniversalMolecule keeps the legacy 0.8/0.15.21 difficulty-trend reward rule:

- Height `0`: `1 UMO`
- Heights `1` through `1440`: `0.001 UMO`
- Heights above `1440`: `0.1 UMO` when difficulty is rising, otherwise `2 UMO`

The historical `3600` to `4000` reward comparison quirk is preserved for chain
continuity.

For configuration examples, see
[`share/examples/universalmolecule.conf`](share/examples/universalmolecule.conf).
For the full runtime option list, run `universalmoleculed -help`.

## Quick Start

```bash
git clone https://github.com/SidGrip/universalmol.git
cd universalmol
git switch 0.25.2
bash ./build.sh --help
```

For most users, downloading a prebuilt release from GitHub Releases is the
simplest path. Use `build.sh` when you need to build the release artifacts
locally.

## UPnP / miniupnpc Build Profiles

UPnP is only for desktop or home-router nodes that need automatic inbound P2P
port mapping. Nodes still sync normally through outbound peers without UPnP.

Pool, explorer, server, and Docker daemon builds should disable UPnP with
`--without-miniupnpc` so the binary has no `libminiupnpc.so.*` runtime
dependency.

UPnP-enabled Ubuntu builds need `libminiupnpc-dev` at build time and the matching
`libminiupnpc` runtime package on the target host.

## Upgrade Notes

Before starting UniversalMolecule Core 0.25.2 on an existing data directory,
close the older wallet cleanly and back up wallet files.

When syncing 0.25.2 from old 0.8/0.15.21-era chains, header presync can look slow or restart because v25 verifies low-work header chains before storing them. For trusted bootstrap only, use `-minimumchainwork=0 -connect=<trusted-node>` and remove those options after the node catches up.

`peers.dat` is only the cached P2P address database. It is safe to remove or
rename when moving between major releases, and UniversalMolecule will rebuild it
on the next start. If startup fails with `Invalid or corrupt peers.dat`, remove
or rename this file:

- Windows: `%APPDATA%\UniversalMolecule\peers.dat`
- Linux: `~/.universalmolecule/peers.dat`
- macOS: `~/Library/Application Support/UniversalMolecule/peers.dat`

Windows PowerShell example:

```powershell
Rename-Item "$env:APPDATA\UniversalMolecule\peers.dat" "peers.dat.bak"
```

Linux example:

```bash
mv ~/.universalmolecule/peers.dat ~/.universalmolecule/peers.dat.bak
```

macOS example:

```bash
mv "$HOME/Library/Application Support/UniversalMolecule/peers.dat" \
   "$HOME/Library/Application Support/UniversalMolecule/peers.dat.bak"
```

If the block index or chainstate database cannot be reused after an upgrade,
restart once with `-reindex` to rebuild the local block database from the stored
block files:

```bash
universalmoleculed -reindex
```

Pruning is disabled by default (`-prune=0`). Public release nodes, explorers,
pools, and bridge/watch services should run unpruned unless they have a specific
reason to discard old block data.

For first-run testing of a new 0.25.2 build, use an isolated data directory so
the test does not touch an existing wallet or chainstate:

```bash
universalmolecule-qt -datadir=/path/to/universalmolecule-25.2-test
```

## Build Options

```bash
bash ./build.sh [PLATFORM] [TARGET] [OPTIONS]

Platforms:
  --native          Build natively on this machine (Linux, macOS, or Windows)
  --appimage        Build portable Linux AppImage
  --windows         Cross-compile for Windows from Linux
  --macos           Cross-compile for macOS from Linux

Targets:
  --daemon          Build daemon only
  --qt              Build Qt wallet only
  --both            Build daemon and Qt wallet (default)

Docker options:
  --pull-docker     Pull prebuilt Docker images from Docker Hub
  --build-docker    Build Docker images locally from repo Dockerfiles
  --no-docker       For --native on Linux: build directly on the host

Other options:
  --hardened-release
                   Native Linux release profile: enable SQLite, ZMQ, and USDT
                   and fail the build if configure disables any of them
  --jobs N          Parallel make jobs
```


<!-- BEGIN electrium-build -->
### Electrium Wallet

Build the UniversalMolecule ([Electrium](https://github.com/BlueDragon747/Blakestream-Electrum)) wallet by
choosing a target (linux/windows build in an **amd64** container, so any amd64 Docker host — Linux,
Windows, or an Intel Mac — can build either; only the macOS app needs a Mac):

```bash
./build-electrum.sh linux      # Linux AppImage    (amd64 Docker host)
./build-electrum.sh windows    # Windows .exe      (amd64 Docker host)
./build-electrum.sh macos      # macOS .dmg/.app   (on a Mac)
./build-electrum.sh all        # everything buildable on this host
```

Artifacts land in `outputs/Electrium/UMO/`, named `Electrium-UMO-<version>`.

The wallet builder is the shared multicoin repo
**[BlueDragon747/Blakestream-Electrum](https://github.com/BlueDragon747/Blakestream-Electrum)** — it also builds
all six BlakeStream wallets at once (`build-single-wallets.sh`) and the ElectrumX **server** Docker
image (`build-electrumx.sh`). `build-electrum.sh` auto-clones it when no local checkout is found.
<!-- END electrium-build -->

## Platform Build Instructions

### Native Linux

```bash
bash ./build.sh --native --both
```

- On supported Ubuntu hosts, `build.sh` auto-detects the OS version and installs
  missing packages automatically.
- Native Linux release packaging targets Ubuntu `20.04`, `22.04`, `24.04`, and
  `26.04`.
- Native Linux builds write directly to `outputs/`.
- `--both` refreshes the full Ubuntu-native wallet files directly in `outputs/`.
- `--daemon` refreshes the daemon-side Linux files directly in `outputs/`.
- `--qt` refreshes the Qt wallet files directly in `outputs/`.
- Native Ubuntu outputs are bare same-Ubuntu binaries that rely on host-installed
  native packages.
- Native Ubuntu builds bootstrap Berkeley DB `4.8.30.NC` into a local repo cache
  and always link legacy wallet builds against that copy.
- SQLite descriptor wallet support is enabled by default for 0.25.2 release
  builds.
- Each Ubuntu output folder gets its own `install-deps.sh`, `README.md`,
  `build-info.txt`, and `universalmolecule.conf`.

### Linux With Docker

Use `--pull-docker` to pull prebuilt images from Docker Hub, or
`--build-docker` to build them locally from the Dockerfiles in `docker/`.

```bash
bash ./build.sh --native --both --pull-docker
bash ./build.sh --native --qt --pull-docker
bash ./build.sh --native --daemon --pull-docker
bash ./build.sh --native --both --build-docker
```

The default native Docker image is `sidgrip/native-base:24.04`. Set
`DOCKER_NATIVE` when you want a specific Ubuntu lane:

```bash
DOCKER_NATIVE=sidgrip/native-base:26.04 \
  bash ./build.sh --native --both --build-docker
```

### Hardened Linux Release

Recommended hardened Ubuntu 26 release build:

```bash
DOCKER_NATIVE=sidgrip/native-base:26.04 \
  bash ./build.sh --native --both --build-docker --hardened-release --jobs 5
```

The hardened Linux release profile requires:

- `USE_BDB=true`
- `USE_SQLITE=true`
- `ENABLE_ZMQ=true`
- `ENABLE_USDT_TRACEPOINTS=true`

USDT runtime attach validation is Linux/eBPF-specific. macOS and Windows builds
do not fail release acceptance because they do not expose the Linux USDT backend.

### AppImage

```bash
bash ./build.sh --appimage --pull-docker
```

- Uses `sidgrip/appimage-base:22.04`.
- Produces a self-contained AppImage in `outputs/AppImage/`.
- The output folder keeps `UniversalMolecule-0.25.2-x86_64.AppImage`,
  `README.md`, and `build-info.txt`.
- Intended for Ubuntu `22.04+`.
- Direct launch on Ubuntu `22.04.5` needs `sudo apt install libfuse2`.
- Direct launch on Ubuntu `24.04.4` and `26.04` needs
  `sudo apt install libfuse2t64`.
- If the host is missing that package, AppImage runtime startup fails with
  `dlopen(): error loading libfuse.so.2`.
- Fallback launch remains `--appimage-extract-and-run`.

### Windows

Cross-build from Linux:

```bash
bash ./build.sh --windows --both --pull-docker
```

- Runs on Linux with Docker using `sidgrip/mxe-base:latest`.
- Writes loose cross-built outputs to `outputs/Windows/`.

Native Windows builds are supported from an MSYS2/MinGW shell:

```bash
bash ./build.sh --native --both
```

### macOS

Cross-build from Linux:

```bash
bash ./build.sh --macos --both --pull-docker
```

- Runs on Linux with Docker using `sidgrip/osxcross-base:sdk-26.2`.
- Produces artifacts in `outputs/Macosx/`.

Native build on macOS:

```bash
bash ./build.sh --native --both
```

- Uses Homebrew on the Mac host.
- `build.sh` installs missing Homebrew dependencies automatically.
- Native macOS builds write to `outputs/Macosx/`.

## Output Structure

```text
outputs/
├── AppImage/
│   ├── UniversalMolecule-0.25.2-x86_64.AppImage
│   ├── README.md
│   └── build-info.txt
├── Macosx/
│   ├── UniversalMolecule-Qt.app
│   ├── universalmolecule-cli-0.25.2
│   ├── universalmolecule-qt-0.25.2
│   ├── universalmolecule-tx-0.25.2
│   ├── universalmolecule-util-0.25.2
│   ├── universalmolecule-wallet-0.25.2
│   ├── universalmoleculed-0.25.2
│   └── build-info.txt
├── Ubuntu-20/
│   ├── README.md
│   ├── universalmolecule-256.png
│   ├── universalmolecule-cli
│   ├── universalmolecule.conf
│   ├── universalmolecule.desktop
│   ├── universalmolecule-qt
│   ├── universalmolecule-qt-bin
│   ├── universalmolecule-tx
│   ├── universalmolecule-util
│   ├── universalmolecule-wallet
│   ├── universalmoleculed
│   ├── build-info.txt
│   └── install-deps.sh
├── Ubuntu-22/
├── Ubuntu-24/
├── Ubuntu-26/
└── Windows/
    ├── universalmolecule-cli-0.25.2.exe
    ├── universalmolecule-qt-0.25.2.exe
    ├── universalmolecule-tx-0.25.2.exe
    ├── universalmolecule-util-0.25.2.exe
    ├── universalmolecule-wallet-0.25.2.exe
    ├── universalmoleculed-0.25.2.exe
    └── build-info.txt
```

For Ubuntu native builds, the current host's final wallet files land in
`outputs/Ubuntu-20/`, `outputs/Ubuntu-22/`, `outputs/Ubuntu-24/`, or
`outputs/Ubuntu-26/` depending on the detected Ubuntu release. These are bare
Ubuntu-native binaries, so each Ubuntu folder gets its own `install-deps.sh`,
`README.md`, `build-info.txt`, and `universalmolecule.conf`. Berkeley DB
`4.8.30.NC` is bootstrapped into a local repo cache by the builder rather than
being installed from apt.

For Windows cross-builds from Linux, the output bundle lands in
`outputs/Windows/` and contains the versioned `.exe` binaries plus
`build-info.txt`.

For native macOS builds, the current host's daemon tools,
`UniversalMolecule-Qt.app`, and the raw `universalmolecule-qt-0.25.2` binary
land in `outputs/Macosx/`.

For AppImage builds, `outputs/AppImage/` keeps
`UniversalMolecule-0.25.2-x86_64.AppImage`, `README.md`, and `build-info.txt`.

## Docker Images

When using `--pull-docker`, the build script uses these prebuilt images:

| Image | Purpose |
|---|---|
| `sidgrip/native-base:20.04` | Native Linux Ubuntu 20.04 build |
| `sidgrip/native-base:22.04` | Native Linux Ubuntu 22.04 build |
| `sidgrip/native-base:24.04` | Native Linux Ubuntu 24.04 build |
| `sidgrip/native-base:26.04` | Native Linux Ubuntu 26.04 build |
| `sidgrip/appimage-base:22.04` | Ubuntu 22+ AppImage build |
| `sidgrip/mxe-base:latest` | Windows cross-compile |
| `sidgrip/osxcross-base:sdk-26.2` | macOS cross-compile |

## Tests

From a configured build tree:

```bash
make -C src -j5 test/test_universalmolecule
src/test/test_universalmolecule
python3 test/functional/test_runner.py feature_segwit.py feature_cltv.py feature_csv_activation.py feature_taproot.py wallet_taproot.py feature_auxpow_rpc.py feature_auxpow_segwit.py
```

Regtest and signet can be used for local feature testing. Mainnet activation
values are fixed in chain parameters and should not be changed without a planned
network release.

## Multi-Coin Builder

For coordinated BlakeStream-family wallet builds, see the
[Blakestream Installer](https://github.com/SidGrip/Blakestream-Installer).

## License

UniversalMolecule Core is released under the terms of the MIT license. See
[COPYING](COPYING) for more information.
