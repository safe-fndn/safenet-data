# safenet-data

Data and JSON files for Safenet Beta, Safenet Aegis, and potentially future Safenet versions. Contains network stats, validator info, sentinel info, and rewards distribution data.

> **Auto-updates:** `assets/network-info.json` and the `participation_rate_14d` field in `assets/validator-info.json` and `assets/sentinel-info.json` are updated automatically every 3h. All other validator and sentinel details are edited manually. Rewards data is updated manually, on a regular basis, if SafeDAO approves proposal for rewards distribution.

---

## Network Info

Aggregate network-level stats for the current Safenet version, Safenet Aegis. See [`assets/network-info.json`](./assets/network-info.json) for current data.

### Example

```json
{
	"total_staked_amount": "123123.12331221",
	"total_transactions_checked": 88764457
}
```

#### `total_staked_amount`

Total SAFE tokens currently staked, as a decimal string with up to 18 decimal places (a string so precision is preserved). Sourced from the staking contract at [`0x115E78f160e1E3eF163B05C84562Fa16fA338509`](https://etherscan.io/address/0x115E78f160e1E3eF163B05C84562Fa16fA338509#code) on Ethereum mainnet.

#### `total_transactions_checked`

A monotonically increasing integer counting the transactions checked by a Safenet version since its deployment. Counts are per version and not aggregated across versions. For Safenet Aegis, this is the number of SentinelOracle requests that at least one Sentinel revealed a verdict (approved or denied) for: requests with a `Revealed` event on the SentinelOracle [`0x4F61B8832978e83b80D69551AEf07557DBE41d03`](https://gnosisscan.io/address/0x4F61B8832978e83b80D69551AEf07557DBE41d03) on Gnosis Chain. A request with several reveals is counted once. Requests that timed out without any reveal are not counted; disputed requests are counted.

Safenet Beta counted `TransactionProposed` events on its consensus contract [`0x223624cBF099e5a8f8cD5aF22aFa424a1d1acEE9`](https://gnosisscan.io/address/0x223624cBF099e5a8f8cD5aF22aFa424a1d1acEE9) on Gnosis Chain. Its final value is kept in the git tag [`beta-final`](https://github.com/safe-fndn/safenet-data/blob/beta-final/assets/network-info.json).

---

## Validator Info

See [`assets/validator-info.json`](./assets/validator-info.json) for current data.

#### `address`

Ethereum address of the validator.

```json
"address": "0x1111111111111111111111111111111111111111"
```

#### `label`

Human-readable display name for the validator.

```json
"label": "Validator A"
```

#### `commission`

Commission percentage applied to any possible rewards.

```json
"commission": 0.05
```

#### `is_active`

`true` if the validator has issued any signing event within the last 1h, `false` otherwise.

```json
"is_active": true
```

#### `participation_rate_14d`

Average participation rate (%) over the last 14 days, across the transactions attested by the Safenet Aegis consensus contract [`0xc855761D619f6002923507cE68B84d7689C2aa96`](https://gnosisscan.io/address/0xc855761D619f6002923507cE68B84d7689C2aa96) on Gnosis Chain. Computed by joining `SignCompleted` events with `SignShared` on `selection_root` and measuring the validator's participation rate across those events.

```json
"participation_rate_14d": 0.8523
```

---

## Sentinel Info

See [`assets/sentinel-info.json`](./assets/sentinel-info.json) for current data.

#### `address`

Ethereum address of the sentinel.

```json
"address": "0x1111111111111111111111111111111111111111"
```

#### `label`

Human-readable display name for the sentinel.

```json
"label": "Sentinel A"
```

#### `is_active`

`true` if the sentinel is currently active, `false` otherwise. Edited manually.

```json
"is_active": true
```

#### `participation_rate_14d`

Average participation rate (%) over the last 14 days, across the requests made to the Safenet Aegis SentinelOracle [`0x4F61B8832978e83b80D69551AEf07557DBE41d03`](https://gnosisscan.io/address/0x4F61B8832978e83b80D69551AEf07557DBE41d03) on Gnosis Chain. Computed as the share of `NewRequest` events in the period that the sentinel emitted a `Revealed` event for. A reveal counts towards the period of its request, not the period it was revealed in.

```json
"participation_rate_14d": 0.8523
```

---

## rewards

Merkle proofs and associated data required for Safenet Aegis rewards distribution, mirrors the files corresponding to the onchain `contenthash` record for `rewards.safenet-gov.eth`. This is provided as a convenience, the onchain record is the source of truth.

The [distribution workflow](.github/workflows/distribution.yml) runs daily at 04:23 UTC and can also be started manually. It verifies the published CAR against ENS and the onchain payout Merkle root, extracts it with IPFS, and commits changes to `assets/rewards/` on `main` using the data bot's SSH key. A proposed distribution is mirrored after execution advances the published database.

When the next rewards period has settled, the workflow exports and independently verifies it, then uploads a `distribution-<periodEnd>-<runId>-<attempt>` artifact containing the CAR, proposal, Safe transaction bundles, checksums and summaries. Until execution, later runs can produce another artifact for the same period. The manual `verifyFromGenesis` option skips the database cache for verification. The workflow creates no GitHub releases and sends no onchain transactions; see the staking scripts' [operator runbook](https://github.com/safe-fndn/safenet-staking-scripts/blob/main/docs/permissionless-distribution.md) for reviewing, proposing and executing the bundle.
