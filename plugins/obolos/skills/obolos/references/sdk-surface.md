# SDK surface and integration rules

Read this for implementation, API selection, batching adapters or SDK review.

## Client shapes

The SDK offers two equivalent styles:

- address-parameterized functions from domain subpaths such as `@obolos/sdk/company`,
  `@obolos/sdk/fundraise` and `@obolos/sdk/compliance`;
- `createObolosClient({ chainId, addresses?, provider?, signer?, executor? })`, which binds shared
  chain context and delegates to the same domain functions.

Use the bound client for applications and coordinated workflows. Use free functions in narrow
libraries, backend jobs and tests, where explicit addresses make dependencies clearer. Reads take a
provider and return normalized protocol values. Writes return `Prepared` and take no signer.

## Transaction model

```ts
type BatchCall = { to: Address; data: Hex; value?: bigint };

type Executor = {
  readonly atomic: boolean;
  send(calls: readonly BatchCall[]): Promise<Hash>;
};

interface Prepared {
  readonly label: string;
  readonly calls: readonly BatchCall[];
  execute(runner?: Executor | Signer, opts?: { atomic?: boolean }): Promise<Hash>;
}
```

Building a `Prepared` operation is safe and sends nothing.

- `batch([opA, condition && opB, opC])` composes compatible operations into one; falsy items are
  discarded.
- When all calls must succeed together, require an `Executor` with `atomic: true` and pass
  `{ atomic: true }` at the execution boundary.
- `fromSigner(signer)` sends one transaction per call and reports `atomic: false`; treat a bare ethers
  `Signer` the same way. Use it only for an explicitly sequential plan whose partial progress is
  understood.
- Split receipt-, proof-, oracle- and threshold-dependent work into stages (see the step classes in
  [workflows.md](workflows.md)). A wallet batch does not remove those state boundaries.
- Never serialize `Prepared.execute`. Across a process boundary, carry the label and `BatchCall[]`,
  then reconstruct or attach the host executor locally.

## Wallet and account-abstraction adapters

An EIP-5792, ERC-4337 or other wallet integration belongs behind `Executor`. Operation builders never
depend on a wallet vendor. For a batching executor:

1. Map each `BatchCall` without changing order, `to`, `data` or `value`.
2. Set `atomic: true` only when the actual wallet path guarantees all-or-nothing execution.
3. Simulate through the same account or bundler path when possible. Per-call `eth_call` does not
   substitute for smart-account validation or for state changes made by earlier calls.
4. Treat the first wallet or bundler return as an operation or bundle ID unless its API guarantees a
   transaction hash; never cast one to a transaction hash. Poll the provider-specific status, surface
   rejection or revert detail, and resolve the executor with the final on-chain hash.
5. Decode nested wallet, viem, ethers and bundler failures with `toObolosError`.

## Important entry points

| Import | Purpose |
| --- | --- |
| `@obolos/sdk` | client/config, address resolution, factories, `Prepared`, `batch`, errors |
| `@obolos/sdk/<module>` | domain reads and prepared writes |
| `@obolos/sdk/jurisdictions` | live-profile descriptions and supported instrument/grant vocabulary |
| `@obolos/sdk/compliance` | policies, rules, transfer checks, approvals and ROFR terms |
| `@obolos/sdk/hub` | credential claims, policy IDs and commitment helpers |
| `@obolos/sdk/attestation` | KYC, investor, accreditation, tax and claim vocabulary |
| `@obolos/sdk/countries` | ISO country mapping used by claims and jurisdiction profiles |
| `@obolos/sdk/fhe` | principal-bound encryption, view claims and decryption helpers |
| `@obolos/sdk/abis` | raw const ABIs for viem and indexers |
| `@obolos/sdk/types` | generated ethers contract interfaces and factories |

`package.json` exports and the generated declarations give the exact surface of the installed
version.

## Errors

Normalize every failure with `toObolosError(error)`. The stable integration surface:

- `message`: short user-facing text;
- `context`: invariant, decoded arguments and a useful next action;
- `source`: contract, rule, wallet, RPC or EVM;
- `code`: stable machine-facing identifier;
- `kind`: rejection, funds, network, revert, panic, compliance or unknown.

Never parse display strings to make program decisions. Use `code`, `kind` and named `args`.

## Repository validation

In the obolos workspace, use the package's pinned commands:

```bash
pnpm --filter @obolos/sdk typecheck
pnpm --filter @obolos/sdk test
pnpm --filter @obolos/sdk build
pnpm --filter @obolos/sdk check:examples
```

After contract ABI changes, build contracts and use the chain-specific ABI pull command the workspace
documents. Never run a default ABI pull that could rewrite an unrelated address map.

Review contract-to-SDK changes for ABI/type regeneration, address maps, error-catalogue coverage,
credit selector tables, client bindings, examples and tests. A compiling wrapper is not enough if it
exposes a path the jurisdiction or compliance layer will reject.
