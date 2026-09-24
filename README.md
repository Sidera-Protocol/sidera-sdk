# @sidera-protocol/sdk

**TypeScript client for the Sidera Stellar Name Service — send payments to
`alice.sid`, not to a checksum.**

Sidera maps human-readable names to Stellar addresses (G- and M-keys) with
**memo hints**, so exchange payments stop getting lost and wallets stop
asking users to paste 56 characters. This SDK is the resolution engine for
wallets (Freighter, Albedo, …) and payment apps.

## Installation

```bash
npm   install @sidera-protocol/sdk
pnpm  add    @sidera-protocol/sdk
yarn  add    @sidera-protocol/sdk
```

## Initialization

```ts
import { SideraClient } from "@sidera-protocol/sdk";

const client = new SideraClient({
  contractId: "C…",       // deployed Sidera registry contract ID
  network: "testnet",     // "testnet" | "mainnet" | custom (with rpcUrl)
  // rpcUrl: "http://localhost:8000",  // optional override
});
```

Network defaults:

| Network  | RPC endpoint (public)                    |
| -------- | ---------------------------------------- |
| testnet  | `https://soroban-testnet.stellar.org`    |
| mainnet  | `https://soroban-rpc.stellar.org`        |

## Resolving a name (reads)

```ts
const res = await client.resolve("alice.sid");

res.fullName;      // "alice.sid"
res.address;       // "GA7X…" — destination
res.owner;         // "GABC…" — name owner
res.memo?.type;    // "id" | "text" | null
res.memo?.value;   // the memo to attach
```

Wallets: feed `res.memo` straight into your payment builder —
`type: "id"` maps to `Memo.id(value)`, `type: "text"` to `Memo.text(value)`.
**If a memo hint is present, refusing to attach it should be a hard error in
your send flow** — a memo-less payment to an exchange is a lost payment.

Also available: `client.ownerOf(name)`, `client.exists(name)`.

## Writing to the registry (transactions)

Write calls build, sign, and submit real Soroban transactions. Provide a
signer in the client config (Freighter/Albedo adapters implement the same
two-line interface):

```ts
const client = new SideraClient({
  contractId: "C…",
  network: "testnet",
  signer: {
    signTx: async (txXdr, networkPassphrase) =>
      freighterApi.signTransaction(txXdr, { networkPassphrase }),
  },
});

// Register a name (owner signs):
const hash = await client.register({
  name: "alice",
  owner: ownerAddress,        // must be the signing account
  address: destinationAddress,
  memo: "1029384756",         // optional memo hint
});

// Re-point a name:
await client.setAddress({ name: "alice", address: newAddress }, ownerAddress);

// Transfer ownership:
await client.transfer({ name: "alice", from: ownerAddress, to: newOwner });
```

## Name rules

- 3–32 characters, `a–z`, `0–9`, inner hyphens (no leading/trailing `-`)
- Input is normalized: `Alice.SID` → `alice`
- Validation runs client-side first (instant feedback), then on-chain

## Development

```bash
pnpm install
pnpm lint    # tsc --noEmit (strict)
pnpm test    # vitest
pnpm build   # tsup → dist/ (ESM + d.ts)
```

## License

MIT
