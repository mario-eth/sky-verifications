# Grove PAU deployment verification

Deploy output: `script/output/1/deploy-mainnet-production-1780680095.json`
Chain: Ethereum mainnet (`chain_id` 1)
Deployment window checked: blocks `25200000` through `25286472`
Deployment output timestamp: `2026-06-05 17:21:35 UTC`

## Summary

| Item | Result |
|---|---:|
| Deploy output addresses checked | 3 / 3 |
| `diamond-pau` release checked | `v1.13.0` |
| `diamond-pau` commit checked | `5c5ad6ae174bf467081ca82342ced2bd42a5c732` |
| `diamond-pau` remote tag matched | yes |
| AccessControls runtime logic matched local artifact | yes |
| Controller runtime logic matched local artifact | yes |
| AdministeredAgent runtime logic matched local artifact | yes |
| Full runtime metadata hash matched | no, metadata footer only |
| Controller constructor dependencies matched | 4 / 4 |
| Fresh controller has `CONTROLLER` on ALMProxy | no |
| Fresh controller has `CONTROLLER` on RateLimits | no |
| AccessControls roles matched scripted state | yes |
| AdministeredAgent roles matched scripted state | yes |
| Controller integrations matched Beacon | 4 / 4 |
| Maple max exchange rate copied from old controller | yes |
| Uniswap V3 AUSD/USDC config copied from old controller | yes |
| AccessControls logs in checked window | 4 / 4 |
| Controller logs in checked window | 11 / 11 |
| AdministeredAgent logs in checked window | 7 / 7 |
| Extra target-contract logs in checked window | 0 |

## Deployed Contracts

| Contract | Address |
|---|---|
| AccessControls | `0x10d1AdE77F1b81Ef95057bb2fACE292313F66277` |
| Controller | `0x0DD65461610Fe5b65cE50A870B10ED0F3d24d8C2` |
| AdministeredAgent | `0x0f7ca6616CC38132530dC4695778a54de42C21F4` |

## Source Releases

