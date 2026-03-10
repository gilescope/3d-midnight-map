# 3D Midnight Block Explorer

A real-time 3D block explorer for the Midnight preview network, built with Three.js and WebXR. Connects via WebSocket to a Substrate RPC node and visualises blocks, extrinsics, and finalization status as a force-directed graph. Cardano preview blocks are displayed alongside, linked via the `mcsh` digest.

**Live:** [GitHub Pages deployment](https://gilescope.github.io/3d-midnight-map/)

## Features

- Real-time block subscription via `chain_subscribeNewHeads` / `chain_subscribeFinalizedHeads`
- Blocks shown as cyan (unfinalized) or gold (finalized) icosahedra
- Extrinsics radiate from their parent block (magenta/orange)
- Cardano preview blocks (blue) linked to Midnight blocks via mcsh cross-chain references
- Force-directed 3D layout with gentle spring physics
- Double-click a block to open it in Polkadot.js Apps or CardanoScan
- Apple Vision Pro support: both-hands pinch-to-zoom, gaze dwell-to-select
- Cyberpunk aesthetic: bloom, scanlines, particle edges, Orbitron font

## How It Works

### Midnight RPC

Connects to `wss://rpc.preview.midnight.network` (Substrate JSON-RPC 2.0 over WebSocket). Subscribes to new heads and finalized heads. For each new block, fetches the full block with `chain_getBlock` to get extrinsics and digest logs.

### mcsh Digest: Linking Midnight to Cardano

Each Midnight block header contains digest log entries. The `mcsh` engine (Midnight Cardano Stable Hash) records the most recent Cardano block hash known at the time of Midnight block production.

#### Digest log binary format

```text
Byte 0:     0x06 (PreRuntime variant)
Bytes 1-4:  0x6d637368 ('mcsh' ASCII = engine ID)
Byte 5:     0x80 (SCALE compact encoding of 32)
Bytes 6-37: 32-byte Cardano block hash
```

In the hex-encoded log string (after stripping `0x` prefix):

| Offset | Length | Content                          |
| ------ | ------ | -------------------------------- |
| 0-1    | 2      | `06` PreRuntime type             |
| 2-9    | 8      | `6d637368` engine ID (`mcsh`)    |
| 10-11  | 2      | `80` SCALE compact length (32)   |
| 12-75  | 64     | Cardano block hash (hex-encoded) |

#### Example

From a Midnight preview block digest log:

```text
0x066d63736880 0b6b87c5daace9a80cbd019b0ad3381a2c1b253b20dd969cadac0f93f830077a
```

- Engine: `mcsh`
- Cardano hash: `0b6b87c5daace9a80cbd019b0ad3381a2c1b253b20dd969cadac0f93f830077a`
- Confirmed as Cardano preview block #4087966 via [Koios API](https://preview.koios.rest/api/v1/block_info)

### Midnight Indexer

Transaction fee data (tDUST) is fetched from the [Midnight Indexer](https://indexer.preview.midnight.network/api/v3/graphql) GraphQL API. For blocks containing ledger transactions (ZK proofs, contract calls), the indexer provides `paidFees` and `estimatedFees`. System extrinsics (Timestamp, consensus) don't carry dust fees.

```graphql
{ block(offset: { height: 12345 }) {
    transactions { ... on RegularTransaction { fees { paidFees estimatedFees } } }
} }
```

## Running Locally

```bash
python3 -m http.server 8080
```

Open <http://localhost:8080>. For Vision Pro WebXR testing, you need HTTPS (use ngrok or deploy to GitHub Pages).

## Tech Stack

- [Three.js](https://threejs.org/) r160 via importmap (unpkg CDN)
- [Troika-three-text](https://github.com/protectwise/troika/tree/main/packages/troika-three-text) for SDF text in VR
- WebXR (VRButton, immersive-vr) for Apple Vision Pro
- UnrealBloomPass for non-VR glow; emissive materials for VR
- Substrate JSON-RPC over WebSocket
- Midnight Indexer GraphQL API for transaction fee data
- Single `index.html` file, no build step
