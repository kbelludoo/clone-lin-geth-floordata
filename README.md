# clone-lin-geth-floordata

**Class: EXPERIMENTAL.** Scalar LIN clone of go-ethereum v1.16.9 `FloorDataGas` (EIP-7623). This is not a Geth node, not a replacement for Ethereum consensus, and not a claim that LIN executes uint64 values above `MaxGasLimit = 2^63-1`.

- **Upstream algorithm:** [ethereum/go-ethereum](https://github.com/ethereum/go-ethereum) tag `v1.16.9` commit `95665d5703e1023995a0ff93e4ce9eb77e8a59bd` `core/state_transition.go` (LGPL-3.0)
- **Constants:** `params/protocol_params.go` `TxGas=21000`, `TxTokenPerNonZeroByte=4`, `TxCostFloorPerToken=10` (token-per-zero-byte is 1)
- **Proofs and harness live in lin-open:** https://github.com/kbelludoo/lin-open (see `examples/geth_floordata/` and `python3 test/prove_geth_floordata_external.py`)

## Files

| File | What |
|---|---|
| `lin_geth_floordata.lin` | LIN kernel (`fd_floor`, `fd_tokens`, `fd_prague_charge`, explicit z/nz counts) |
| `NOTICE` | Upstream LGPL-3.0 attribution |

## Canonical vectors

| case | gas |
|---|---:|
| empty transfer | 21000 |
| 1 zero byte | 21010 |
| 1 nonzero byte | 21040 |
| 1 zero + 1 nonzero | 21050 |
| 100 zero bytes | 22000 |
| 100 nonzero bytes | 25000 |
| 500 zero + 1000 nonzero | 66000 |
| 1 nonzero Istanbul intrinsic | 21016 |
| 1 nonzero Prague charge | 21040 |

## What this clone improves versus Geth

1. `bytes.Count` / `len([]byte)` become explicit integer arguments `z` / `nz`.
2. `nz * TxTokenPerNonZeroByte` is overflow-checked. Pinned Geth v1.16.9 does not guard that multiply; uint64 wrap of `nz=2^62` returns 21000. LIN fail-closes to 0. Unreachable at `MaxBlockSize=8388608`; not a mainnet consensus bug.
3. Overflow fail-closes to 0 at i64max (the published `MaxGasLimit`) instead of `(0, error)` against `MaxUint64`.
4. Token-per-zero-byte is an explicit constant (`1`).

## Not claimed

LIN does not replace go-ethereum. Operand domain is i64 non-negative. Merkle receipts in lin-open are tamper-evidence plus re-execution, not zk. Compiler 0 does not execute `go_from_go` string-literal paths (`VM_REJ_STRING_LITERAL`).
