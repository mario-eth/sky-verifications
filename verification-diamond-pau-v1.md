# Sky PAU registry bytecode and wiring verification

Registry: [sky-ecosystem/sky-pau-registry `src/Ethereum.sol` @ `161bac0c17a7d2c4d4e0455e1febe401a7a36edb`](https://github.com/sky-ecosystem/sky-pau-registry/blob/161bac0c17a7d2c4d4e0455e1febe401a7a36edb/src/Ethereum.sol)
Chain: Ethereum mainnet (`chain_id` 1)
Block: `25273612`

## Summary

| Item | Count |
|---|---:|
| Registry addresses checked | 28 |
| Exact runtime bytecode matches | 28 |
| Forge creation full matches | 28 |
| Forge runtime full matches | 28 |
| `PAU_FACTORY.beacon()` matches registry `BEACON` | 1 |
| Beacon integrations matched | 25 / 25 |
| Beacon selector dispatches matched | 223 / 223 |
| Facet dependency getters matched | 29 / 29 |
| External dependencies checked | 20 |
| External dependency sanity checks passed | 20 / 20 |
| External token metadata checks passed | 9 / 9 |
| Beacon role IDs seen on-chain | `DEFAULT_ADMIN_ROLE` only |
| Beacon admin identity | Sky `MCD_PAUSE_PROXY` |
| `PAU_FACTORY` access model | permissionless |
| Registry Solidity constants diffed against verified address set | 28 / 28 |
| Registry address checksums (EIP-55) valid | 28 / 28 |
| Registry facets vs deploy-script facets | 25 / 25 (exact, none omitted/extra) |
| Wiring reference source | clean `diamond-pau@v1.13.0` checkout |
| Administered-agent factory live fork checks | 2 / 2 |
| Ordered deploy event provenance test | gated by `RUN_EVENTS=1` |
| Wiring mismatches | 0 |
| Failures | 0 |
| Unchecked registry addresses | 0 |

## Source Releases

| Scope | Release | Commit |
|---|---|---|
| Diamond core and facets | [sky-ecosystem/diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | `5c5ad6ae174bf467081ca82342ced2bd42a5c732` |
| Administered-agent factory | [sky-ecosystem/pau-administered-agent v1.0.0](https://github.com/sky-ecosystem/pau-administered-agent/releases/tag/v1.0.0) | `bfaaf709a8664d74d12604455f0365a0a12439cf` |

## Method

| Scope | Check |
|---|---|
| Registry constants | `sky-pau-registry/src/Ethereum.sol` is diffed against the 28 verified addresses and each address is checked for EIP-55 casing |
| Diamond core and facets | [sky-ecosystem/diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0), mainnet constructor args, `eth_getCode`, `forge verify-bytecode` |
| Administered-agent factory | [sky-ecosystem/pau-administered-agent v1.0.0](https://github.com/sky-ecosystem/pau-administered-agent/releases/tag/v1.0.0), Etherscan source, `forge verify-bytecode`, runtime hash, live fork deploy/admin behavior |
| Wiring | `PAU_FACTORY.beacon()`, Beacon admin role, Beacon integrations, Beacon selector dispatches, facet dependency getters, reference rebuilt in a clean temp root from `diamond-pau@v1.13.0` |
| External dependencies | Mainnet code, Etherscan source/proxy metadata, token metadata, identity getters |
| Blocks | `25270766`, `25273612` |

## One-Stop Verification

Prerequisites: `git`, `bash`, `forge`, `cast`, Ethereum mainnet RPC.

Copy the full block, replace `MAINNET_RPC_URL`, and run it from any directory. The block writes the verifier scripts into `VERIFY_DIR`, clones the pinned GitHub sources, and runs the checks.

```bash
set -euo pipefail

export MAINNET_RPC_URL="${MAINNET_RPC_URL:-<mainnet-rpc>}"
if [[ -z "${MAINNET_RPC_URL:-}" || "$MAINNET_RPC_URL" == "<mainnet-rpc>" ]]; then
  echo 'ERROR: set MAINNET_RPC_URL to an Ethereum mainnet JSON-RPC URL, or replace <mainnet-rpc> above' >&2
  echo '  example: MAINNET_RPC_URL=https://ethereum-rpc.publicnode.com bash <this-script>' >&2
  exit 2
fi

export VERIFY_DIR="${VERIFY_DIR:-$(mktemp -d)}"
mkdir -p "$VERIFY_DIR/verification_scripts"

cat > "$VERIFY_DIR/verification_scripts/verify-sky-pau-registry-constants.sh" <<'SKY_PAU_CONSTANTS_SH'
#!/usr/bin/env bash
set -euo pipefail

usage() {
    cat <<'USAGE'
Usage:
  verification_scripts/verify-sky-pau-registry-constants.sh [registry-sol]

Default registry file:
  ../sky-pau-registry/src/Ethereum.sol, resolved from this deploy repo.

Checks:
  - all 28 expected Sky PAU registry constants are present
  - every expected constant equals the verified address
  - each address is EIP-55 checksummed
  - there are no extra address constants in the registry file
USAGE
}

if [[ "${1:-}" == "-h" || "${1:-}" == "--help" ]]; then
    usage
    exit 0
fi

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
REPO_ROOT="$(cd "$SCRIPT_DIR/.." && pwd)"
REGISTRY_FILE="${1:-${REGISTRY_FILE:-$REPO_ROOT/../sky-pau-registry/src/Ethereum.sol}}"

if [[ ! -f "$REGISTRY_FILE" ]]; then
    echo "ERROR: registry file not found: $REGISTRY_FILE" >&2
    exit 2
fi

if ! command -v cast >/dev/null 2>&1; then
    echo "ERROR: cast is required" >&2
    exit 2
fi

expected_count=0

check_constant() {
    local name="$1"
    local expected="$2"
    local actual
    local checksum

    expected_count=$((expected_count + 1))

    actual="$(
        sed -nE "s/.*address[[:space:]]+[^;]*constant[[:space:]]+$name[[:space:]]*=[[:space:]]*(0x[0-9a-fA-F]{40});.*/\\1/p" "$REGISTRY_FILE"
    )"

    if [[ -z "$actual" ]]; then
        echo "FAIL: missing registry constant $name" >&2
        return 1
    fi

    if [[ "$actual" != "$expected" ]]; then
        echo "FAIL: $name mismatch" >&2
        echo "  expected: $expected" >&2
        echo "  actual:   $actual" >&2
        return 1
    fi

    checksum="$(cast to-check-sum-address "$actual")"
    if [[ "$checksum" != "$actual" ]]; then
        echo "FAIL: $name is not EIP-55 checksummed" >&2
        echo "  expected checksum: $checksum" >&2
        echo "  actual:            $actual" >&2
        return 1
    fi

    echo "pass: $name = $actual"
}

check_constant BEACON                     0x829dC2b7E94B1954F0764E573f2E0d45Afa28199
check_constant PAU_FACTORY                0x69A5d548830AC2A4Ba90A44a2C75BDA71f97fc66
check_constant ADMINISTERED_AGENT_FACTORY 0x2968c3b5478cF93B70aB1e24255d4EDBBd27a089

check_constant AAVE_FACET           0x8CE890A96a193ff2DD4B2eA3C682326F655f6b62
check_constant BASIN_FACET          0xC84825BCD13AEddc372400239499380376a44A39
check_constant CCTP_FACET           0xADf62692340e46EF90336f2e75ce3b37f1148873
check_constant CENTRIFUGE_FACET     0xa0A10BA97be1412730D694B8dE1afe7eff20eC31
check_constant CURVE_FACET          0x139D81d7d6040fAeF7cF0EF5A2636Ca8a97a30d8
check_constant DAIUSDS_FACET        0x3817F734CAe6AD2BDb79F9ff23091F2AD478da5F
check_constant ERC4626_FACET        0x1dCA18608c89174181153E786778705b4A0E1a06
check_constant ERC7540_FACET        0x4f7e0E3612b0e1E156A2B6570a51d4BD709F1315
check_constant ETHENA_FACET         0xEc48D773CEef1c6b07CdA1afA2716C478b55187B
check_constant FARM_FACET           0xF24E91f5D8529436c9fB92dd94F80d4A6C25d0f0
check_constant LAYER_ZERO_FACET     0xA0c323a0acb20F259eA4ff343319D450BE6472e5
check_constant MAPLE_FACET          0x691b5c26aD2B74d2376f4eD87904E9D3E47bD630
check_constant MERKL_FACET          0x321138Db5E056e9d0080D4c278e10A1EdC091Eb0
check_constant OTC_FACET            0x46b24ba00B65CB4f603447590e539b08097fb7Ac
check_constant PENDLE_FACET         0xcC9dD4c9B2a9c08f2692e7060F43d29A03E87348
check_constant PSM_FACET            0xE4A5dAc768a310cc2316f258901b32E499653064
check_constant SPARK_VAULT_FACET    0xff0d19920E207e3A17eb5A2E5bA3AFA44836362b
check_constant SUPERSTATE_FACET     0xeE197475607E9a27cCAA4786e740d2F0d0E706A7
check_constant TRANSFER_ASSET_FACET 0x4DA7608C331b8f135df5b985018933780eCd089D
check_constant UNISWAP_V3_FACET     0x445D9Dc752F269Be48250f1A180CAC4c61cE4bab
check_constant UNISWAP_V4_FACET     0x75D35ffB8e6B871E12EB549CcF6afD324c46E47D
check_constant USDS_FACET           0x1221CC4B85Ab260660aD21C2829e0EB516dffBc7
check_constant WEETH_FACET          0x1d8D089EB7D558F5dc6aA0cf98DDe13B77b3F641
check_constant WRAP_PROXY_ETH_FACET 0x081506DE21C695Af5e61a81aD288C8A96B6b59B9
check_constant WSTETH_FACET         0x3a82D11Cd37Fb0098363262Dc69425d07Fa05516

actual_count="$(
    sed -nE 's/.*address[[:space:]]+[^;]*constant[[:space:]]+[A-Z0-9_]+[[:space:]]*=[[:space:]]*0x[0-9a-fA-F]{40};.*/x/p' "$REGISTRY_FILE" |
        wc -l |
        tr -d ' '
)"

if [[ "$actual_count" != "$expected_count" ]]; then
    echo "FAIL: registry address constant count mismatch" >&2
    echo "  expected: $expected_count" >&2
    echo "  actual:   $actual_count" >&2
    exit 1
fi

echo
echo "PASS: $expected_count Sky PAU registry constants match verified addresses"
SKY_PAU_CONSTANTS_SH

cat > "$VERIFY_DIR/verification_scripts/verify-sky-pau-registry-bytecode.sh" <<'SKY_PAU_BYTECODE_SH'
#!/usr/bin/env bash
set -euo pipefail

usage() {
    cat <<'USAGE'
Usage:
  MAINNET_RPC_URL=<rpc> verification_scripts/verify-sky-pau-registry-bytecode.sh [diamond-pau-tag]

Default:
  diamond-pau-tag v1.13.0

Optional env:
  ETH_RPC_URL                 fallback RPC env var if MAINNET_RPC_URL is unset
  VERIFY_BLOCK                block for historical bytecode, default 25270766
  VERIFY_ONLY                 comma-separated registry constants, for example: BEACON,PAU_FACTORY,AAVE_FACET
  DIAMOND_PAU_REPO            default https://github.com/sky-ecosystem/diamond-pau.git
  VERIFY_DIR                  working root, default: mktemp
  DIAMOND_PAU_RELEASE_DIR     checkout dir, default $VERIFY_DIR/diamond-pau-v1.13.0
  DIAMOND_PAU_WORKDIR         parent build dir, default $VERIFY_DIR/diamond-pau-parent-v1.13.0
  REGISTRY_FILE               default ../sky-pau-registry/src/Ethereum.sol
  SKIP_REGISTRY_CONSTANTS=1   skip registry constants diff
  SKIP_AA=1                   skip ADMINISTERED_AGENT_FACTORY
  AA_WORKDIR                  temp parent build dir, default $VERIFY_DIR/pau-aa-parent
  AA_RELEASE_DIR              pau-administered-agent checkout, default $VERIFY_DIR/pau-administered-agent-v1.0.0

This verifies:
  - BEACON
  - PAU_FACTORY
  - 25 PAU facets
  - ADMINISTERED_AGENT_FACTORY, unless SKIP_AA=1

The script requires both:
  Creation code matched with status full
  Runtime code matched with status full
USAGE
}

if [[ "${1:-}" == "-h" || "${1:-}" == "--help" ]]; then
    usage
    exit 0
fi

TAG="${1:-v1.13.0}"
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
REPO_ROOT="$(cd "$SCRIPT_DIR/.." && pwd)"
RPC_URL="${MAINNET_RPC_URL:-${ETH_RPC_URL:-}}"
VERIFY_BLOCK="${VERIFY_BLOCK:-25270766}"
VERIFY_DIR="${VERIFY_DIR:-$(mktemp -d)}"
VERIFY_ONLY="${VERIFY_ONLY:-}"
SKIP_AA="${SKIP_AA:-0}"
SKIP_REGISTRY_CONSTANTS="${SKIP_REGISTRY_CONSTANTS:-0}"
REGISTRY_FILE="${REGISTRY_FILE:-$REPO_ROOT/../sky-pau-registry/src/Ethereum.sol}"
DIAMOND_PAU_REPO="${DIAMOND_PAU_REPO:-https://github.com/sky-ecosystem/diamond-pau.git}"

if [[ -z "$RPC_URL" ]]; then
    echo "ERROR: set MAINNET_RPC_URL or ETH_RPC_URL" >&2
    exit 2
fi

if [[ "$RPC_URL" == "<mainnet-rpc>" ]]; then
    echo "ERROR: replace <mainnet-rpc> with an Ethereum mainnet RPC URL" >&2
    exit 2
fi

if ! command -v forge >/dev/null 2>&1; then
    echo "ERROR: forge is required" >&2
    exit 2
fi

should_run() {
    local name="$1"

    if [[ -z "$VERIFY_ONLY" ]]; then
        return 0
    fi

    case ",$VERIFY_ONLY," in
        *",$name,"*) return 0 ;;
        *) return 1 ;;
    esac
}

require_full_match() {
    local name="$1"
    local output_file="$2"

    if ! grep -q "Creation code matched with status full" "$output_file"; then
        cat "$output_file"
        echo "FAIL: $name creation bytecode did not fully match" >&2
        return 1
    fi

    if ! grep -q "Runtime code matched with status full" "$output_file"; then
        cat "$output_file"
        echo "FAIL: $name runtime bytecode did not fully match" >&2
        return 1
    fi

    grep -E "Creation code matched|Runtime code matched" "$output_file"
}

verify_one() {
    local name="$1"
    local address="$2"
    local contract="$3"
    shift 3

    if ! should_run "$name"; then
        return 0
    fi

    local cmd=(forge verify-bytecode --root "$DIAMOND_PAU_BUILD_ROOT" --rpc-url "$RPC_URL" --block "$VERIFY_BLOCK" "$address" "$contract")
    if (( $# > 0 )); then
        cmd+=(--constructor-args "$@")
    fi

    local tmp
    tmp="$(mktemp "$VERIFY_DIR/pau-registry-bytecode.XXXXXX")"

    echo "==> $name $address $contract"
    if ! "${cmd[@]}" >"$tmp" 2>&1; then
        cat "$tmp"
        rm -f "$tmp"
        echo "FAIL: $name forge verify-bytecode exited nonzero" >&2
        return 1
    fi

    require_full_match "$name" "$tmp"
    rm -f "$tmp"
}

write_diamond_pau_foundry_toml() {
    local target="$1"

    cat > "$target" <<'EOF'
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = '0.8.34'
optimizer = true
optimizer_runs = 200
evm_version = 'cancun'
auto_detect_remappings = false
ffi = true
fs_permissions = [
    { access = "read", path = "./script/input/" },
    { access = "read-write", path = "./script/output/"}
]
remappings = [
    'lib/diamond-pau/:dss-test/=lib/diamond-pau/lib/dss-test/src/',
    '@ensdomains/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/v4-core/node_modules/@ensdomains/',
    '@layerzerolabs/lz-evm-messagelib-v2/=lib/diamond-pau/lib/layerzero-v2/packages/layerzero-v2/evm/messagelib/',
    '@layerzerolabs/lz-evm-protocol-v2/=lib/diamond-pau/lib/layerzero-v2/packages/layerzero-v2/evm/protocol/',
    '@layerzerolabs/oft-evm/=lib/diamond-pau/lib/grove-xchain-helpers/lib/devtools/packages/oft-evm/',
    '@openzeppelin/contracts-upgradeable/=lib/diamond-pau/lib/oz-upgradeable/contracts/',
    '@openzeppelin/contracts/=lib/diamond-pau/lib/openzeppelin-contracts/contracts/',
    '@uniswap/v4-core/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/v4-core/',
    'LayerZero-v2/=lib/diamond-pau/lib/grove-xchain-helpers/lib/',
    'aave-v3-core/=lib/diamond-pau/lib/aave-v3-origin/src/core/',
    'aave-v3-origin/=lib/diamond-pau/lib/aave-v3-origin/',
    'aave-v3-periphery/=lib/diamond-pau/lib/aave-v3-origin/src/periphery/',
    'devtools/=lib/diamond-pau/lib/grove-xchain-helpers/lib/devtools/packages/toolbox-foundry/src/',
    'diamond-pau/=lib/diamond-pau/src/',
    'ds-test/=lib/diamond-pau/lib/grove-address-registry/lib/forge-std/lib/ds-test/src/',
    'dss-allocator/=lib/diamond-pau/lib/dss-allocator/',
    'dss-interfaces/=lib/dss-test/lib/dss-interfaces/src/',
    'dss-test/=lib/dss-test/src/',
    'erc20-helpers/=lib/diamond-pau/lib/grove-basin/lib/erc20-helpers/src/',
    'erc4626-tests/=lib/diamond-pau/lib/metamorpho/lib/erc4626-tests/',
    'forge-gas-snapshot/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/permit2/lib/forge-gas-snapshot/src/',
    'forge-std/=lib/forge-std/src/',
    'grove-address-registry/=lib/diamond-pau/lib/grove-address-registry/src/',
    'grove-basin/=lib/diamond-pau/lib/grove-basin/',
    'grove-xchain-helpers/=lib/diamond-pau/lib/grove-xchain-helpers/src/',
    'halmos-cheatcodes/=lib/diamond-pau/lib/grove-basin/lib/openzeppelin-contracts/lib/halmos-cheatcodes/src/',
    'hardhat/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/v4-core/node_modules/hardhat/',
    'layerzero-v2/=lib/diamond-pau/lib/',
    'layerzerolabs/oapp-evm/=lib/diamond-pau/lib/grove-xchain-helpers/lib/devtools/packages/oapp-evm/',
    'metamorpho/=lib/diamond-pau/lib/metamorpho/src/',
    'morpho-blue/=lib/diamond-pau/lib/metamorpho/lib/morpho-blue/',
    'murky/=lib/diamond-pau/lib/metamorpho/lib/universal-rewards-distributor/lib/murky/src/',
    'openzeppelin-contracts-upgradeable/=lib/diamond-pau/lib/spark-vaults-v2/lib/openzeppelin-contracts-upgradeable/',
    'openzeppelin-contracts/=lib/diamond-pau/lib/openzeppelin-contracts/',
    'openzeppelin/=lib/diamond-pau/lib/metamorpho/lib/universal-rewards-distributor/lib/openzeppelin-contracts/contracts/',
    'oz-upgradeable/=lib/diamond-pau/lib/oz-upgradeable/',
    'permit2/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/permit2/',
    'solidity-bytes-utils/=lib/diamond-pau/lib/solidity-bytes-utils/',
    'solidity-utils/=lib/diamond-pau/lib/aave-v3-origin/lib/solidity-utils/',
    'solmate/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/v4-core/lib/solmate/',
    'spark-address-registry/=lib/diamond-pau/lib/spark-address-registry/src/',
    'spark-psm/=lib/diamond-pau/lib/spark-psm/',
    'spark-vaults-v2/=lib/diamond-pau/lib/spark-vaults-v2/',
    'sparklend-address-registry/=lib/diamond-pau/lib/grove-basin/lib/xchain-ssr-oracle/lib/sparklend-address-registry/',
    'token-tests/=lib/diamond-pau/lib/spark-vaults-v2/lib/token-tests/src/',
    'uniswap-v4-periphery/=lib/diamond-pau/lib/uniswap-v4-periphery/',
    'universal-rewards-distributor/=lib/diamond-pau/lib/metamorpho/lib/universal-rewards-distributor/src/',
    'v4-core/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/v4-core/src/',
    'xchain-helpers/=lib/diamond-pau/lib/xchain-helpers/src/',
    'xchain-ssr-oracle/=lib/diamond-pau/lib/grove-basin/lib/xchain-ssr-oracle/',
]

[fuzz]
runs = 1000

[etherscan]
mainnet      = { key = "${MAINNET_API_KEY}" }
base         = { key = "${BASESCAN_API_KEY}" }
arbitrum_one = { key = "${ARBISCAN_API_KEY}" }
avalanche    = { key = "${SNOWTRACE_API_KEY}" }
EOF
}

write_aa_foundry_toml() {
    local target="$1"

    # The deployed AdministeredAgentFactory was compiled inside the diamond-pau-deploy
    # workspace, so its metadata hash embeds the full deploy-repo remapping list
    # (confirmed against the Etherscan-verified standard-json settings). The remappings
    # must match exactly for a "full" bytecode match; only the pau-administered-agent
    # sources need to exist on disk.
    cat > "$target" <<'EOF'
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = '0.8.34'
optimizer = true
optimizer_runs = 200
evm_version = 'cancun'
auto_detect_remappings = false
remappings = [
    '@ensdomains/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/v4-core/node_modules/@ensdomains/',
    '@layerzerolabs/lz-evm-messagelib-v2/=lib/diamond-pau/lib/layerzero-v2/packages/layerzero-v2/evm/messagelib/',
    '@layerzerolabs/lz-evm-protocol-v2/=lib/diamond-pau/lib/layerzero-v2/packages/layerzero-v2/evm/protocol/',
    '@layerzerolabs/oft-evm/=lib/diamond-pau/lib/grove-xchain-helpers/lib/devtools/packages/oft-evm/',
    '@openzeppelin/contracts-upgradeable/=lib/diamond-pau/lib/oz-upgradeable/contracts/',
    '@openzeppelin/contracts/=lib/diamond-pau/lib/openzeppelin-contracts/contracts/',
    '@uniswap/v4-core/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/v4-core/',
    'LayerZero-v2/=lib/diamond-pau/lib/grove-xchain-helpers/lib/',
    'aave-v3-core/=lib/diamond-pau/lib/aave-v3-origin/src/core/',
    'aave-v3-origin/=lib/diamond-pau/lib/aave-v3-origin/',
    'aave-v3-periphery/=lib/diamond-pau/lib/aave-v3-origin/src/periphery/',
    'devtools/=lib/diamond-pau/lib/grove-xchain-helpers/lib/devtools/packages/toolbox-foundry/src/',
    'diamond-pau/=lib/diamond-pau/src/',
    'ds-test/=lib/grove-address-registry/lib/forge-std/lib/ds-test/src/',
    'dss-allocator/=lib/diamond-pau/lib/dss-allocator/',
    'dss-interfaces/=lib/dss-test/lib/dss-interfaces/src/',
    'dss-test/=lib/dss-test/src/',
    'erc20-helpers/=lib/diamond-pau/lib/grove-basin/lib/erc20-helpers/src/',
    'erc4626-tests/=lib/diamond-pau/lib/metamorpho/lib/erc4626-tests/',
    'forge-gas-snapshot/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/permit2/lib/forge-gas-snapshot/src/',
    'forge-std/=lib/forge-std/src/',
    'grove-address-registry/=lib/grove-address-registry/src/',
    'grove-basin/=lib/diamond-pau/lib/grove-basin/',
    'grove-xchain-helpers/=lib/diamond-pau/lib/grove-xchain-helpers/src/',
    'halmos-cheatcodes/=lib/pau-administered-agent/lib/openzeppelin/lib/halmos-cheatcodes/src/',
    'hardhat/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/v4-core/node_modules/hardhat/',
    'layerzero-v2/=lib/diamond-pau/lib/',
    'layerzerolabs/oapp-evm/=lib/diamond-pau/lib/grove-xchain-helpers/lib/devtools/packages/oapp-evm/',
    'metamorpho/=lib/diamond-pau/lib/metamorpho/src/',
    'morpho-blue/=lib/diamond-pau/lib/metamorpho/lib/morpho-blue/',
    'murky/=lib/diamond-pau/lib/metamorpho/lib/universal-rewards-distributor/lib/murky/src/',
    'openzeppelin-contracts-upgradeable/=lib/diamond-pau/lib/spark-vaults-v2/lib/openzeppelin-contracts-upgradeable/',
    'openzeppelin-contracts/=lib/diamond-pau/lib/openzeppelin-contracts/',
    'openzeppelin/=lib/pau-administered-agent/lib/openzeppelin/',
    'oz-upgradeable/=lib/diamond-pau/lib/oz-upgradeable/',
    'pau-administered-agent/=lib/pau-administered-agent/src/',
    'permit2/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/permit2/',
    'solidity-bytes-utils/=lib/diamond-pau/lib/solidity-bytes-utils/',
    'solidity-utils/=lib/diamond-pau/lib/aave-v3-origin/lib/solidity-utils/',
    'solmate/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/v4-core/lib/solmate/',
    'spark-address-registry/=lib/diamond-pau/lib/spark-address-registry/src/',
    'spark-psm/=lib/diamond-pau/lib/spark-psm/',
    'spark-vaults-v2/=lib/diamond-pau/lib/spark-vaults-v2/',
    'sparklend-address-registry/=lib/diamond-pau/lib/grove-basin/lib/xchain-ssr-oracle/lib/sparklend-address-registry/',
    'token-tests/=lib/diamond-pau/lib/spark-vaults-v2/lib/token-tests/src/',
    'uniswap-v4-periphery/=lib/diamond-pau/lib/uniswap-v4-periphery/',
    'universal-rewards-distributor/=lib/diamond-pau/lib/metamorpho/lib/universal-rewards-distributor/src/',
    'v4-core/=lib/diamond-pau/lib/uniswap-v4-periphery/lib/v4-core/src/',
    'xchain-helpers/=lib/diamond-pau/lib/xchain-helpers/src/',
    'xchain-ssr-oracle/=lib/diamond-pau/lib/grove-basin/lib/xchain-ssr-oracle/',
]
EOF
}

init_nested_submodules() {
    local release_dir="$1"

    git -C "$release_dir/lib/uniswap-v4-periphery" submodule update --init --recursive --quiet \
        lib/v4-core \
        lib/permit2

    git -C "$release_dir/lib/metamorpho" submodule update --init --recursive --quiet \
        lib/morpho-blue \
        lib/universal-rewards-distributor

    git -C "$release_dir/lib/grove-basin" submodule update --init --recursive --quiet \
        lib/erc20-helpers \
        lib/xchain-ssr-oracle

    git -C "$release_dir/lib/spark-psm" submodule update --init --recursive --quiet \
        lib/erc20-helpers

    git -C "$release_dir/lib/spark-vaults-v2" submodule update --init --recursive --quiet \
        lib/openzeppelin-contracts-upgradeable \
        lib/token-tests
}

prepare_diamond_pau_workdir() {
    local release_dir="${DIAMOND_PAU_RELEASE_DIR:-$VERIFY_DIR/diamond-pau-${TAG}}"
    local workdir="${DIAMOND_PAU_WORKDIR:-$VERIFY_DIR/diamond-pau-parent-${TAG}}"

    if [[ ! -d "$release_dir/.git" ]]; then
        echo "==> cloning diamond-pau $TAG into $release_dir" >&2
        rm -rf "$release_dir"
        git clone --quiet "$DIAMOND_PAU_REPO" "$release_dir"
    fi

    git -C "$release_dir" fetch --quiet --tags

    if ! git -C "$release_dir" rev-parse --verify "${TAG}^{commit}" >/dev/null 2>&1; then
        echo "ERROR: unknown diamond-pau tag/ref after fetch: $TAG" >&2
        exit 2
    fi

    if ! git -C "$release_dir" cat-file -e "${TAG}:src/facets/Facet.sol" 2>/dev/null; then
        echo "NOT_COMPARABLE: diamond-pau $TAG does not contain src/facets/Facet.sol"
        echo "The Sky PAU registry checked here is a Beacon + 25-facet deployment, so it cannot bytecode-match this tag."
        exit 20
    fi

    git -C "$release_dir" checkout --quiet "$TAG"
    # Keep this targeted. A full recursive update is slow and some historical nested refs can be
    # unavailable, but these nested dependencies are required by the facet build.
    git -C "$release_dir" submodule update --init --quiet
    init_nested_submodules "$release_dir"

    rm -rf "$workdir"
    mkdir -p "$workdir/lib" "$workdir/src"
    cp -R "$release_dir" "$workdir/lib/diamond-pau"
    write_diamond_pau_foundry_toml "$workdir/foundry.toml"

    forge build \
        --root "$workdir" \
        lib/diamond-pau/src/Beacon.sol \
        lib/diamond-pau/src/PAUFactory.sol \
        lib/diamond-pau/src/facets >/dev/null

    printf '%s\n' "$workdir"
}

ensure_diamond_pau_is_comparable() {
    local release_dir="${DIAMOND_PAU_RELEASE_DIR:-$VERIFY_DIR/diamond-pau-${TAG}}"

    if [[ ! -d "$release_dir/.git" ]]; then
        echo "==> cloning diamond-pau $TAG into $release_dir" >&2
        rm -rf "$release_dir"
        git clone --quiet "$DIAMOND_PAU_REPO" "$release_dir"
    fi

    git -C "$release_dir" fetch --quiet --tags

    if ! git -C "$release_dir" rev-parse --verify "${TAG}^{commit}" >/dev/null 2>&1; then
        echo "ERROR: unknown diamond-pau tag/ref after fetch: $TAG" >&2
        exit 2
    fi

    if ! git -C "$release_dir" cat-file -e "${TAG}:src/facets/Facet.sol" 2>/dev/null; then
        echo "NOT_COMPARABLE: diamond-pau $TAG does not contain src/facets/Facet.sol"
        echo "The Sky PAU registry checked here is a Beacon + 25-facet deployment, so it cannot bytecode-match this tag."
        exit 20
    fi
}

prepare_aa_workdir() {
    local release_dir="${AA_RELEASE_DIR:-$VERIFY_DIR/pau-administered-agent-v1.0.0}"
    local workdir="${AA_WORKDIR:-$VERIFY_DIR/pau-aa-parent}"

    if [[ ! -d "$release_dir/.git" ]]; then
        echo "==> cloning pau-administered-agent v1.0.0 into $release_dir" >&2
        rm -rf "$release_dir"
        git clone --quiet https://github.com/sky-ecosystem/pau-administered-agent.git "$release_dir"
    fi

    git -C "$release_dir" fetch --quiet --tags
    git -C "$release_dir" checkout --quiet bfaaf709a8664d74d12604455f0365a0a12439cf
    git -C "$release_dir" submodule update --init --recursive --quiet

    rm -rf "$workdir"
    mkdir -p "$workdir/lib" "$workdir/src"
    cp -R "$release_dir" "$workdir/lib/pau-administered-agent"
    write_aa_foundry_toml "$workdir/foundry.toml"

    forge build --root "$workdir" lib/pau-administered-agent/src/AdministeredAgentFactory.sol >/dev/null

    printf '%s\n' "$workdir"
}

verify_administered_agent_factory() {
    if ! should_run "ADMINISTERED_AGENT_FACTORY"; then
        return 0
    fi

    if [[ "$SKIP_AA" == "1" ]]; then
        echo "SKIP: ADMINISTERED_AGENT_FACTORY because SKIP_AA=1"
        return 0
    fi

    local workdir
    workdir="$(prepare_aa_workdir)"

    local tmp
    tmp="$(mktemp "$VERIFY_DIR/pau-aa-bytecode.XXXXXX")"

    echo "==> ADMINISTERED_AGENT_FACTORY 0x2968c3b5478cF93B70aB1e24255d4EDBBd27a089 pau-administered-agent/AdministeredAgentFactory.sol:AdministeredAgentFactory"
    if ! forge verify-bytecode \
        --root "$workdir" \
        --rpc-url "$RPC_URL" \
        --block "$VERIFY_BLOCK" \
        0x2968c3b5478cF93B70aB1e24255d4EDBBd27a089 \
        pau-administered-agent/AdministeredAgentFactory.sol:AdministeredAgentFactory >"$tmp" 2>&1
    then
        cat "$tmp"
        rm -f "$tmp"
        echo "FAIL: ADMINISTERED_AGENT_FACTORY forge verify-bytecode exited nonzero" >&2
        return 1
    fi

    require_full_match "ADMINISTERED_AGENT_FACTORY" "$tmp"
    rm -f "$tmp"
}

# Registry-level constants from sky-pau-registry Ethereum.sol and verification.md.
BEACON=0x829dC2b7E94B1954F0764E573f2E0d45Afa28199
PAU_FACTORY=0x69A5d548830AC2A4Ba90A44a2C75BDA71f97fc66
DEPLOYER=0x1ca4ECaF0E13ca833c80dA835DEEa15e1684361d

DAI=0x6B175474E89094C44Da98b954EedeAC495271d0F
USDC=0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48
USDE=0x4c9EDD5852cd905f086C759E8383e09bff1E68B3
USDS=0xdC035D45d973E3EC169d2276DDab16f1e407384F
SUSDE=0x9D39A5DE30e57443BfF2A8307A4256c8797A3497
USTB=0x43415eB6ff9DB7E26A15b704e7A3eDCe97d31C4e
WEETH=0xCd5fE23C85820F7B72D0926FC9b05b43E359b7ee
WETH=0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2
WSTETH=0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0
DAI_USDS=0x3225737a9Bbb6473CB4a45b7244ACa2BeFdB276A
PSM=0xf6e72Db5454dd049d0788e411b06CfAF16853042
CCTP_TOKEN_MESSENGER=0x28b5a0e9C621a5BadaA536219b3a228C8168cf5d
ETHENA_MINTER=0xe3490297a08d6fC8Da46Edb7B6142E4F461b62D3
WSTETH_WITHDRAW_QUEUE=0x889edC2eDab5f40e902b864aD4d7AdE8E412F9B1
PENDLE_ROUTER=0x888888888889758F76e7103c6CbF23ABbF58F946
UNISWAP_V3_POSITION_MANAGER=0xC36442b4a4522E871399CD717aBDD847Ab11FE88
UNISWAP_V3_ROUTER=0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45
PERMIT2=0x000000000022D473030F116dDEE9F6B43aC78BA3
UNISWAP_V4_POSITION_MANAGER=0xbD216513d74C8cf14cf4747E6AaA6420FF64ee9e
UNISWAP_V4_ROUTER=0x66a9893cC07D91D95644AEDD05D03f95e1dBA8Af

if [[ "$SKIP_REGISTRY_CONSTANTS" != "1" ]]; then
    "$SCRIPT_DIR/verify-sky-pau-registry-constants.sh" "$REGISTRY_FILE"
    echo
fi

ensure_diamond_pau_is_comparable
DIAMOND_PAU_BUILD_ROOT="$(prepare_diamond_pau_workdir)"
target_commit="$(git -C "$DIAMOND_PAU_BUILD_ROOT/lib/diamond-pau" rev-parse HEAD)"

echo "diamond-pau tag: $TAG ($target_commit)"
echo "diamond-pau root: $DIAMOND_PAU_BUILD_ROOT"
echo "block:           $VERIFY_BLOCK"
echo

verify_one BEACON                 "$BEACON"                                      diamond-pau/Beacon.sol:Beacon "$DEPLOYER"
verify_one PAU_FACTORY            "$PAU_FACTORY"                                 diamond-pau/PAUFactory.sol:PAUFactory "$BEACON"
verify_one AAVE_FACET             0x8CE890A96a193ff2DD4B2eA3C682326F655f6b62      diamond-pau/facets/aave/AaveFacet.sol:AaveFacet
verify_one BASIN_FACET            0xC84825BCD13AEddc372400239499380376a44A39      diamond-pau/facets/basin/BasinFacet.sol:BasinFacet
verify_one CCTP_FACET             0xADf62692340e46EF90336f2e75ce3b37f1148873      diamond-pau/facets/cctp/CCTPFacet.sol:CCTPFacet "$CCTP_TOKEN_MESSENGER" "$USDC"
verify_one CENTRIFUGE_FACET       0xa0A10BA97be1412730D694B8dE1afe7eff20eC31      diamond-pau/facets/centrifuge/CentrifugeFacet.sol:CentrifugeFacet
verify_one CURVE_FACET            0x139D81d7d6040fAeF7cF0EF5A2636Ca8a97a30d8      diamond-pau/facets/curve/CurveFacet.sol:CurveFacet
verify_one DAIUSDS_FACET          0x3817F734CAe6AD2BDb79F9ff23091F2AD478da5F      diamond-pau/facets/dai-usds/DAIUSDSFacet.sol:DAIUSDSFacet "$DAI" "$DAI_USDS" "$USDS"
verify_one ERC4626_FACET          0x1dCA18608c89174181153E786778705b4A0E1a06      diamond-pau/facets/erc4626/ERC4626Facet.sol:ERC4626Facet
verify_one ERC7540_FACET          0x4f7e0E3612b0e1E156A2B6570a51d4BD709F1315      diamond-pau/facets/erc7540/ERC7540Facet.sol:ERC7540Facet
verify_one ETHENA_FACET           0xEc48D773CEef1c6b07CdA1afA2716C478b55187B      diamond-pau/facets/ethena/EthenaFacet.sol:EthenaFacet "$ETHENA_MINTER" "$SUSDE" "$USDC" "$USDE"
verify_one FARM_FACET             0xF24E91f5D8529436c9fB92dd94F80d4A6C25d0f0      diamond-pau/facets/farm/FarmFacet.sol:FarmFacet
verify_one LAYER_ZERO_FACET       0xA0c323a0acb20F259eA4ff343319D450BE6472e5      diamond-pau/facets/layer-zero/LayerZeroFacet.sol:LayerZeroFacet
verify_one MAPLE_FACET            0x691b5c26aD2B74d2376f4eD87904E9D3E47bD630      diamond-pau/facets/maple/MapleFacet.sol:MapleFacet
verify_one MERKL_FACET            0x321138Db5E056e9d0080D4c278e10A1EdC091Eb0      diamond-pau/facets/merkl/MerklFacet.sol:MerklFacet
verify_one OTC_FACET              0x46b24ba00B65CB4f603447590e539b08097fb7Ac      diamond-pau/facets/otc/OTCFacet.sol:OTCFacet
verify_one PENDLE_FACET           0xcC9dD4c9B2a9c08f2692e7060F43d29A03E87348      diamond-pau/facets/pendle/PendleFacet.sol:PendleFacet "$PENDLE_ROUTER"
verify_one PSM_FACET              0xE4A5dAc768a310cc2316f258901b32E499653064      diamond-pau/facets/psm/PSMFacet.sol:PSMFacet "$DAI" "$DAI_USDS" "$PSM" "$USDC" "$USDS"
verify_one SPARK_VAULT_FACET      0xff0d19920E207e3A17eb5A2E5bA3AFA44836362b      diamond-pau/facets/spark-vault/SparkVaultFacet.sol:SparkVaultFacet
verify_one SUPERSTATE_FACET       0xeE197475607E9a27cCAA4786e740d2F0d0E706A7      diamond-pau/facets/superstate/SuperstateFacet.sol:SuperstateFacet "$USDC" "$USTB"
verify_one TRANSFER_ASSET_FACET   0x4DA7608C331b8f135df5b985018933780eCd089D      diamond-pau/facets/transfer-asset/TransferAssetFacet.sol:TransferAssetFacet
verify_one UNISWAP_V3_FACET       0x445D9Dc752F269Be48250f1A180CAC4c61cE4bab      diamond-pau/facets/uniswap-v3/UniswapV3Facet.sol:UniswapV3Facet "$UNISWAP_V3_POSITION_MANAGER" "$UNISWAP_V3_ROUTER"
verify_one UNISWAP_V4_FACET       0x75D35ffB8e6B871E12EB549CcF6afD324c46E47D      diamond-pau/facets/uniswap-v4/UniswapV4Facet.sol:UniswapV4Facet "$PERMIT2" "$UNISWAP_V4_POSITION_MANAGER" "$UNISWAP_V4_ROUTER"
verify_one USDS_FACET             0x1221CC4B85Ab260660aD21C2829e0EB516dffBc7      diamond-pau/facets/usds/USDSFacet.sol:USDSFacet "$USDS"
verify_one WEETH_FACET            0x1d8D089EB7D558F5dc6aA0cf98DDe13B77b3F641      diamond-pau/facets/weeth/WEETHFacet.sol:WEETHFacet "$WEETH" "$WETH"
verify_one WRAP_PROXY_ETH_FACET   0x081506DE21C695Af5e61a81aD288C8A96B6b59B9      diamond-pau/facets/wrap-proxy-eth/WrapProxyETHFacet.sol:WrapProxyETHFacet "$WETH"
verify_one WSTETH_FACET           0x3a82D11Cd37Fb0098363262Dc69425d07Fa05516      diamond-pau/facets/wsteth/WSTETHFacet.sol:WSTETHFacet "$WETH" "$WSTETH_WITHDRAW_QUEUE" "$WSTETH"
verify_administered_agent_factory

echo
echo "PASS: selected Sky PAU registry bytecode entries fully match"
SKY_PAU_BYTECODE_SH

cat > "$VERIFY_DIR/verification_scripts/verify-sky-pau-registry-bytecode-matrix.sh" <<'SKY_PAU_MATRIX_SH'
#!/usr/bin/env bash
set -euo pipefail

usage() {
    cat <<'USAGE'
Usage:
  MAINNET_RPC_URL=<rpc> verification_scripts/verify-sky-pau-registry-bytecode-matrix.sh [tag ...]

Default tags:
  v1.13.0 v1.1.0

This is the registry-level matrix for the Sky PAU registry verification.
Comparable tags are bytecode-verified against the Sky PAU registry addresses.
Old-layout tags, such as v1.1.0, are reported as NOT_COMPARABLE because they predate Beacon/facets.
USAGE
}

if [[ "${1:-}" == "-h" || "${1:-}" == "--help" ]]; then
    usage
    exit 0
fi

if (( $# == 0 )); then
    set -- v1.13.0 v1.1.0
fi

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
verifier="$SCRIPT_DIR/verify-sky-pau-registry-bytecode.sh"

overall=0

for requested_tag in "$@"; do
    echo "================================================================================"
    echo "diamond-pau tag: $requested_tag"

    set +e
    "$verifier" "$requested_tag"
    rc=$?
    set -e

    if (( rc == 20 )); then
        continue
    fi

    if (( rc != 0 )); then
        echo "FAIL: $requested_tag verification exited with status $rc"
        overall=1
        continue
    fi
done

echo "================================================================================"

if (( overall != 0 )); then
    echo "FAIL: one or more comparable tags were not verified"
    exit "$overall"
fi

echo "DONE: comparable tags verified; non-comparable old-layout tags were reported explicitly"
SKY_PAU_MATRIX_SH

cat > "$VERIFY_DIR/verification_scripts/verify-sky-pau-registry-wiring.sh" <<'SKY_PAU_WIRING_SH'
#!/usr/bin/env bash
set -euo pipefail

usage() {
    cat <<'USAGE'
Usage:
  MAINNET_RPC_URL=<rpc> verification_scripts/verify-sky-pau-registry-wiring.sh

Optional env:
  ETH_RPC_URL                 fallback RPC env var if MAINNET_RPC_URL is unset

Checks:
  - PAU_FACTORY.beacon() equals BEACON
  - Beacon admin role state
  - facet dependency getters
  - external dependency code and token metadata
  - Beacon integrations, wire counts, and dispatch self-consistency
  - AdministeredAgentFactory deploy(address) eth_call returns nonzero and no owner/admin/beacon getter is exposed
USAGE
}

if [[ "${1:-}" == "-h" || "${1:-}" == "--help" ]]; then
    usage
    exit 0
fi

RPC_URL="${MAINNET_RPC_URL:-${ETH_RPC_URL:-}}"

if [[ -z "$RPC_URL" ]]; then
    echo "ERROR: set MAINNET_RPC_URL or ETH_RPC_URL" >&2
    exit 2
fi

if [[ "$RPC_URL" == "<mainnet-rpc>" ]]; then
    echo "ERROR: replace <mainnet-rpc> with an Ethereum mainnet RPC URL" >&2
    exit 2
fi

for cmd in cast grep head sed wc tr; do
    if ! command -v "$cmd" >/dev/null 2>&1; then
        echo "ERROR: $cmd is required" >&2
        exit 2
    fi
done

BEACON=0x829dC2b7E94B1954F0764E573f2E0d45Afa28199
PAU_FACTORY=0x69A5d548830AC2A4Ba90A44a2C75BDA71f97fc66
ADMINISTERED_AGENT_FACTORY=0x2968c3b5478cF93B70aB1e24255d4EDBBd27a089
DEPLOYER=0x1ca4ECaF0E13ca833c80dA835DEEa15e1684361d
ADMIN=0xBE8E3e3618f7474F8cB1d074A26afFef007E98FB
DEFAULT_ADMIN_ROLE=0x0000000000000000000000000000000000000000000000000000000000000000

AAVE_FACET=0x8CE890A96a193ff2DD4B2eA3C682326F655f6b62
BASIN_FACET=0xC84825BCD13AEddc372400239499380376a44A39
CCTP_FACET=0xADf62692340e46EF90336f2e75ce3b37f1148873
CENTRIFUGE_FACET=0xa0A10BA97be1412730D694B8dE1afe7eff20eC31
CURVE_FACET=0x139D81d7d6040fAeF7cF0EF5A2636Ca8a97a30d8
DAIUSDS_FACET=0x3817F734CAe6AD2BDb79F9ff23091F2AD478da5F
ERC4626_FACET=0x1dCA18608c89174181153E786778705b4A0E1a06
ERC7540_FACET=0x4f7e0E3612b0e1E156A2B6570a51d4BD709F1315
ETHENA_FACET=0xEc48D773CEef1c6b07CdA1afA2716C478b55187B
FARM_FACET=0xF24E91f5D8529436c9fB92dd94F80d4A6C25d0f0
LAYER_ZERO_FACET=0xA0c323a0acb20F259eA4ff343319D450BE6472e5
MAPLE_FACET=0x691b5c26aD2B74d2376f4eD87904E9D3E47bD630
MERKL_FACET=0x321138Db5E056e9d0080D4c278e10A1EdC091Eb0
OTC_FACET=0x46b24ba00B65CB4f603447590e539b08097fb7Ac
PENDLE_FACET=0xcC9dD4c9B2a9c08f2692e7060F43d29A03E87348
PSM_FACET=0xE4A5dAc768a310cc2316f258901b32E499653064
SPARK_VAULT_FACET=0xff0d19920E207e3A17eb5A2E5bA3AFA44836362b
SUPERSTATE_FACET=0xeE197475607E9a27cCAA4786e740d2F0d0E706A7
TRANSFER_ASSET_FACET=0x4DA7608C331b8f135df5b985018933780eCd089D
UNISWAP_V3_FACET=0x445D9Dc752F269Be48250f1A180CAC4c61cE4bab
UNISWAP_V4_FACET=0x75D35ffB8e6B871E12EB549CcF6afD324c46E47D
USDS_FACET=0x1221CC4B85Ab260660aD21C2829e0EB516dffBc7
WEETH_FACET=0x1d8D089EB7D558F5dc6aA0cf98DDe13B77b3F641
WRAP_PROXY_ETH_FACET=0x081506DE21C695Af5e61a81aD288C8A96B6b59B9
WSTETH_FACET=0x3a82D11Cd37Fb0098363262Dc69425d07Fa05516

USDC=0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48
DAI=0x6B175474E89094C44Da98b954EedeAC495271d0F
USDS=0xdC035D45d973E3EC169d2276DDab16f1e407384F
DAI_USDS=0x3225737a9Bbb6473CB4a45b7244ACa2BeFdB276A
PSM=0xf6e72Db5454dd049d0788e411b06CfAF16853042
WETH=0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2
WSTETH=0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0
ETHENA_MINTER=0xe3490297a08d6fC8Da46Edb7B6142E4F461b62D3
USDE=0x4c9EDD5852cd905f086C759E8383e09bff1E68B3
SUSDE=0x9D39A5DE30e57443BfF2A8307A4256c8797A3497
PENDLE_ROUTER=0x888888888889758F76e7103c6CbF23ABbF58F946
UNISWAP_V3_POSITION_MANAGER=0xC36442b4a4522E871399CD717aBDD847Ab11FE88
UNISWAP_V3_ROUTER=0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45
PERMIT2=0x000000000022D473030F116dDEE9F6B43aC78BA3
UNISWAP_V4_POSITION_MANAGER=0xbD216513d74C8cf14cf4747E6AaA6420FF64ee9e
UNISWAP_V4_ROUTER=0x66a9893cC07D91D95644AEDD05D03f95e1dBA8Af
CCTP_TOKEN_MESSENGER=0x28b5a0e9C621a5BadaA536219b3a228C8168cf5d
WSTETH_WITHDRAW_QUEUE=0x889edC2eDab5f40e902b864aD4d7AdE8E412F9B1
USTB=0x43415eB6ff9DB7E26A15b704e7A3eDCe97d31C4e
WEETH=0xCd5fE23C85820F7B72D0926FC9b05b43E359b7ee

lc() {
    printf '%s' "$1" | tr '[:upper:]' '[:lower:]'
}

assert_eq() {
    local label="$1"
    local actual="$2"
    local expected="$3"

    if [[ "$actual" != "$expected" ]]; then
        echo "FAIL: $label" >&2
        echo "  expected: $expected" >&2
        echo "  actual:   $actual" >&2
        exit 1
    fi

    echo "pass: $label"
}

assert_addr_eq() {
    local label="$1"
    local actual="$2"
    local expected="$3"
    assert_eq "$label" "$(lc "$actual")" "$(lc "$expected")"
}

call() {
    cast call --rpc-url "$RPC_URL" "$@"
}

assert_address_call() {
    local label="$1"
    local target="$2"
    local signature="$3"
    local expected="$4"
    local actual

    actual="$(call "$target" "$signature")"
    assert_addr_eq "$label" "$actual" "$expected"
}

assert_bool_call() {
    local label="$1"
    local target="$2"
    local signature="$3"
    local expected="$4"
    shift 4
    local actual

    actual="$(call "$target" "$signature" "$@")"
    assert_eq "$label" "$actual" "$expected"
}

assert_code() {
    local label="$1"
    local target="$2"
    local code

    code="$(cast code --rpc-url "$RPC_URL" "$target")"
    if [[ "$code" == "0x" || -z "$code" ]]; then
        echo "FAIL: $label has no code" >&2
        exit 1
    fi

    echo "pass: $label code"
}

assert_token() {
    local label="$1"
    local target="$2"
    local expected_symbol="$3"
    local expected_decimals="$4"
    local symbol
    local decimals

    symbol="$(call "$target" 'symbol()(string)')"
    symbol="${symbol%\"}"
    symbol="${symbol#\"}"
    decimals="$(call "$target" 'decimals()(uint8)')"

    assert_eq "$label symbol" "$symbol" "$expected_symbol"
    assert_eq "$label decimals" "$decimals" "$expected_decimals"
}

extract_first_address() {
    grep -oE '0x[0-9a-fA-F]{40}' | head -1
}

extract_first_selector() {
    grep -oE '0x[0-9a-fA-F]{8}' | head -1
}

extract_second_selector() {
    grep -oE '0x[0-9a-fA-F]{8}' | sed -n '2p'
}

total_wires=0

assert_integration() {
    local id="$1"
    local expected_facet="$2"
    local expected_wire_count="$3"
    local id_bytes32
    local cfg
    local facet
    local wire_count
    local wires

    id_bytes32="$(cast format-bytes32-string "$id")"
    cfg="$(call "$BEACON" 'getConfig(bytes32)((address,(bytes4,bytes4)[]))' "$id_bytes32")"
    facet="$(printf '%s' "$cfg" | extract_first_address)"
    assert_addr_eq "$id facet" "$facet" "$expected_facet"

    wires="$(printf '%s' "$cfg" | grep -oE '\(0x[0-9a-fA-F]{8}, 0x[0-9a-fA-F]{8}\)' || true)"
    if [[ -z "$wires" ]]; then
        wire_count=0
    else
        wire_count="$(printf '%s\n' "$wires" | wc -l | tr -d ' ')"
    fi

    assert_eq "$id wire count" "$wire_count" "$expected_wire_count"
    total_wires=$((total_wires + wire_count))

    while IFS= read -r wire; do
        [[ -z "$wire" ]] && continue

        local call_selector
        local delegate_selector
        local dispatch
        local dispatch_facet
        local dispatch_delegate

        call_selector="$(printf '%s' "$wire" | extract_first_selector)"
        delegate_selector="$(printf '%s' "$wire" | extract_second_selector)"
        dispatch="$(call "$BEACON" 'getDispatch(bytes4)((address,bytes4))' "$call_selector")"
        dispatch_facet="$(printf '%s' "$dispatch" | extract_first_address)"
        dispatch_delegate="$(printf '%s' "$dispatch" | extract_second_selector)"

        assert_addr_eq "$id dispatch $call_selector facet" "$dispatch_facet" "$expected_facet"
        assert_eq "$id dispatch $call_selector delegate" "$dispatch_delegate" "$delegate_selector"
    done <<< "$wires"
}

echo "==> factory and admin"
assert_address_call 'PAU_FACTORY.beacon()' "$PAU_FACTORY" 'beacon()(address)' "$BEACON"
assert_bool_call 'Beacon admin has DEFAULT_ADMIN_ROLE' "$BEACON" 'hasRole(bytes32,address)(bool)' true "$DEFAULT_ADMIN_ROLE" "$ADMIN"
assert_bool_call 'Beacon deployer lacks DEFAULT_ADMIN_ROLE' "$BEACON" 'hasRole(bytes32,address)(bool)' false "$DEFAULT_ADMIN_ROLE" "$DEPLOYER"
assert_eq 'Beacon DEFAULT_ADMIN_ROLE member count' "$(call "$BEACON" 'getRoleMemberCount(bytes32)(uint256)' "$DEFAULT_ADMIN_ROLE")" 1
assert_addr_eq 'Beacon DEFAULT_ADMIN_ROLE member[0]' "$(call "$BEACON" 'getRoleMember(bytes32,uint256)(address)' "$DEFAULT_ADMIN_ROLE" 0)" "$ADMIN"

echo
echo "==> facet dependency getters"
assert_address_call 'CCTP_FACET.cctp()' "$CCTP_FACET" 'cctp()(address)' "$CCTP_TOKEN_MESSENGER"
assert_address_call 'CCTP_FACET.usdc()' "$CCTP_FACET" 'usdc()(address)' "$USDC"
assert_address_call 'DAIUSDS_FACET.dai()' "$DAIUSDS_FACET" 'dai()(address)' "$DAI"
assert_address_call 'DAIUSDS_FACET.daiUSDS()' "$DAIUSDS_FACET" 'daiUSDS()(address)' "$DAI_USDS"
assert_address_call 'DAIUSDS_FACET.usds()' "$DAIUSDS_FACET" 'usds()(address)' "$USDS"
assert_address_call 'ETHENA_FACET.minter()' "$ETHENA_FACET" 'minter()(address)' "$ETHENA_MINTER"
assert_address_call 'ETHENA_FACET.susde()' "$ETHENA_FACET" 'susde()(address)' "$SUSDE"
assert_address_call 'ETHENA_FACET.usdc()' "$ETHENA_FACET" 'usdc()(address)' "$USDC"
assert_address_call 'ETHENA_FACET.usde()' "$ETHENA_FACET" 'usde()(address)' "$USDE"
assert_address_call 'PENDLE_FACET.router()' "$PENDLE_FACET" 'router()(address)' "$PENDLE_ROUTER"
assert_address_call 'PSM_FACET.dai()' "$PSM_FACET" 'dai()(address)' "$DAI"
assert_address_call 'PSM_FACET.daiUSDS()' "$PSM_FACET" 'daiUSDS()(address)' "$DAI_USDS"
assert_address_call 'PSM_FACET.psm()' "$PSM_FACET" 'psm()(address)' "$PSM"
assert_address_call 'PSM_FACET.usdc()' "$PSM_FACET" 'usdc()(address)' "$USDC"
assert_address_call 'PSM_FACET.usds()' "$PSM_FACET" 'usds()(address)' "$USDS"
assert_address_call 'SUPERSTATE_FACET.usdc()' "$SUPERSTATE_FACET" 'usdc()(address)' "$USDC"
assert_address_call 'SUPERSTATE_FACET.ustb()' "$SUPERSTATE_FACET" 'ustb()(address)' "$USTB"
assert_address_call 'UNISWAP_V3_FACET.positionManager()' "$UNISWAP_V3_FACET" 'positionManager()(address)' "$UNISWAP_V3_POSITION_MANAGER"
assert_address_call 'UNISWAP_V3_FACET.router()' "$UNISWAP_V3_FACET" 'router()(address)' "$UNISWAP_V3_ROUTER"
assert_address_call 'UNISWAP_V4_FACET.permit2()' "$UNISWAP_V4_FACET" 'permit2()(address)' "$PERMIT2"
assert_address_call 'UNISWAP_V4_FACET.positionManager()' "$UNISWAP_V4_FACET" 'positionManager()(address)' "$UNISWAP_V4_POSITION_MANAGER"
assert_address_call 'UNISWAP_V4_FACET.router()' "$UNISWAP_V4_FACET" 'router()(address)' "$UNISWAP_V4_ROUTER"
assert_address_call 'USDS_FACET.usds()' "$USDS_FACET" 'usds()(address)' "$USDS"
assert_address_call 'WEETH_FACET.weeth()' "$WEETH_FACET" 'weeth()(address)' "$WEETH"
assert_address_call 'WEETH_FACET.weth()' "$WEETH_FACET" 'weth()(address)' "$WETH"
assert_address_call 'WRAP_PROXY_ETH_FACET.weth()' "$WRAP_PROXY_ETH_FACET" 'weth()(address)' "$WETH"
assert_address_call 'WSTETH_FACET.weth()' "$WSTETH_FACET" 'weth()(address)' "$WETH"
assert_address_call 'WSTETH_FACET.withdrawQueue()' "$WSTETH_FACET" 'withdrawQueue()(address)' "$WSTETH_WITHDRAW_QUEUE"
assert_address_call 'WSTETH_FACET.wsteth()' "$WSTETH_FACET" 'wsteth()(address)' "$WSTETH"

echo
echo "==> external dependency code and token metadata"
assert_code USDC "$USDC"
assert_code DAI "$DAI"
assert_code USDS "$USDS"
assert_code DAI_USDS "$DAI_USDS"
assert_code PSM "$PSM"
assert_code WETH "$WETH"
assert_code WSTETH "$WSTETH"
assert_code ETHENA_MINTER "$ETHENA_MINTER"
assert_code USDE "$USDE"
assert_code SUSDE "$SUSDE"
assert_code PENDLE_ROUTER "$PENDLE_ROUTER"
assert_code UNISWAP_V3_ROUTER "$UNISWAP_V3_ROUTER"
assert_code UNISWAP_V3_POSITION_MANAGER "$UNISWAP_V3_POSITION_MANAGER"
assert_code UNISWAP_V4_ROUTER "$UNISWAP_V4_ROUTER"
assert_code UNISWAP_V4_POSITION_MANAGER "$UNISWAP_V4_POSITION_MANAGER"
assert_code PERMIT2 "$PERMIT2"
assert_code CCTP_TOKEN_MESSENGER "$CCTP_TOKEN_MESSENGER"
assert_code WSTETH_WITHDRAW_QUEUE "$WSTETH_WITHDRAW_QUEUE"
assert_code USTB "$USTB"
assert_code WEETH "$WEETH"

assert_token USDC "$USDC" USDC 6
assert_token DAI "$DAI" DAI 18
assert_token USDS "$USDS" USDS 18
assert_token USDE "$USDE" USDe 18
assert_token SUSDE "$SUSDE" sUSDe 18
assert_token USTB "$USTB" USTB 6
assert_token WEETH "$WEETH" weETH 18
assert_token WETH "$WETH" WETH 18
assert_token WSTETH "$WSTETH" wstETH 18

echo
echo "==> beacon integrations and dispatches"
integration_output="$(call "$BEACON" 'integrations()((bytes32,(address,(bytes4,bytes4)[]))[])')"
integration_count="$(printf '%s' "$integration_output" | grep -oE '0x[0-9a-fA-F]{64}, \(' | wc -l | tr -d ' ')"
assert_eq 'Beacon integrations length' "$integration_count" 25

assert_integration AAVE_FACET "$AAVE_FACET" 7
assert_integration BASIN_FACET "$BASIN_FACET" 5
assert_integration CCTP_FACET "$CCTP_FACET" 10
assert_integration CENTRIFUGE_FACET "$CENTRIFUGE_FACET" 14
assert_integration CURVE_FACET "$CURVE_FACET" 11
assert_integration DAIUSDS_FACET "$DAIUSDS_FACET" 8
assert_integration ERC4626_FACET "$ERC4626_FACET" 9
assert_integration ERC7540_FACET "$ERC7540_FACET" 9
assert_integration ETHENA_FACET "$ETHENA_FACET" 18
assert_integration FARM_FACET "$FARM_FACET" 7
assert_integration LAYER_ZERO_FACET "$LAYER_ZERO_FACET" 6
assert_integration MAPLE_FACET "$MAPLE_FACET" 5
assert_integration MERKL_FACET "$MERKL_FACET" 3
assert_integration OTC_FACET "$OTC_FACET" 14
assert_integration PENDLE_FACET "$PENDLE_FACET" 4
assert_integration PSM_FACET "$PSM_FACET" 11
assert_integration SPARK_VAULT_FACET "$SPARK_VAULT_FACET" 3
assert_integration SUPERSTATE_FACET "$SUPERSTATE_FACET" 5
assert_integration TRANSFER_ASSET_FACET "$TRANSFER_ASSET_FACET" 3
assert_integration UNISWAP_V3_FACET "$UNISWAP_V3_FACET" 23
assert_integration UNISWAP_V4_FACET "$UNISWAP_V4_FACET" 17
assert_integration USDS_FACET "$USDS_FACET" 8
assert_integration WEETH_FACET "$WEETH_FACET" 9
assert_integration WRAP_PROXY_ETH_FACET "$WRAP_PROXY_ETH_FACET" 4
assert_integration WSTETH_FACET "$WSTETH_FACET" 10
assert_eq 'Beacon total dispatch wires checked' "$total_wires" 223

echo
echo "==> administered-agent factory"
factory_code="$(cast code --rpc-url "$RPC_URL" "$ADMINISTERED_AGENT_FACTORY")"
if [[ "$factory_code" == "0x" || -z "$factory_code" ]]; then
    echo "FAIL: ADMINISTERED_AGENT_FACTORY has no code" >&2
    exit 1
fi

simulated_agent="$(call "$ADMINISTERED_AGENT_FACTORY" 'deploy(address)(address)' 0x000000000000000000000000000000000000dEaD)"
if [[ "$(lc "$simulated_agent")" == "0x0000000000000000000000000000000000000000" ]]; then
    echo "FAIL: ADMINISTERED_AGENT_FACTORY.deploy(address) returned zero address in eth_call" >&2
    exit 1
fi
echo "pass: ADMINISTERED_AGENT_FACTORY.deploy(address) eth_call returned $simulated_agent"

for getter in 'owner()(address)' 'admin()(address)' 'beacon()(address)'; do
    if call "$ADMINISTERED_AGENT_FACTORY" "$getter" >/dev/null 2>&1; then
        echo "FAIL: ADMINISTERED_AGENT_FACTORY unexpectedly exposes $getter" >&2
        exit 1
    fi
    echo "pass: ADMINISTERED_AGENT_FACTORY no $getter"
done

echo
echo "PASS: Sky PAU registry wiring checks matched"
SKY_PAU_WIRING_SH

chmod +x "$VERIFY_DIR"/verification_scripts/verify-sky-pau-registry-*.sh

if [[ ! -d "$VERIFY_DIR/sky-pau-registry/.git" ]]; then
  git clone https://github.com/sky-ecosystem/sky-pau-registry.git "$VERIFY_DIR/sky-pau-registry"
fi
git -C "$VERIFY_DIR/sky-pau-registry" fetch --quiet --tags
git -C "$VERIFY_DIR/sky-pau-registry" checkout --quiet 161bac0c17a7d2c4d4e0455e1febe401a7a36edb

REGISTRY_FILE="$VERIFY_DIR/sky-pau-registry/src/Ethereum.sol" \
  bash "$VERIFY_DIR/verification_scripts/verify-sky-pau-registry-constants.sh" \
  "$VERIFY_DIR/sky-pau-registry/src/Ethereum.sol"

MAINNET_RPC_URL="$MAINNET_RPC_URL" \
VERIFY_DIR="$VERIFY_DIR" \
REGISTRY_FILE="$VERIFY_DIR/sky-pau-registry/src/Ethereum.sol" \
DIAMOND_PAU_RELEASE_DIR="$VERIFY_DIR/diamond-pau-v1.13.0" \
DIAMOND_PAU_WORKDIR="$VERIFY_DIR/diamond-pau-parent-v1.13.0" \
AA_RELEASE_DIR="$VERIFY_DIR/pau-administered-agent-v1.0.0" \
AA_WORKDIR="$VERIFY_DIR/pau-aa-parent" \
SKIP_REGISTRY_CONSTANTS=1 \
  bash "$VERIFY_DIR/verification_scripts/verify-sky-pau-registry-bytecode.sh" v1.13.0

MAINNET_RPC_URL="$MAINNET_RPC_URL" \
VERIFY_DIR="$VERIFY_DIR" \
REGISTRY_FILE="$VERIFY_DIR/sky-pau-registry/src/Ethereum.sol" \
DIAMOND_PAU_RELEASE_DIR="$VERIFY_DIR/diamond-pau-v1.13.0" \
DIAMOND_PAU_WORKDIR="$VERIFY_DIR/diamond-pau-parent-v1.13.0" \
AA_RELEASE_DIR="$VERIFY_DIR/pau-administered-agent-v1.0.0" \
AA_WORKDIR="$VERIFY_DIR/pau-aa-parent" \
SKIP_REGISTRY_CONSTANTS=1 \
  bash "$VERIFY_DIR/verification_scripts/verify-sky-pau-registry-bytecode-matrix.sh" v1.13.0 v1.1.0

MAINNET_RPC_URL="$MAINNET_RPC_URL" \
  bash "$VERIFY_DIR/verification_scripts/verify-sky-pau-registry-wiring.sh"
```

Expected terminal results:

| Command | Expected result |
|---|---|
| `verify-sky-pau-registry-constants.sh` | `PASS: 28 Sky PAU registry constants match verified addresses` |
| `verify-sky-pau-registry-bytecode.sh v1.13.0` | all checked entries print creation `full` and runtime `full` |
| `verify-sky-pau-registry-bytecode-matrix.sh v1.13.0 v1.1.0` | v1.13.0 verified; old-layout v1.1.0 reported `NOT_COMPARABLE` |
| `verify-sky-pau-registry-wiring.sh` | `PASS: Sky PAU registry wiring checks matched` |

## Wiring Verification

| Check | Result |
|---|---:|
| `PAU_FACTORY.beacon()` equals `BEACON` (`0x829dC2b7E94B1954F0764E573f2E0d45Afa28199`) | pass |
| Beacon `DEFAULT_ADMIN_ROLE` has exactly one member, `0xBE8E3e3618f7474F8cB1d074A26afFef007E98FB` (Sky `MCD_PAUSE_PROXY`) | pass |
| Beacon role event replay saw only `DEFAULT_ADMIN_ROLE`; no `RoleAdminChanged` events | pass |
| Beacon integration facets match registry facet addresses | 25 / 25 |
| Beacon integration wire arrays match deployment script expectation | 25 / 25 |
| Beacon selector dispatches match expected facet + delegate selector | 223 / 223 |
| Constructor-wired facet dependency getters match expected addresses | 29 / 29 |
| Reference wiring source is a clean `diamond-pau@v1.13.0` checkout, not the vendored deploy repo submodule | pass |
| `ADMINISTERED_AGENT_FACTORY.deploy(admin)` creates an agent with exactly one initial admin, the supplied `admin` | pass |
| `ADMINISTERED_AGENT_FACTORY` exposes no persistent `owner()`, `admin()`, or `beacon()` address getter | pass |
| `test_postDeployEvents` default script status | skipped unless `RUN_EVENTS=1` and `ETHERSCAN_API_KEY` or `MAINNET_API_KEY` is set |
| Wiring mismatches | 0 |

| Item | Result |
|---|---|
| Controller/proxy instances | not checked |
| Controller/proxy addresses in registry | none |

## Governance and Admin Identity

| Item | Address | Identity (Sky chainlog) |
|---|---|---|
| Beacon admin (`DEFAULT_ADMIN_ROLE`) | `0xBE8E3e3618f7474F8cB1d074A26afFef007E98FB` | `MCD_PAUSE_PROXY` (`DSPauseProxy`) |
| Beacon admin owner | `0xbE286431454714F511008713973d3B053A2d38f3` | `MCD_PAUSE` |
| Beacon deployer | `0x1ca4ECaF0E13ca833c80dA835DEEa15e1684361d` | no current role |

| Item | Result |
|---|---|
| `MCD_PAUSE_PROXY` | `0xBE8E3e3618f7474F8cB1d074A26afFef007E98FB` |
| `MCD_PAUSE` | `0xbE286431454714F511008713973d3B053A2d38f3` |
| `PAU_FACTORY` access | permissionless |
| `PAU_FACTORY.beacon()` | `0x829dC2b7E94B1954F0764E573f2E0d45Afa28199` |

## External Dependency Sanity

| Check | Result |
|---|---:|
| Unique external dependencies from facet getters | 20 |
| Nonzero code at block `25270766` | 20 / 20 |
| Nonzero code at latest during scan | 20 / 20 |
| Runtime code hash stable from block `25270766` to latest | 20 / 20 |
| Etherscan source verified | 20 / 20 |
| Etherscan proxy/self-implementation metadata entries reviewed | 10 |
| Direct token metadata checks | 9 / 9 |
| Failed dependency checks | 0 |

| Dependency getter | Address | Facet getter(s) |
|---|---|---|
| `usdc` | `USDC`: `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` | `CCTP_FACET.usdc()`<br>`ETHENA_FACET.usdc()`<br>`PSM_FACET.usdc()`<br>`SUPERSTATE_FACET.usdc()` |
| `dai` | `MCD_DAI`: `0x6B175474E89094C44Da98b954EedeAC495271d0F` | `DAIUSDS_FACET.dai()`<br>`PSM_FACET.dai()` |
| `usds` | `USDS`: `0xdC035D45d973E3EC169d2276DDab16f1e407384F` | `DAIUSDS_FACET.usds()`<br>`PSM_FACET.usds()`<br>`USDS_FACET.usds()` |
| `daiUSDS` | `DAI_USDS`: `0x3225737a9Bbb6473CB4a45b7244ACa2BeFdB276A` | `DAIUSDS_FACET.daiUSDS()`<br>`PSM_FACET.daiUSDS()` |
| `psm` | `MCD_LITE_PSM_USDC_A`: `0xf6e72Db5454dd049d0788e411b06CfAF16853042` | `PSM_FACET.psm()` |
| `weth` | `ETH`: `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` | `WEETH_FACET.weth()`<br>`WRAP_PROXY_ETH_FACET.weth()`<br>`WSTETH_FACET.weth()` |
| `wsteth` | `WSTETH`: `0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0` | `WSTETH_FACET.wsteth()` |
| Ethena minter / `USDe` / `sUSDe` | `Ethena Minting`: `0xe3490297a08d6fC8Da46Edb7B6142E4F461b62D3`<br>`USDe`: `0x4c9EDD5852cd905f086C759E8383e09bff1E68B3`<br>`sUSDe`: `0x9D39A5DE30e57443BfF2A8307A4256c8797A3497` | `ETHENA_FACET.minter()`<br>`ETHENA_FACET.usde()`<br>`ETHENA_FACET.susde()` |
| Pendle router | `Pendle Router`: `0x888888888889758F76e7103c6CbF23ABbF58F946` | `PENDLE_FACET.router()` |
| Uniswap V3/V4 routers and managers | `SwapRouter02`: `0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45`<br>`V3 NonfungiblePositionManager`: `0xC36442b4a4522E871399CD717aBDD847Ab11FE88`<br>`V4 UniversalRouter`: `0x66a9893cC07D91D95644AEDD05D03f95e1dBA8Af`<br>`V4 PositionManager`: `0xbD216513d74C8cf14cf4747E6AaA6420FF64ee9e` | `UNISWAP_V3_FACET.router()`<br>`UNISWAP_V3_FACET.positionManager()`<br>`UNISWAP_V4_FACET.router()`<br>`UNISWAP_V4_FACET.positionManager()` |
| Permit2 | `Permit2`: `0x000000000022D473030F116dDEE9F6B43aC78BA3` | `UNISWAP_V4_FACET.permit2()` |
| Circle CCTP TokenMessenger | `TokenMessenger`: `0x28b5a0e9C621a5BadaA536219b3a228C8168cf5d` | `CCTP_FACET.cctp()` |
| Lido WithdrawalQueueERC721 | `WithdrawalQueueERC721`: `0x889edC2eDab5f40e902b864aD4d7AdE8E412F9B1` | `WSTETH_FACET.withdrawQueue()` |
| USTB | `USTB`: `0x43415eB6ff9DB7E26A15b704e7A3eDCe97d31C4e` | `SUPERSTATE_FACET.ustb()` |
| weETH | `weETH`: `0xCd5fE23C85820F7B72D0926FC9b05b43E359b7ee` | `WEETH_FACET.weeth()` |

## Per-address Results

| Registry constant | Address | Checked against | Contract checked | Runtime keccak | Exact |
|---|---|---|---|---|---:|
| `BEACON` | `0x829dC2b7E94B1954F0764E573f2E0d45Afa28199` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [Beacon](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/Beacon.sol) | `0x6942fe0a7dfabe69099c5ac1078b5c3581b6bcb39b0a5fa6038117cb9ae38ec7` | yes |
| `PAU_FACTORY` | `0x69A5d548830AC2A4Ba90A44a2C75BDA71f97fc66` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [PAUFactory](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/PAUFactory.sol) | `0x0c64c1999a9eab8c3d273bba1dd53a76e4bb74df4e7c39e9e872b9313e92ceec` | yes |
| `AAVE_FACET` | `0x8CE890A96a193ff2DD4B2eA3C682326F655f6b62` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [AaveFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/aave/AaveFacet.sol) | `0x3157486bb8dd60e7899fba569c2cd6a649b9da92898cd6eb3409d1a63c945881` | yes |
| `BASIN_FACET` | `0xC84825BCD13AEddc372400239499380376a44A39` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [BasinFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/basin/BasinFacet.sol) | `0xaaa4e7d649c1876207f8fd07d5de9bacab19b096be07b4b15a55c4cef01f945d` | yes |
| `CCTP_FACET` | `0xADf62692340e46EF90336f2e75ce3b37f1148873` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [CCTPFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/cctp/CCTPFacet.sol) | `0x71c6b48e1adb4dfe5c4eab8117ff7e71239bbad74f31261aa719c9f15fa68b8f` | yes |
| `CENTRIFUGE_FACET` | `0xa0A10BA97be1412730D694B8dE1afe7eff20eC31` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [CentrifugeFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/centrifuge/CentrifugeFacet.sol) | `0x2e13cf572d0a5b74c97036ae09b6e288465234889d9f7b95232ef9e25c8cbf34` | yes |
| `CURVE_FACET` | `0x139D81d7d6040fAeF7cF0EF5A2636Ca8a97a30d8` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [CurveFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/curve/CurveFacet.sol) | `0x25357da7ab4b033969777de0d76ca5ac3853dc9bcf4fb824941d1773fec4e3cd` | yes |
| `DAIUSDS_FACET` | `0x3817F734CAe6AD2BDb79F9ff23091F2AD478da5F` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [DAIUSDSFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/dai-usds/DAIUSDSFacet.sol) | `0x9b9ce00d908f4d1c92f72da291b48db5a0cc52a4e90947ce873ed1830e667355` | yes |
| `ERC4626_FACET` | `0x1dCA18608c89174181153E786778705b4A0E1a06` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [ERC4626Facet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/erc4626/ERC4626Facet.sol) | `0xa3653b3e840684d7b5f3e02700280b1e2d6236587e39864f77c21cd903866916` | yes |
| `ERC7540_FACET` | `0x4f7e0E3612b0e1E156A2B6570a51d4BD709F1315` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [ERC7540Facet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/erc7540/ERC7540Facet.sol) | `0x377959f0409af06ff8926611707f102c1403055f47dc0ec7d2271924f13cf035` | yes |
| `ETHENA_FACET` | `0xEc48D773CEef1c6b07CdA1afA2716C478b55187B` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [EthenaFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/ethena/EthenaFacet.sol) | `0x87bbb809b29c98cb3fd929d2a747d84f2ccff9cbcc6f47104eb9a0223d4d14eb` | yes |
| `FARM_FACET` | `0xF24E91f5D8529436c9fB92dd94F80d4A6C25d0f0` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [FarmFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/farm/FarmFacet.sol) | `0x07ec20c8f1ca3487709e0300978ef46e3f45888c456c1f888bf9094e31d67894` | yes |
| `LAYER_ZERO_FACET` | `0xA0c323a0acb20F259eA4ff343319D450BE6472e5` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [LayerZeroFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/layer-zero/LayerZeroFacet.sol) | `0xa85e4a26ce588166912e32a7469c5495827d905777234a2d32cf255ffdc16cc2` | yes |
| `MAPLE_FACET` | `0x691b5c26aD2B74d2376f4eD87904E9D3E47bD630` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [MapleFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/maple/MapleFacet.sol) | `0x72071252b13d99aaac40d45177f3072a9a5a92c07bffcb3e838b8cddcf3c6b6a` | yes |
| `MERKL_FACET` | `0x321138Db5E056e9d0080D4c278e10A1EdC091Eb0` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [MerklFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/merkl/MerklFacet.sol) | `0xe8b533e3c99738bf0f9523ada447c76a3d77f51cdc2ecb82668fb2e328ecad68` | yes |
| `OTC_FACET` | `0x46b24ba00B65CB4f603447590e539b08097fb7Ac` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [OTCFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/otc/OTCFacet.sol) | `0x16d7cb7384f11f5a96e1266296287ad1b8d9d34b143378aea2665ccd537679aa` | yes |
| `PENDLE_FACET` | `0xcC9dD4c9B2a9c08f2692e7060F43d29A03E87348` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [PendleFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/pendle/PendleFacet.sol) | `0xe8805f0090daad6fe385fb1e88054c565beafe0c7be096030ce10390e67e101a` | yes |
| `PSM_FACET` | `0xE4A5dAc768a310cc2316f258901b32E499653064` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [PSMFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/psm/PSMFacet.sol) | `0xb17165681e669b85828cf26152ce8ebc5c0d487e91f4531cf93adf662a37e941` | yes |
| `SPARK_VAULT_FACET` | `0xff0d19920E207e3A17eb5A2E5bA3AFA44836362b` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [SparkVaultFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/spark-vault/SparkVaultFacet.sol) | `0xa155a82c1ad0f2697010e691f981c2c94d75565353ddc7a61df60ea2ae3cf155` | yes |
| `SUPERSTATE_FACET` | `0xeE197475607E9a27cCAA4786e740d2F0d0E706A7` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [SuperstateFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/superstate/SuperstateFacet.sol) | `0xbda7b457c2dca190ebf152eebf4572cb8d98ff26d3ca6025d15e444a7532c070` | yes |
| `TRANSFER_ASSET_FACET` | `0x4DA7608C331b8f135df5b985018933780eCd089D` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [TransferAssetFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/transfer-asset/TransferAssetFacet.sol) | `0xc09a16c2a7b120f19b65d2ad94bc6481fc0f1720ae9407393c0f38ddd23cb2c8` | yes |
| `UNISWAP_V3_FACET` | `0x445D9Dc752F269Be48250f1A180CAC4c61cE4bab` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [UniswapV3Facet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/uniswap-v3/UniswapV3Facet.sol) | `0x225d028286a5090b27a9590c63b80d486a6c6da3a6a6762932abae8d9bff9233` | yes |
| `UNISWAP_V4_FACET` | `0x75D35ffB8e6B871E12EB549CcF6afD324c46E47D` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [UniswapV4Facet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/uniswap-v4/UniswapV4Facet.sol) | `0x0789f012604a887e711bcbb4e86e73890e67960a6a90c553cad1ef501bed76e0` | yes |
| `USDS_FACET` | `0x1221CC4B85Ab260660aD21C2829e0EB516dffBc7` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [USDSFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/usds/USDSFacet.sol) | `0x60a29823fd0c123add0ab8b7abfa75f6c9d1a4d18acaf5b7e3414634a022e685` | yes |
| `WEETH_FACET` | `0x1d8D089EB7D558F5dc6aA0cf98DDe13B77b3F641` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [WEETHFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/weeth/WEETHFacet.sol) | `0xc82d8941bc0da453d2a50d54f92541ad2b0fe876136305d79026abb89866b516` | yes |
| `WRAP_PROXY_ETH_FACET` | `0x081506DE21C695Af5e61a81aD288C8A96B6b59B9` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [WrapProxyETHFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/wrap-proxy-eth/WrapProxyETHFacet.sol) | `0x3a82a316722d3146ea8f1cc1fbd965b48bac1cf062850762e197e5b7dbeae57f` | yes |
| `WSTETH_FACET` | `0x3a82D11Cd37Fb0098363262Dc69425d07Fa05516` | [diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0) | [WSTETHFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/wsteth/WSTETHFacet.sol) | `0x2464ee33b04c9d7d84e755521f02c12ae378f951b55707ece27f31f0f39b7e45` | yes |
| `ADMINISTERED_AGENT_FACTORY` | `0x2968c3b5478cF93B70aB1e24255d4EDBBd27a089` | [pau-administered-agent v1.0.0](https://github.com/sky-ecosystem/pau-administered-agent/releases/tag/v1.0.0) | [AdministeredAgentFactory](https://github.com/sky-ecosystem/pau-administered-agent/blob/bfaaf709a8664d74d12604455f0365a0a12439cf/src/AdministeredAgentFactory.sol) | `0x5ae519a20818627fb29bf44c613a3b6aea174ffe694803bafd1ad1ab8db0fe38` | yes |

## Constructor Args Used

| Registry constant | Contract | Constructor args |
|---|---|---|
| `BEACON` | [Beacon](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/Beacon.sol) | `0x1ca4ECaF0E13ca833c80da835deea15e1684361d` |
| `PAU_FACTORY` | [PAUFactory](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/PAUFactory.sol) | `0x829dC2b7E94B1954F0764E573f2E0d45Afa28199` |
| `CCTP_FACET` | [CCTPFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/cctp/CCTPFacet.sol) | `0x28b5a0e9C621a5BadaA536219b3a228C8168cf5d`<br>`0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` |
| `DAIUSDS_FACET` | [DAIUSDSFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/dai-usds/DAIUSDSFacet.sol) | `0x6B175474E89094C44Da98b954EedeAC495271d0F`<br>`0x3225737a9Bbb6473CB4a45b7244ACa2BeFdB276A`<br>`0xdC035D45d973E3EC169d2276DDab16f1e407384F` |
| `ETHENA_FACET` | [EthenaFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/ethena/EthenaFacet.sol) | `0xe3490297a08d6fC8Da46Edb7B6142E4F461b62D3`<br>`0x9D39A5DE30e57443BfF2A8307A4256c8797A3497`<br>`0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48`<br>`0x4c9EDD5852cd905f086C759E8383e09bff1E68B3` |
| `PENDLE_FACET` | [PendleFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/pendle/PendleFacet.sol) | `0x888888888889758F76e7103c6CbF23ABbF58F946` |
| `PSM_FACET` | [PSMFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/psm/PSMFacet.sol) | `0x6B175474E89094C44Da98b954EedeAC495271d0F`<br>`0x3225737a9Bbb6473CB4a45b7244ACa2BeFdB276A`<br>`0xf6e72Db5454dd049d0788e411b06CfAF16853042`<br>`0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48`<br>`0xdC035D45d973E3EC169d2276DDab16f1e407384F` |
| `SUPERSTATE_FACET` | [SuperstateFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/superstate/SuperstateFacet.sol) | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48`<br>`0x43415eB6ff9DB7E26A15b704e7A3eDCe97d31C4e` |
| `UNISWAP_V3_FACET` | [UniswapV3Facet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/uniswap-v3/UniswapV3Facet.sol) | `0xC36442b4a4522E871399CD717aBDD847Ab11FE88`<br>`0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45` |
| `UNISWAP_V4_FACET` | [UniswapV4Facet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/uniswap-v4/UniswapV4Facet.sol) | `0x000000000022D473030F116dDEE9F6B43aC78BA3`<br>`0xbD216513d74C8cf14cf4747E6AaA6420FF64ee9e`<br>`0x66a9893cC07D91D95644AEDD05D03f95e1dBA8Af` |
| `USDS_FACET` | [USDSFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/usds/USDSFacet.sol) | `0xdC035D45d973E3EC169d2276DDab16f1e407384F` |
| `WEETH_FACET` | [WEETHFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/weeth/WEETHFacet.sol) | `0xCd5fE23C85820F7B72D0926FC9b05b43E359b7ee`<br>`0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` |
| `WRAP_PROXY_ETH_FACET` | [WrapProxyETHFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/wrap-proxy-eth/WrapProxyETHFacet.sol) | `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` |
| `WSTETH_FACET` | [WSTETHFacet](https://github.com/sky-ecosystem/diamond-pau/blob/5c5ad6ae174bf467081ca82342ced2bd42a5c732/src/facets/wsteth/WSTETHFacet.sol) | `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2`<br>`0x889edC2eDab5f40e902b864aD4d7AdE8E412F9B1`<br>`0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0` |

## GitHub References

- Registry source: [sky-ecosystem/sky-pau-registry `src/Ethereum.sol` @ `161bac0c17a7d2c4d4e0455e1febe401a7a36edb`](https://github.com/sky-ecosystem/sky-pau-registry/blob/161bac0c17a7d2c4d4e0455e1febe401a7a36edb/src/Ethereum.sol)
- Diamond source release: [sky-ecosystem/diamond-pau v1.13.0](https://github.com/sky-ecosystem/diamond-pau/releases/tag/v1.13.0)
- Administered-agent source release: [sky-ecosystem/pau-administered-agent v1.0.0](https://github.com/sky-ecosystem/pau-administered-agent/releases/tag/v1.0.0)
- Sky chainlog source: [sky-ecosystem/chainlog-ui `api/mainnet/active.json` @ `d0cbbe08fbc591991fd92197b1e7698311e82c26`](https://github.com/sky-ecosystem/chainlog-ui/blob/d0cbbe08fbc591991fd92197b1e7698311e82c26/api/mainnet/active.json)