| Scope | Release | Commit |
|---|---|---|
| Diamond core | `sky-ecosystem/diamond-pau v1.13.0` | `5c5ad6ae174bf467081ca82342ced2bd42a5c732` |
| AdministeredAgent source | [sky-ecosystem/pau-administered-agent](https://github.com/sky-ecosystem/pau-administered-agent/tree/bfaaf709a8664d74d12604455f0365a0a12439cf) | `bfaaf709a8664d74d12604455f0365a0a12439cf` |

## Method

| Scope | Check |
|---|---|
| Deploy output | JSON addresses compared to the expected mainnet deployment addresses |
| Source release | `lib/diamond-pau` gitlink and remote `v1.13.0` tag checked |
| Bytecode | initial audit check used `forge build`, `eth_getCode`, local artifact runtime comparison, and Controller immutable beacon patching; one-time script checks the resulting executable-runtime hashes |
| Metadata caveat | executable runtime matched exactly after stripping the Solidity CBOR metadata footer; full bytecode differed only in the 53-byte metadata hash |
| Controller dependencies | `beacon()`, `accessControls()`, `proxy()`, and `rateLimits()` read live |
| AccessControls | `DEFAULT_ADMIN_ROLE` and `ALLOCATOR_ROLE` member counts and members read live |
| AdministeredAgent | admin, actor, grantor, and revoker counts/members read live |
| Controller integrations | four configured integration IDs read live and compared against the live Beacon config |
| Copied config | Maple `erc4626_getMaxExchangeRate` and Uniswap V3 pool params compared to old ALM controller |
| Events | chunked `eth_getLogs` scans over the target contracts, avoiding public RPC 50k-block limits |

## Verified Live State

### AccessControls

| Role | Expected member set |
|---|---|
| `DEFAULT_ADMIN_ROLE` | `0x1369f7b2b38c76B6478c0f0E66D94923421891Ba` |
| `ALLOCATOR_ROLE` | `0x0f7ca6616CC38132530dC4695778a54de42C21F4` |

The deployer `0x1ca4ECaF0E13ca833c80dA835DEEa15e1684361d` and `PAU_FACTORY` have no `DEFAULT_ADMIN_ROLE`.

### Controller

| Getter | Expected value |
|---|---|
| `beacon()` | `0x829dC2b7E94B1954F0764E573f2E0d45Afa28199` |
| `accessControls()` | `0x10d1AdE77F1b81Ef95057bb2fACE292313F66277` |
| `proxy()` | `0x491EDFB0B8b608044e227225C715981a30F3A44E` |
| `rateLimits()` | `0x5F5cfCB8a463868E37Ab27B5eFF3ba02112dF19a` |

External dependency roles:

| Dependency | Role | Fresh controller member? |
|---|---|---:|
| `ALM_PROXY` `0x491EDFB0B8b608044e227225C715981a30F3A44E` | `CONTROLLER` | no |
| `ALM_RATE_LIMITS` `0x5F5cfCB8a463868E37Ab27B5eFF3ba02112dF19a` | `CONTROLLER` | no |

The deployment scripts store these dependency addresses in the fresh controller, but they do not grant the fresh controller the external `CONTROLLER` role on either shared dependency.

Configured integrations:

- `BASIN_FACET`
- `ERC4626_FACET`
- `MAPLE_FACET`
- `UNISWAP_V3_FACET`

Copied config:

| Field | Live value |
|---|---:|
| Maple `MAPLE_SYRUP_USDC` max exchange rate | `3000000000000000000000000000000000000` |
| Uniswap V3 AUSD/USDC max slippage | `999000000000000000` |
| Uniswap V3 AUSD/USDC max tick delta | `200` |
| Uniswap V3 AUSD/USDC lower tick | `-10` |
| Uniswap V3 AUSD/USDC upper tick | `10` |
| Uniswap V3 AUSD/USDC TWAP seconds ago | `600` |

### AdministeredAgent

| Set | Expected values |
|---|---|
| Admins | `0x1369f7b2b38c76B6478c0f0E66D94923421891Ba` |
| Actors | `0x0eEC86649E756a23CBc68d9EFEd756f16aD5F85f`, `0x4364D17B578b0eD1c42Be9075D774D1d6AeAFe96`, `0x9187807e07112359C481870feB58f0c117a29179` |
| Grantors | none |
| Revokers | `0xB0113804960345fd0a245788b3423319c86940e5` |

## One-Time Verification Script

Prerequisites: `git`, `bash`, `node`, `cast`, Ethereum mainnet RPC.

Run from any directory. The script uses the current `grove-pau-deploy` checkout when it is already in one; otherwise it clones the pinned repo into `VERIFY_DIR`. The bytecode check compares live executable runtime hashes against the hashes produced by the artifact comparison above, so it does not need to initialize the full dependency tree. By default the event check scans the recorded audit window, blocks `25200000` through `25286472`.

```bash
set -euo pipefail

export MAINNET_RPC_URL="${MAINNET_RPC_URL:-<mainnet-rpc>}"
if [[ -z "${MAINNET_RPC_URL:-}" || "$MAINNET_RPC_URL" == "<mainnet-rpc>" ]]; then
  echo 'ERROR: set MAINNET_RPC_URL to an Ethereum mainnet JSON-RPC URL' >&2
  echo '  example: MAINNET_RPC_URL=https://ethereum-rpc.publicnode.com bash verify-grove-pau.sh' >&2
  exit 2
fi

for cmd in git node cast; do
  if ! command -v "$cmd" >/dev/null 2>&1; then
    echo "ERROR: missing required command: $cmd" >&2
    exit 2
  fi
done

export VERIFY_DIR="${VERIFY_DIR:-$(mktemp -d)}"
export GROVE_PAU_DEPLOY_REPO="${GROVE_PAU_DEPLOY_REPO:-https://github.com/sky-ecosystem/grove-pau-deploy.git}"
export GROVE_PAU_DEPLOY_REF="${GROVE_PAU_DEPLOY_REF:-3d6c4e761c0df94a831567065c7084e997f04d35}"
export GROVE_PAU_DEPLOY_DIR="${GROVE_PAU_DEPLOY_DIR:-$VERIFY_DIR/grove-pau-deploy}"

if [[ -f script/output/1/deploy-mainnet-production-1780680095.json && -d .git ]]; then
  GROVE_PAU_DEPLOY_DIR="$(pwd)"
else
  mkdir -p "$VERIFY_DIR"
  if [[ -d "$GROVE_PAU_DEPLOY_DIR/.git" ]]; then
    git -C "$GROVE_PAU_DEPLOY_DIR" fetch --tags origin
  else
    git clone "$GROVE_PAU_DEPLOY_REPO" "$GROVE_PAU_DEPLOY_DIR"
  fi
  git -C "$GROVE_PAU_DEPLOY_DIR" checkout "$GROVE_PAU_DEPLOY_REF"
fi

cd "$GROVE_PAU_DEPLOY_DIR"

if [[ ! -f script/output/1/deploy-mainnet-production-1780680095.json ]]; then
  echo 'ERROR: deploy output not found after preparing grove-pau-deploy checkout' >&2
  exit 2
fi

node <<'NODE'
const { execFileSync } = require('child_process');
const fs = require('fs');

const rpc = process.env.MAINNET_RPC_URL;
const fromBlock = Number(process.env.VERIFY_FROM_BLOCK || '25200000');
const toBlockEnv = process.env.VERIFY_TO_BLOCK || '25286472';
const output = JSON.parse(fs.readFileSync('script/output/1/deploy-mainnet-production-1780680095.json', 'utf8'));

const expected = {
  accessControls: '0x10d1AdE77F1b81Ef95057bb2fACE292313F66277',
  controller: '0x0DD65461610Fe5b65cE50A870B10ED0F3d24d8C2',
  administeredAgent: '0x0f7ca6616CC38132530dC4695778a54de42C21F4',
};

const constants = {
  deployer: '0x1ca4ECaF0E13ca833c80dA835DEEa15e1684361d',
  groveProxy: '0x1369f7b2b38c76B6478c0f0E66D94923421891Ba',
  pauFactory: '0x69A5d548830AC2A4Ba90A44a2C75BDA71f97fc66',
  administeredAgentFactory: '0x2968c3b5478cF93B70aB1e24255d4EDBBd27a089',
  beacon: '0x829dC2b7E94B1954F0764E573f2E0d45Afa28199',
  almProxy: '0x491EDFB0B8b608044e227225C715981a30F3A44E',
  rateLimits: '0x5F5cfCB8a463868E37Ab27B5eFF3ba02112dF19a',
  oldController: '0xfd9dEA9a8D5B955649579Af482DB7198A392A9F5',
  mapleSyrupUsdc: '0x80ac24aA929eaF5013f6436cdA2a7ba190f5Cc0b',
  uniswapV3AusdUsdc: '0xbAFeAd7c60Ea473758ED6c6021505E8BBd7e8E5d',
  almRelayer: '0x0eEC86649E756a23CBc68d9EFEd756f16aD5F85f',
  primaryRelayerOperator: '0x4364D17B578b0eD1c42Be9075D774D1d6AeAFe96',
  secondaryRelayerOperator: '0x9187807e07112359C481870feB58f0c117a29179',
  almFreezer: '0xB0113804960345fd0a245788b3423319c86940e5',
  defaultAdminRole: '0x' + '00'.repeat(32),
  allocatorRole: '0x68bf109b95a5c15fb2bb99041323c27d15f8675e11bf7420a1cd6ad64c394f46',
  controllerRole: '0x70546d1c92f8c2132ae23a23f5177aa8526356051c7510df99f50e012d221529',
};

function sh(cmd, args) {
  return execFileSync(cmd, args, { encoding: 'utf8', stdio: ['ignore', 'pipe', 'pipe'] }).trim();
}

function cast(args) {
  return sh('cast', [...args, '--rpc-url', rpc]);
}

function assertEq(label, actual, expectedValue) {
  const a = String(actual).trim();
  const e = String(expectedValue).trim();
  if (a !== e) {
    throw new Error(`${label}: expected ${e}, got ${a}`);
  }
  console.log(`pass: ${label}`);
}

function assertAddr(label, actual, expectedValue) {
  assertEq(label, actual.toLowerCase(), expectedValue.toLowerCase());
}

function dec(value) {
  return BigInt(String(value).split(/\s+/)[0]).toString();
}

function normalize(hex) {
  return String(hex).trim().replace(/^0x/, '').toLowerCase();
}

function stripMetadata(hex) {
  const bytes = Buffer.from(normalize(hex), 'hex');
  const len = bytes.readUInt16BE(bytes.length - 2);
  const cut = bytes.length - 2 - len;
  if (cut <= 0 || cut > bytes.length) return bytes.toString('hex');
  return bytes.subarray(0, cut).toString('hex');
}

function keccak(hex) {
  return sh('cast', ['keccak', '0x' + normalize(hex)]);
}

function call(address, signature, args = []) {
  return cast(['call', address, signature, ...args]);
}

function logs(address, from, to) {
  const raw = cast(['logs', '--json', '--from-block', String(from), '--to-block', String(to), '--address', address]);
  return raw ? JSON.parse(raw) : [];
}

for (const [key, value] of Object.entries(expected)) {
  assertAddr(`deploy output ${key}`, output[key], value);
}

const diamondGitlink = sh('git', ['ls-tree', 'HEAD', 'lib/diamond-pau']).split(/\s+/)[2];
assertEq('diamond-pau gitlink is v1.13.0 commit', diamondGitlink, '5c5ad6ae174bf467081ca82342ced2bd42a5c732');

const administeredAgentGitlink = sh('git', ['ls-tree', 'HEAD', 'lib/pau-administered-agent']).split(/\s+/)[2];
assertEq('pau-administered-agent gitlink', administeredAgentGitlink, 'bfaaf709a8664d74d12604455f0365a0a12439cf');

const remoteTag = sh('git', ['ls-remote', '--tags', 'https://github.com/sky-ecosystem/diamond-pau.git', 'refs/tags/v1.13.0']).split(/\s+/)[0];
assertEq('diamond-pau remote v1.13.0 tag', remoteTag, '5c5ad6ae174bf467081ca82342ced2bd42a5c732');

const bytecodeChecks = [
  {
    name: 'AccessControls',
    address: expected.accessControls,
    logicBytes: 1900,
    logicHash: '0x3e9ac6ac38bc908e14d6874b22b29d0dad05f31f3a230f9ed20959abeadb3b9e',
  },
  {
    name: 'Controller',
    address: expected.controller,
    logicBytes: 6283,
    logicHash: '0xdcb5cb728da3e9e2bf9fcd7bb7bfa2c354485f03b3515e0b2e5d6c1759e1ffbc',
  },
  {
    name: 'AdministeredAgent',
    address: expected.administeredAgent,
    logicBytes: 4949,
    logicHash: '0xdccb2938a597505ace6afe1cb3b2ef4825f5c74a767af9dd2aa53a483fba6b5b',
  },
];

for (const check of bytecodeChecks) {
  const liveLogic = stripMetadata(cast(['code', check.address]));
  assertEq(`${check.name} executable runtime byte length`, liveLogic.length / 2, check.logicBytes);
  assertEq(`${check.name} executable runtime hash`, keccak(liveLogic), check.logicHash);
}

assertAddr('controller.beacon()', call(expected.controller, 'beacon()(address)'), constants.beacon);
assertAddr('controller.accessControls()', call(expected.controller, 'accessControls()(address)'), expected.accessControls);
assertAddr('controller.proxy()', call(expected.controller, 'proxy()(address)'), constants.almProxy);
assertAddr('controller.rateLimits()', call(expected.controller, 'rateLimits()(address)'), constants.rateLimits);
assertEq('ALMProxy CONTROLLER role absent for fresh controller', call(constants.almProxy, 'hasRole(bytes32,address)(bool)', [constants.controllerRole, expected.controller]), 'false');
assertEq('RateLimits CONTROLLER role absent for fresh controller', call(constants.rateLimits, 'hasRole(bytes32,address)(bool)', [constants.controllerRole, expected.controller]), 'false');

assertEq('AccessControls default admin count', dec(call(expected.accessControls, 'getRoleMemberCount(bytes32)(uint256)', [constants.defaultAdminRole])), '1');
assertAddr(
  'AccessControls default admin member',
  call(expected.accessControls, 'getRoleMember(bytes32,uint256)(address)', [constants.defaultAdminRole, '0']),
  constants.groveProxy
);
assertEq('AccessControls allocator count', dec(call(expected.accessControls, 'getRoleMemberCount(bytes32)(uint256)', [constants.allocatorRole])), '1');
assertAddr(
  'AccessControls allocator member',
  call(expected.accessControls, 'getRoleMember(bytes32,uint256)(address)', [constants.allocatorRole, '0']),
  expected.administeredAgent
);
assertEq('deployer default admin revoked', call(expected.accessControls, 'hasRole(bytes32,address)(bool)', [constants.defaultAdminRole, constants.deployer]), 'false');
assertEq('PAUFactory default admin absent', call(expected.accessControls, 'hasRole(bytes32,address)(bool)', [constants.defaultAdminRole, constants.pauFactory]), 'false');

assertEq('AdministeredAgent admin count', dec(call(expected.administeredAgent, 'adminCount()(uint256)')), '1');
assertAddr('AdministeredAgent admin[0]', call(expected.administeredAgent, 'getAdmin(uint256)(address)', ['0']), constants.groveProxy);
assertEq('AdministeredAgent actor count', dec(call(expected.administeredAgent, 'actorCount()(uint256)')), '3');
assertAddr('AdministeredAgent actor[0]', call(expected.administeredAgent, 'getActor(uint256)(address)', ['0']), constants.almRelayer);
assertAddr('AdministeredAgent actor[1]', call(expected.administeredAgent, 'getActor(uint256)(address)', ['1']), constants.primaryRelayerOperator);
assertAddr('AdministeredAgent actor[2]', call(expected.administeredAgent, 'getActor(uint256)(address)', ['2']), constants.secondaryRelayerOperator);
assertEq('AdministeredAgent grantor count', dec(call(expected.administeredAgent, 'grantorCount()(uint256)')), '0');
assertEq('AdministeredAgent revoker count', dec(call(expected.administeredAgent, 'revokerCount()(uint256)')), '1');
assertAddr('AdministeredAgent revoker[0]', call(expected.administeredAgent, 'getRevoker(uint256)(address)', ['0']), constants.almFreezer);

const integrationIds = ['BASIN_FACET', 'ERC4626_FACET', 'MAPLE_FACET', 'UNISWAP_V3_FACET'];
const integrations = call(expected.controller, 'integrations()((bytes32,(address,(bytes4,bytes4)[]))[])');
const integrationCount = (integrations.match(/0x[0-9a-fA-F]{40}, \[/g) || []).length;
assertEq('controller integration count', integrationCount, 4);

for (const id of integrationIds) {
  const idBytes32 = sh('cast', ['format-bytes32-string', id]);
  const controllerConfig = call(expected.controller, 'getConfig(bytes32)((address,(bytes4,bytes4)[]))', [idBytes32]);
  const beaconConfig = call(constants.beacon, 'getConfig(bytes32)((address,(bytes4,bytes4)[]))', [idBytes32]);
  assertEq(`controller config matches Beacon: ${id}`, controllerConfig, beaconConfig);
}

assertEq(
  'Maple max exchange rate copied',
  dec(call(expected.controller, 'erc4626_getMaxExchangeRate(address)(uint256)', [constants.mapleSyrupUsdc])),
  dec(call(constants.oldController, 'maxExchangeRates(address)(uint256)', [constants.mapleSyrupUsdc]))
);

assertEq(
  'Uniswap max slippage copied',
  dec(call(expected.controller, 'uniswapV3_getMaxSlippage(address)(uint256)', [constants.uniswapV3AusdUsdc])),
  dec(call(constants.oldController, 'maxSlippages(address)(uint256)', [constants.uniswapV3AusdUsdc]))
);
assertEq('Uniswap max tick delta', dec(call(expected.controller, 'uniswapV3_getMaxTickDelta(address)(uint24)', [constants.uniswapV3AusdUsdc])), '200');
assertEq('Uniswap tick bounds', call(expected.controller, 'uniswapV3_getLiquidityTickBounds(address)(int24,int24)', [constants.uniswapV3AusdUsdc]), '-10\n10');
assertEq('Uniswap TWAP seconds ago', dec(call(expected.controller, 'uniswapV3_getTWAPSecondsAgo(address)(uint32)', [constants.uniswapV3AusdUsdc])), '600');

let toBlock;
if (toBlockEnv === 'latest') {
  toBlock = Number(cast(['block-number']));
} else {
  toBlock = Number(toBlockEnv);
}

const expectedLogCounts = new Map([
  [expected.accessControls.toLowerCase(), 4],
  [expected.controller.toLowerCase(), 11],
  [expected.administeredAgent.toLowerCase(), 7],
]);

for (const [address, expectedCount] of expectedLogCounts.entries()) {
  let count = 0;
  for (let start = fromBlock; start <= toBlock; start += 50000) {
    const end = Math.min(start + 49999, toBlock);
    count += logs(address, start, end).length;
  }
  assertEq(`log count ${address}`, count, expectedCount);
}

console.log('');
console.log('PASS: Grove PAU deployment bytecode, live state, and target-contract event counts verified');
NODE
```


