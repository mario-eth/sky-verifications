# Grove Basin v1.0.0 deployment verification

Address source: [iamchrissmith Grove Basin v1.0.0 deployed contracts gist](https://gist.github.com/iamchrissmith/7cc487983bdf0c00c1de6506047ab130)

Chain: Ethereum mainnet (`chain_id` 1)

Audited source commit: [`grove-labs/grove-basin@9c812fcb32df0475ceaf443e3db39c3302a5e56c`](https://github.com/grove-labs/grove-basin/commit/9c812fcb32df0475ceaf443e3db39c3302a5e56c)

Deployer: `0x6D99f476E7E9FCcd189fb87023cFa301364Fa817`

## Summary

| Item | Count |
|---|---:|
| Gist deployment addresses checked | 12 |
| Core factories/providers checked | 4 |
| GroveBasin instances checked | 2 |
| UsdsUsdcPocket instances checked | 2 |
| Token redeemers checked | 2 |
| Timelocks checked | 2 |
| Fork deployment verifier | `test/fork/VerifyDeployment.sol` |
| Expected wiring mismatches | 0 |
| Expected stale/old roles | 0 |
| Etherscan-verified source | 12 of 12 gist addresses |
| Open timelock operations | 1 BUIDL pending no-op |

Last live read-only run: Alchemy mainnet RPC on June 16, 2026, block `25331319`, `146` RPC checks passed.

## Deployed Contracts

| Contract | Address | Expected role |
|---|---|---|
| `GroveBasinFactory` | `0x78Dc98D689Fe9A1b0056ac1cDFC14722bDA6D49a` | deploys seeded basins |
| `FixedRateProvider` (USDS/USDC 1:1) | `0x7928A185B8137D1CD2a0996a810A04dB2837419D` | USDS and USDC rate provider |
| BUIDL `ChronicleRateProvider` | `0x69a171853575FFD41574EA80Abfc6337AcbC4d43` | BUIDL credit-token rate provider |
| JTRSY `ChronicleRateProvider` | `0x29209ceCFeFa6f675E6f1f829320D67cE2b025E5` | JTRSY credit-token rate provider |
| JTRSY admin timelock | `0xA52dC9876aB4A9DB6dAfbb83410554086054d140` | owner/admin timelock for JTRSY basin |
| JTRSY `GroveBasin` | `0xf08943f817e1F902dEbC884c7B19Ea5764594Ac9` | JTRSY USDS/USDC basin |
| JTRSY `UsdsUsdcPocket` | `0x2Cd296095788A2741e72056D66B3Ae1fAeE23ea2` | JTRSY basin pocket |
| JTRSY `JTRSYTokenRedeemer` | `0x7c5Ce1a1D50a6cb3Da97C9e202B3E7CD8e5b5b6c` | JTRSY token redeemer |
| BUIDL admin timelock | `0xdB8C7c814E9780659B23478EF4Bda9032CC9Ff34` | owner/admin timelock for BUIDL basin |
| BUIDL `GroveBasin` | `0xCBa428fB052B365557DAf52b744DFfF20d5FbEdD` | BUIDL USDS/USDC basin |
| BUIDL `UsdsUsdcPocket` | `0x39548FeF138370Db06e172eF0739894b2a613DF9` | BUIDL basin pocket |
| BUIDL `BUIDLTokenRedeemer` | `0x73414528187A4986E2Af5D551fD14871b723E506` | BUIDL token redeemer |

## External Addresses

| Name | Address |
|---|---|
| `DPAU_ALM_PROXY` | `0x0DcD9298e163dFD3c0B5b00F0d9093C36e40A153` |
| `GROVE_PROXY` | `0x1369f7b2b38c76B6478c0f0E66D94923421891Ba` |
| `ALM_RELAYER` | `0x0eEC86649E756a23CBc68d9EFEd756f16aD5F85f` |
| `ALM_FREEZER` | `0xB0113804960345fd0a245788b3423319c86940e5` |
| `USDS` | `0xdC035D45d973E3EC169d2276DDab16f1e407384F` |
| `USDC` | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` |
| `USDS_PSM_WRAPPER` | `0xA188EEC8F81263234dA3622A406892F3D630f98c` |
| `JTRSY_TOKEN` | `0x8c213ee79581Ff4984583C6a801e5263418C4b86` |
| `CENTRIFUGE_JTRSY` | `0xFE6920eB6C421f1179cA8c8d4170530CDBdfd77A` |
| `JTRSY_ISSUER_MULTISIG` | `0x9184DdBCc4824B76CE2AEFA72534a1a87aA5037c` |
| `JTRSY_REDEEMER` | `0xb6e8D3E47c4FC5606E6C24D097Dd1791885Ce05a` |
| `BUIDL_TOKEN` | `0x7712c34205737192402172409a8F7ccef8aA2AEc` |
| `BUIDL_ISSUER_MULTISIG` | `0x453A28B31fdc31858C35B02bc3A42BCD8bfbAd3a` |
| `BUIDL_REDEEMER` | `0x488F27168a19472c51f003fbC5b75B1ACc3B7b4c` |
| `BUIDL_REDEMPTION_ADDRESS` | `0x8780Dd016171B91E4Df47075dA0a947959C34200` |
| `BUIDL_CHRONICLE_ORACLE` | `0x8c68E0CacB61a065b99E2104457aCC829d61cbB0` |
| `JTRSY_CHRONICLE_ORACLE` | `0xE980a33EFA3EDDaa689eCbdCE4B2278D4DB94471` |

## Method

| Scope | Check |
|---|---|
| Address source | The 12 deployed contract addresses match the gist table |
| Bytecode presence | Every listed deployed contract has non-empty code on Ethereum mainnet |
| Rate providers | Fixed provider returns `1e27`; Chronicle providers point at expected BUIDL/JTRSY oracles and expose `1e27` precision |
| Basins | Tokens, rate providers, pocket, liquidity provider, fee bounds, staleness defaults, and pause flags match expected setup |
| Pockets | `basin`, `usds`, `usdc`, PSM wrapper, Grove Proxy, and USDS allowance are wired correctly |
| Redeemers | Redeemer `basin`, `creditToken`, and vault/redemption address match expected deployment |
| Timelocks | Delay, proposer, executor, canceller, and admin roles match expected clean deployed state |
| Clean state | No deployer, old issuer, old redeemer, or old token redeemer roles remain in the live deployment |

## Source Verification Status

Source: Etherscan `getsourcecode`. Checked on June 16, 2026.

| Contract | Address | Etherscan source status | Contract name |
|---|---|---|---|
| `GroveBasinFactory` | `0x78Dc98D689Fe9A1b0056ac1cDFC14722bDA6D49a` | verified | `GroveBasinFactory` |
| `FixedRateProvider` | `0x7928A185B8137D1CD2a0996a810A04dB2837419D` | verified | `FixedRateProvider` |
| BUIDL `ChronicleRateProvider` | `0x69a171853575FFD41574EA80Abfc6337AcbC4d43` | verified | `ChronicleRateProvider` |
| JTRSY `ChronicleRateProvider` | `0x29209ceCFeFa6f675E6f1f829320D67cE2b025E5` | verified | `ChronicleRateProvider` |
| JTRSY admin timelock | `0xA52dC9876aB4A9DB6dAfbb83410554086054d140` | verified | `TimelockController` |
| JTRSY `GroveBasin` | `0xf08943f817e1F902dEbC884c7B19Ea5764594Ac9` | verified | `GroveBasin` |
| JTRSY `UsdsUsdcPocket` | `0x2Cd296095788A2741e72056D66B3Ae1fAeE23ea2` | verified | `UsdsUsdcPocket` |
| JTRSY `JTRSYTokenRedeemer` | `0x7c5Ce1a1D50a6cb3Da97C9e202B3E7CD8e5b5b6c` | verified | `JTRSYTokenRedeemer` |
| BUIDL admin timelock | `0xdB8C7c814E9780659B23478EF4Bda9032CC9Ff34` | verified | `TimelockController` |
| BUIDL `GroveBasin` | `0xCBa428fB052B365557DAf52b744DFfF20d5FbEdD` | verified | `GroveBasin` |
| BUIDL `UsdsUsdcPocket` | `0x39548FeF138370Db06e172eF0739894b2a613DF9` | verified | `UsdsUsdcPocket` |
| BUIDL `BUIDLTokenRedeemer` | `0x73414528187A4986E2Af5D551fD14871b723E506` | verified | `BUIDLTokenRedeemer` |

All 12 gist addresses currently have verified source. The two final basin deployments are also provenance-linked to the verified `GroveBasinFactory` through the factory calls listed below.

## Factory Deployment Provenance

The final JTRSY/BUIDL basin deployments were created by the verified `GroveBasinFactory` via `deploy(address,address,address,address,address,address,address,address)`.

| Basin | Factory tx | Owner argument | Liquidity provider argument | Swap token | Collateral token | Credit token | Swap/collateral rate provider | Credit rate provider |
|---|---|---|---|---|---|---|---|---|
| JTRSY | `0x17e4e472d6a5874fd057f7e34c2e3cb8d29fa5544b373ef371e46ff6ce6332da` | `DEPLOYER` | `DPAU_ALM_PROXY` | `USDS` | `USDC` | `JTRSY_TOKEN` | `FixedRateProvider` | JTRSY `ChronicleRateProvider` |
| BUIDL | `0x990afcd93b55b0fe5463bb91d0dc66dcfad685f3c47b393cedfbceae20e82340` | `DEPLOYER` | `DPAU_ALM_PROXY` | `USDS` | `USDC` | `BUIDL_TOKEN` | `FixedRateProvider` | BUIDL `ChronicleRateProvider` |

## Transaction History Snapshot

Source: Etherscan account `txlist`, `txlistinternal`, `tokentx`, and timelock event logs. Checked on June 16, 2026 at mainnet block `25331319`.

| Contract | Normal txs | Internal txs | ERC20 transfers | Notes |
|---|---:|---:|---:|---|
| `GroveBasinFactory` | 7 | 6 | 12 | deploy plus six seed calls, including final JTRSY/BUIDL basin seeds |
| `FixedRateProvider` | 1 | 0 | 0 | deploy only |
| BUIDL `ChronicleRateProvider` | 1 | 0 | 0 | deploy only |
| JTRSY `ChronicleRateProvider` | 1 | 0 | 0 | deploy only |
| JTRSY admin timelock | 3 | 0 | 0 | deploy, grant canceller, revoke deployer admin |
| JTRSY `GroveBasin` | 17 | 1 | 2 | setup txs plus final deployer role renounce |
| JTRSY `UsdsUsdcPocket` | 1 | 0 | 1 | deploy plus seeded USDS transfer |
| JTRSY `JTRSYTokenRedeemer` | 1 | 0 | 0 | deploy only |
| BUIDL admin timelock | 12 | 0 | 0 | deploy/setup plus one pending scheduled no-op |
| BUIDL `GroveBasin` | 17 | 1 | 2 | setup txs plus final deployer role renounce |
| BUIDL `UsdsUsdcPocket` | 1 | 0 | 1 | deploy plus seeded USDS transfer |
| BUIDL `BUIDLTokenRedeemer` | 1 | 0 | 0 | deploy only |

Timelock event check:

| Timelock | `CallScheduled` | `CallExecuted` | `Cancelled` | Notes |
|---|---:|---:|---:|---|
| BUIDL admin timelock | 1 | 0 | 0 | operation `0x13bdfcbacee698fdfbc8de83e0ce86a238900e77348b0fa964df0efce295892e` is pending and ready, target `BUIDL_ISSUER_MULTISIG`, value `0`, data `0x`, predecessor `0x0`, salt `0x0`, delay `604800` |
| JTRSY admin timelock | 0 | 0 | 0 | no scheduled/executed/cancelled operations |

The BUIDL pending operation has no calldata and sends no ETH. It is not an immediate malicious payload, but it is not a perfectly clean timelock queue. If it was not intentional, it should be cancelled by an authorized canceller.

## Client Checklist Reconciliation

The client deployment checklist was used as an input to reconcile against chain data, not as trusted evidence by itself.

| Checklist item | Independent verification result |
|---|---|
| Commit `9c812fcb32df0475ceaf443e3db39c3302a5e56c` | Matches the audited source commit recorded above. |
| Deployer `0x6D99f476E7E9FCcd189fb87023cFa301364Fa817` | Matches the deployer in the listed mainnet deployment transactions. |
| BUIDL `GroveBasin` tx `0x990afcd93b55b0fe5463bb91d0dc66dcfad685f3c47b393cedfbceae20e82340` | Confirmed as the final BUIDL factory deployment transaction. |
| BUIDL `UsdsUsdcPocket` tx `0x77f6e80c7819d585ffb2e1f4bd8172edf19c89b12ca0b3ac0f02ec69f8d6c05f` | Confirmed as the live BUIDL pocket deployment. |
| BUIDL `BUIDLTokenRedeemer` tx `0x888266d7097dbd63221f7087e33dad9abf6b11c31bd8ce5799b5993b12bc5a99` | Confirmed as the live BUIDL token redeemer deployment. |
| BUIDL `addTokenRedeemer` tx `0x4f6f5e737ae0fd4c7900a6bce8f7e884f476973bcbfa3b6d86b4c70d841e7b61` | Confirmed. Live role checks also confirm only the expected new BUIDL token redeemer has `REDEEMER_CONTRACT_ROLE`. |
| BUIDL `grantRole(REDEEMER_ROLE)` tx `0x35887af49a71944ccb14cdc156bdc92a94d0aa2c3f72b76c987cfc9c8fc3d1a9` | Confirmed. Live role checks confirm the current BUIDL issuer redeemer has `REDEEMER_ROLE`. |
| BUIDL blocks `25324010-25324038`, 20 transactions | Confirmed from the deployer transaction history for the deployment window; all listed BUIDL deployment/configuration transactions are present. |
| JTRSY `GroveBasin` tx `0x17e4e472d6a5874fd057f7e34c2e3cb8d29fa5544b373ef371e46ff6ce6332da` | Confirmed as the final JTRSY factory deployment transaction. |
| JTRSY `UsdsUsdcPocket` tx `0xa7d03ed1f6ec5f718858a1c5c53cbf693604b7050cba4692de150a87b86a42a0` | Confirmed as the live JTRSY pocket deployment. |
| JTRSY `JTRSYTokenRedeemer` tx `0x17bf9bd8e28c8ca2ffeb39baaada6615c75c749a1bfaf2596172df531e6289af` | Confirmed as the live JTRSY token redeemer deployment. |
| Tenderly private-testnet transactions | Not independently verified in this document. They are client-provided process evidence, not mainnet wiring evidence. |
| Foundry version, fresh clone, key handling, no private key in env/history, independent team review | Not independently provable from chain state. Treat as client process claims only. |

The BUIDL checklist shows a separate helper command for `deployRedeemerContractAndGrantRedeemerRole(address)`. The live chain state is clean under the expected final policy: the BUIDL basin points at the expected token/redeemer setup, the new BUIDL token redeemer has the expected role, and the old Securitize/BUIDL redeemer roles are absent. The separate helper command is therefore not a wiring bug by itself; the final-state role checks are the source of truth.

## One-Stop Verification

Prerequisites: `forge`, `cast`, Ethereum mainnet RPC.

Run from `grove-basin/`:

```bash
MAINNET_RPC_URL=<mainnet-rpc> forge test --match-path test/fork/VerifyDeployment.sol -q
```

This fork test is the canonical live deployment wiring test in this repo. It checks:

| Area | Coverage |
|---|---|
| Timelocks | self-admin, 7 day delay, issuer proposer, Grove Proxy executor, issuer and freezer cancellers, no deployer roles |
| Basins | owner timelock, Grove Proxy manager-admin, ALM relayer manager, ALM freezer pauser, DPAU ALM liquidity provider |
| Fees and pause flags | fee claimer zero, min/max fees, paused credit-in directions, unpaused credit-out directions |
| Pockets | pocket address, basin link, USDS/USDC/PSM wiring, Grove Proxy allowance |
| Redeemers | token redeemer address, basin link, credit token, vault/redemption address |
| Clean old state | old Securitize issuer, old redeemer, and old BUIDL token redeemer have no relevant roles |
| Actions | timelock schedule/execute/cancel flow, role revocation, freezer flow, deposits, withdraws, swaps, sweeps |

## Additional Cast Smoke Checks

Use this when you want a quick read-only check without running Foundry tests.

```bash
set -euo pipefail

RPC="${MAINNET_RPC_URL:-${ETH_RPC_URL:-}}"
if [ -z "$RPC" ]; then
  echo "ERROR: set MAINNET_RPC_URL or ETH_RPC_URL" >&2
  exit 2
fi

ZERO=0x0000000000000000000000000000000000000000
MAX_UINT=115792089237316195423570985008687907853269984665640564039457584007913129639935

DEPLOYER=0x6D99f476E7E9FCcd189fb87023cFa301364Fa817
DPAU_ALM_PROXY=0x0DcD9298e163dFD3c0B5b00F0d9093C36e40A153
GROVE_PROXY=0x1369f7b2b38c76B6478c0f0E66D94923421891Ba
ALM_RELAYER=0x0eEC86649E756a23CBc68d9EFEd756f16aD5F85f
ALM_FREEZER=0xB0113804960345fd0a245788b3423319c86940e5

USDS=0xdC035D45d973E3EC169d2276DDab16f1e407384F
USDC=0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48
USDS_PSM_WRAPPER=0xA188EEC8F81263234dA3622A406892F3D630f98c

FACTORY=0x78Dc98D689Fe9A1b0056ac1cDFC14722bDA6D49a
FIXED_RATE_PROVIDER=0x7928A185B8137D1CD2a0996a810A04dB2837419D

BUIDL_ORACLE=0x8c68E0CacB61a065b99E2104457aCC829d61cbB0
BUIDL_RATE_PROVIDER=0x69a171853575FFD41574EA80Abfc6337AcbC4d43
BUIDL_TIMELOCK=0xdB8C7c814E9780659B23478EF4Bda9032CC9Ff34
BUIDL_BASIN=0xCBa428fB052B365557DAf52b744DFfF20d5FbEdD
BUIDL_POCKET=0x39548FeF138370Db06e172eF0739894b2a613DF9
BUIDL_REDEEMER_CONTRACT=0x73414528187A4986E2Af5D551fD14871b723E506
BUIDL_TOKEN=0x7712c34205737192402172409a8F7ccef8aA2AEc
BUIDL_ISSUER_MULTISIG=0x453A28B31fdc31858C35B02bc3A42BCD8bfbAd3a
BUIDL_REDEEMER=0x488F27168a19472c51f003fbC5b75B1ACc3B7b4c
BUIDL_REDEMPTION_ADDRESS=0x8780Dd016171B91E4Df47075dA0a947959C34200
OLD_SECURITIZE_ISSUER_MULTISIG=0x551e841e6fb54431a0664C8776784F6d7E611428
OLD_SECURITIZE_REDEEMER=0xdfC603076EA75895DD4d59c6e2ee5038f881CB74
OLD_BUIDL_TOKEN_REDEEMER=0x0D46f8A832B76A79AC3B5F29fFfc35ACeebad885

JTRSY_ORACLE=0xE980a33EFA3EDDaa689eCbdCE4B2278D4DB94471
JTRSY_RATE_PROVIDER=0x29209ceCFeFa6f675E6f1f829320D67cE2b025E5
JTRSY_TIMELOCK=0xA52dC9876aB4A9DB6dAfbb83410554086054d140
JTRSY_BASIN=0xf08943f817e1F902dEbC884c7B19Ea5764594Ac9
JTRSY_POCKET=0x2Cd296095788A2741e72056D66B3Ae1fAeE23ea2
JTRSY_REDEEMER_CONTRACT=0x7c5Ce1a1D50a6cb3Da97C9e202B3E7CD8e5b5b6c
JTRSY_TOKEN=0x8c213ee79581Ff4984583C6a801e5263418C4b86
CENTRIFUGE_JTRSY=0xFE6920eB6C421f1179cA8c8d4170530CDBdfd77A
JTRSY_ISSUER_MULTISIG=0x9184DdBCc4824B76CE2AEFA72534a1a87aA5037c
JTRSY_REDEEMER=0xb6e8D3E47c4FC5606E6C24D097Dd1791885Ce05a

call() {
  cast call "$1" "$2" "${@:3}" --rpc-url "$RPC"
}

fail() {
  echo "FAIL: $*" >&2
  exit 1
}

pass() {
  echo "pass: $*"
}

expect_eq() {
  local label="$1"
  local actual="$2"
  local expected="$3"
  if [ "$actual" != "$expected" ]; then
    fail "$label expected $expected got $actual"
  fi
  pass "$label = $expected"
}

expect_addr() {
  local label="$1"
  local actual="$2"
  local expected="$3"
  expect_eq "$label" "$(cast to-check-sum-address "$actual")" "$(cast to-check-sum-address "$expected")"
}

expect_bool() {
  local label="$1"
  local actual="$2"
  local expected="$3"
  expect_eq "$label" "$actual" "$expected"
}

expect_code() {
  local name="$1"
  local addr="$2"
  local code
  code="$(cast code "$addr" --rpc-url "$RPC")"
  if [ "$code" = "0x" ]; then
    fail "$name has no code at $addr"
  fi
  pass "$name code present"
}

expect_has_role() {
  local label="$1"
  local target="$2"
  local role="$3"
  local account="$4"
  local expected="$5"
  expect_bool "$label" "$(call "$target" 'hasRole(bytes32,address)(bool)' "$role" "$account")" "$expected"
}

verify_timelock() {
  local name="$1"
  local tl="$2"
  local issuer="$3"
  local proposer_role executor_role canceller_role
  proposer_role="$(call "$tl" 'PROPOSER_ROLE()(bytes32)')"
  executor_role="$(call "$tl" 'EXECUTOR_ROLE()(bytes32)')"
  canceller_role="$(call "$tl" 'CANCELLER_ROLE()(bytes32)')"

  expect_eq "$name min delay" "$(call "$tl" 'getMinDelay()(uint256)')" "604800"
  expect_has_role "$name self default admin" "$tl" 0x0000000000000000000000000000000000000000000000000000000000000000 "$tl" true
  expect_has_role "$name deployer default admin" "$tl" 0x0000000000000000000000000000000000000000000000000000000000000000 "$DEPLOYER" false
  expect_has_role "$name issuer proposer" "$tl" "$proposer_role" "$issuer" true
  expect_has_role "$name Grove Proxy executor" "$tl" "$executor_role" "$GROVE_PROXY" true
  expect_has_role "$name open executor disabled" "$tl" "$executor_role" "$ZERO" false
  expect_has_role "$name issuer canceller" "$tl" "$canceller_role" "$issuer" true
  expect_has_role "$name ALM Freezer canceller" "$tl" "$canceller_role" "$ALM_FREEZER" true
}

verify_basin_common() {
  local name="$1"
  local basin="$2"
  local timelock="$3"
  local pocket="$4"
  local credit_token="$5"
  local credit_provider="$6"
  local redeemer_contract="$7"
  local issuer_redeemer="$8"
  local owner_role manager_admin_role manager_role pauser_role redeemer_role redeemer_contract_role

  owner_role="$(call "$basin" 'OWNER_ROLE()(bytes32)')"
  manager_admin_role="$(call "$basin" 'MANAGER_ADMIN_ROLE()(bytes32)')"
  manager_role="$(call "$basin" 'MANAGER_ROLE()(bytes32)')"
  pauser_role="$(call "$basin" 'PAUSER_ROLE()(bytes32)')"
  redeemer_role="$(call "$basin" 'REDEEMER_ROLE()(bytes32)')"
  redeemer_contract_role="$(call "$basin" 'REDEEMER_CONTRACT_ROLE()(bytes32)')"

  expect_addr "$name liquidity provider" "$(call "$basin" 'liquidityProvider()(address)')" "$DPAU_ALM_PROXY"
  expect_addr "$name swap token" "$(call "$basin" 'swapToken()(address)')" "$USDS"
  expect_addr "$name collateral token" "$(call "$basin" 'collateralToken()(address)')" "$USDC"
  expect_addr "$name credit token" "$(call "$basin" 'creditToken()(address)')" "$credit_token"
  expect_addr "$name pocket" "$(call "$basin" 'pocket()(address)')" "$pocket"
  expect_addr "$name swap rate provider" "$(call "$basin" 'swapTokenRateProvider()(address)')" "$FIXED_RATE_PROVIDER"
  expect_addr "$name collateral rate provider" "$(call "$basin" 'collateralTokenRateProvider()(address)')" "$FIXED_RATE_PROVIDER"
  expect_addr "$name credit rate provider" "$(call "$basin" 'creditTokenRateProvider()(address)')" "$credit_provider"
  expect_eq "$name max fee" "$(call "$basin" 'maxFee()(uint256)')" "500"
  expect_eq "$name fee claimer" "$(call "$basin" 'feeClaimer()(address)')" "$ZERO"
  expect_eq "$name staleness threshold" "$(call "$basin" 'stalenessThreshold()(uint256)')" "604800"
  expect_has_role "$name timelock owner role" "$basin" "$owner_role" "$timelock" true
  expect_has_role "$name deployer owner role" "$basin" "$owner_role" "$DEPLOYER" false
  expect_has_role "$name Grove Proxy manager admin" "$basin" "$manager_admin_role" "$GROVE_PROXY" true
  expect_has_role "$name deployer manager admin" "$basin" "$manager_admin_role" "$DEPLOYER" false
  expect_has_role "$name ALM Relayer manager" "$basin" "$manager_role" "$ALM_RELAYER" true
  expect_has_role "$name ALM Freezer pauser" "$basin" "$pauser_role" "$ALM_FREEZER" true
  expect_has_role "$name issuer redeemer" "$basin" "$redeemer_role" "$issuer_redeemer" true
  expect_has_role "$name redeemer contract" "$basin" "$redeemer_contract_role" "$redeemer_contract" true
}

verify_pocket() {
  local name="$1"
  local pocket="$2"
  local basin="$3"
  expect_addr "$name pocket basin" "$(call "$pocket" 'basin()(address)')" "$basin"
  expect_addr "$name pocket USDS" "$(call "$pocket" 'usds()(address)')" "$USDS"
  expect_addr "$name pocket USDC" "$(call "$pocket" 'usdc()(address)')" "$USDC"
  expect_addr "$name pocket PSM" "$(call "$pocket" 'psm()(address)')" "$USDS_PSM_WRAPPER"
  expect_addr "$name pocket Grove Proxy" "$(call "$pocket" 'groveProxy()(address)')" "$GROVE_PROXY"
  expect_eq "$name Grove Proxy USDS allowance" "$(call "$USDS" 'allowance(address,address)(uint256)' "$pocket" "$GROVE_PROXY")" "$MAX_UINT"
}

verify_redeemer() {
  local name="$1"
  local redeemer="$2"
  local basin="$3"
  local credit_token="$4"
  local vault_or_redemption="$5"
  expect_addr "$name redeemer basin" "$(call "$redeemer" 'basin()(address)')" "$basin"
  expect_addr "$name redeemer credit token" "$(call "$redeemer" 'creditToken()(address)')" "$credit_token"
  expect_addr "$name redeemer vault/redemption" "$(call "$redeemer" 'vault()(address)')" "$vault_or_redemption"
}

for item in \
  "$FACTORY:GroveBasinFactory" \
  "$FIXED_RATE_PROVIDER:FixedRateProvider" \
  "$BUIDL_RATE_PROVIDER:BUIDL ChronicleRateProvider" \
  "$JTRSY_RATE_PROVIDER:JTRSY ChronicleRateProvider" \
  "$BUIDL_TIMELOCK:BUIDL Timelock" \
  "$JTRSY_TIMELOCK:JTRSY Timelock" \
  "$BUIDL_BASIN:BUIDL GroveBasin" \
  "$JTRSY_BASIN:JTRSY GroveBasin" \
  "$BUIDL_POCKET:BUIDL UsdsUsdcPocket" \
  "$JTRSY_POCKET:JTRSY UsdsUsdcPocket" \
  "$BUIDL_REDEEMER_CONTRACT:BUIDLTokenRedeemer" \
  "$JTRSY_REDEEMER_CONTRACT:JTRSYTokenRedeemer"
do
  addr="${item%%:*}"
  name="${item#*:}"
  expect_code "$name" "$addr"
done

expect_eq "fixed rate" "$(call "$FIXED_RATE_PROVIDER" 'rate()(uint256)')" "1000000000000000000000000000"
expect_eq "fixed rate precision" "$(call "$FIXED_RATE_PROVIDER" 'getRatePrecision()(uint256)')" "1000000000000000000000000000"
expect_addr "BUIDL Chronicle oracle" "$(call "$BUIDL_RATE_PROVIDER" 'oracle()(address)')" "$BUIDL_ORACLE"
expect_addr "JTRSY Chronicle oracle" "$(call "$JTRSY_RATE_PROVIDER" 'oracle()(address)')" "$JTRSY_ORACLE"
expect_eq "BUIDL Chronicle precision" "$(call "$BUIDL_RATE_PROVIDER" 'getRatePrecision()(uint256)')" "1000000000000000000000000000"
expect_eq "JTRSY Chronicle precision" "$(call "$JTRSY_RATE_PROVIDER" 'getRatePrecision()(uint256)')" "1000000000000000000000000000"

verify_timelock "BUIDL" "$BUIDL_TIMELOCK" "$BUIDL_ISSUER_MULTISIG"
verify_timelock "JTRSY" "$JTRSY_TIMELOCK" "$JTRSY_ISSUER_MULTISIG"

verify_basin_common "BUIDL" "$BUIDL_BASIN" "$BUIDL_TIMELOCK" "$BUIDL_POCKET" "$BUIDL_TOKEN" "$BUIDL_RATE_PROVIDER" "$BUIDL_REDEEMER_CONTRACT" "$BUIDL_REDEEMER"
verify_basin_common "JTRSY" "$JTRSY_BASIN" "$JTRSY_TIMELOCK" "$JTRSY_POCKET" "$JTRSY_TOKEN" "$JTRSY_RATE_PROVIDER" "$JTRSY_REDEEMER_CONTRACT" "$JTRSY_REDEEMER"

verify_pocket "BUIDL" "$BUIDL_POCKET" "$BUIDL_BASIN"
verify_pocket "JTRSY" "$JTRSY_POCKET" "$JTRSY_BASIN"

verify_redeemer "BUIDL" "$BUIDL_REDEEMER_CONTRACT" "$BUIDL_BASIN" "$BUIDL_TOKEN" "$BUIDL_REDEMPTION_ADDRESS"
verify_redeemer "JTRSY" "$JTRSY_REDEEMER_CONTRACT" "$JTRSY_BASIN" "$JTRSY_TOKEN" "$CENTRIFUGE_JTRSY"

buidl_redeemer_role="$(call "$BUIDL_BASIN" 'REDEEMER_ROLE()(bytes32)')"
buidl_redeemer_contract_role="$(call "$BUIDL_BASIN" 'REDEEMER_CONTRACT_ROLE()(bytes32)')"
buidl_timelock_proposer_role="$(call "$BUIDL_TIMELOCK" 'PROPOSER_ROLE()(bytes32)')"
buidl_timelock_executor_role="$(call "$BUIDL_TIMELOCK" 'EXECUTOR_ROLE()(bytes32)')"
buidl_timelock_canceller_role="$(call "$BUIDL_TIMELOCK" 'CANCELLER_ROLE()(bytes32)')"

expect_has_role "old BUIDL issuer proposer" "$BUIDL_TIMELOCK" "$buidl_timelock_proposer_role" "$OLD_SECURITIZE_ISSUER_MULTISIG" false
expect_has_role "old BUIDL issuer executor" "$BUIDL_TIMELOCK" "$buidl_timelock_executor_role" "$OLD_SECURITIZE_ISSUER_MULTISIG" false
expect_has_role "old BUIDL issuer canceller" "$BUIDL_TIMELOCK" "$buidl_timelock_canceller_role" "$OLD_SECURITIZE_ISSUER_MULTISIG" false
expect_has_role "old BUIDL redeemer role" "$BUIDL_BASIN" "$buidl_redeemer_role" "$OLD_SECURITIZE_REDEEMER" false
expect_has_role "old BUIDL token redeemer role" "$BUIDL_BASIN" "$buidl_redeemer_contract_role" "$OLD_BUIDL_TOKEN_REDEEMER" false

echo
echo "PASS: Grove Basin gist addresses are wired and clean under the expected live-deployment policy"
```

## Recheck Source and Activity

Use this to refresh Etherscan source status and transaction counts. It intentionally prints counts only, not full transaction payloads.

```bash
set -euo pipefail

: "${ETHERSCAN_API_KEY:?set ETHERSCAN_API_KEY}"

ADDRS='
GroveBasinFactory|0x78Dc98D689Fe9A1b0056ac1cDFC14722bDA6D49a
FixedRateProvider|0x7928A185B8137D1CD2a0996a810A04dB2837419D
BUIDL ChronicleRateProvider|0x69a171853575FFD41574EA80Abfc6337AcbC4d43
JTRSY ChronicleRateProvider|0x29209ceCFeFa6f675E6f1f829320D67cE2b025E5
JTRSY Timelock|0xA52dC9876aB4A9DB6dAfbb83410554086054d140
JTRSY GroveBasin|0xf08943f817e1F902dEbC884c7B19Ea5764594Ac9
JTRSY Pocket|0x2Cd296095788A2741e72056D66B3Ae1fAeE23ea2
JTRSY TokenRedeemer|0x7c5Ce1a1D50a6cb3Da97C9e202B3E7CD8e5b5b6c
BUIDL Timelock|0xdB8C7c814E9780659B23478EF4Bda9032CC9Ff34
BUIDL GroveBasin|0xCBa428fB052B365557DAf52b744DFfF20d5FbEdD
BUIDL Pocket|0x39548FeF138370Db06e172eF0739894b2a613DF9
BUIDL TokenRedeemer|0x73414528187A4986E2Af5D551fD14871b723E506
'

api() {
  local query="$1"
  curl -fsS "https://api.etherscan.io/v2/api?chainid=1&${query}&apikey=${ETHERSCAN_API_KEY}"
}

printf '%s\n' "$ADDRS" | sed '/^$/d' | while IFS='|' read -r name addr; do
  source_status="$(api "module=contract&action=getsourcecode&address=${addr}" | jq -r '.result[0] | if (.SourceCode != "" and .ABI != "Contract source code not verified") then "verified" else "not-verified" end')"
  normal="$(api "module=account&action=txlist&address=${addr}&startblock=0&endblock=99999999&page=1&offset=10000&sort=asc" | jq -r 'if .status == "1" then (.result | length) else 0 end')"
  internal="$(api "module=account&action=txlistinternal&address=${addr}&startblock=0&endblock=99999999&page=1&offset=10000&sort=asc" | jq -r 'if .status == "1" then (.result | length) else 0 end')"
  erc20="$(api "module=account&action=tokentx&address=${addr}&startblock=0&endblock=99999999&page=1&offset=10000&sort=asc" | jq -r 'if .status == "1" then (.result | length) else 0 end')"
  printf '%s | source=%s normal=%s internal=%s erc20=%s\n' "$name" "$source_status" "$normal" "$internal" "$erc20"
done
```

Use this to refresh the currently known BUIDL timelock queued no-op.

```bash
set -euo pipefail

RPC="${MAINNET_RPC_URL:-${ETH_RPC_URL:-}}"
if [ -z "$RPC" ]; then
  echo "ERROR: set MAINNET_RPC_URL or ETH_RPC_URL" >&2
  exit 2
fi

BUIDL_TIMELOCK=0xdB8C7c814E9780659B23478EF4Bda9032CC9Ff34
BUIDL_PENDING_NOOP_OPERATION=0x13bdfcbacee698fdfbc8de83e0ce86a238900e77348b0fa964df0efce295892e

cast call "$BUIDL_TIMELOCK" 'getOperationState(bytes32)(uint8)' "$BUIDL_PENDING_NOOP_OPERATION" --rpc-url "$RPC"
cast call "$BUIDL_TIMELOCK" 'isOperationPending(bytes32)(bool)' "$BUIDL_PENDING_NOOP_OPERATION" --rpc-url "$RPC"
cast call "$BUIDL_TIMELOCK" 'isOperationReady(bytes32)(bool)' "$BUIDL_PENDING_NOOP_OPERATION" --rpc-url "$RPC"
cast call "$BUIDL_TIMELOCK" 'isOperationDone(bytes32)(bool)' "$BUIDL_PENDING_NOOP_OPERATION" --rpc-url "$RPC"
```

## Manual Review Checklist

- Confirm the checked source is `9c812fcb32df0475ceaf443e3db39c3302a5e56c`.
- Confirm all 12 gist addresses still have verified source on Etherscan.
- Confirm Chronicle providers are tolled/kissed on their underlying Chronicle oracles.
- Do not copy the standalone fork-test oracle constants from `test/fork/ChronicleRateProviderForkTest.sol` for the deployed providers; the live deployed providers point at the oracles listed above.
- Confirm `DeployChronicleRateProvider` outputs are not considered ready until oracle authorization is complete.
- Confirm the BUIDL deployment flow did not accidentally add more than one active `BUIDLTokenRedeemer`.
- Confirm no address has unexpected open roles, especially `EXECUTOR_ROLE` on `address(0)`.
- Confirm whether the pending BUIDL timelock no-op should remain queued. If it was not intentional, an authorized canceller should cancel it.
- Confirm any deployer/admin retention is an intentional policy decision. This live-address document follows the clean deployed-state checks in `test/fork/VerifyDeployment.sol`.
- Keep client deployment-process claims separate from chain-verified facts. Do not treat fresh-clone, key-handling, Tenderly, or independent-review checklist ticks as independently verified unless separate evidence is provided.

