---
description: TOML 구성 파일로 복잡한 멀티 체인 배포 스크립트를 조정하기 위해 forge-std의 `Config` 컨트랙트를 사용하는 방법을 알아봅니다.
---

## Config로 스크립트 조정하기

forge-std `Config` 컨트랙트는 특히 멀티 체인 환경에서 복잡한 배포 스크립트를 관리하는 강력한 방법을 제공합니다. 구성을 TOML 파일에 중앙 집중화함으로써 다양한 네트워크 및 배포 시나리오에 적응하는 유지 보수 가능하고 재사용 가능한 스크립트를 만들 수 있습니다.

## 스크립트에 Config를 사용하는 이유는 무엇인가요?

전통적인 스크립팅 접근 방식은 종종 다음을 수반합니다:
- 주소 및 매개변수 하드코딩
- 도우미 파일과 상호 작용하기 위해 ffi 치트코드 처리
- 배포 주소를 수동으로 추적하고 도우미 파일에 다시 기록
- 복잡한 환경 변수 관리

이러한 모든 관행은 개발자가 경험이 풍부하고 꼼꼼하지 않으면 스크립트 배포 시 오류가 발생하기 쉽습니다. `Config` 컨트랙트는 다음을 제공하여 이러한 문제를 해결합니다:
- 사람이 읽을 수 있는 TOML 파일의 중앙 집중식 구성
- 자동 환경 변수 해결
- 구성 값에 대한 타입 안전 액세스
- 양방향 업데이트 (읽기 및 쓰기)
- 내장된 멀티 체인 지원

## 구성 설정

### 1. 구성 파일 생성

프로젝트 루트에 `deployments.toml` 파일을 생성하세요:

```toml
# deployments.toml
#
# 중요: 체인 키는 다음 중 하나여야 합니다:
# - 숫자 체인 ID (예: [1], [11155111], [10])
# - 유효한 Alloy 체인 별칭 (예: [mainnet], [sepolia], [optimism])
#
# 유효한 별칭은 https://github.com/alloy-rs/chains를 참조하세요. 새/사용자 정의 체인의 경우 숫자 ID를 사용하거나 PR을 여는 것을 고려하세요.

[mainnet]
endpoint_url = "${MAINNET_RPC_URL}"

[mainnet.address]
# 의존성
weth = "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2"
usdc = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"
multisig = "${MAINNET_MULTISIG}"

[mainnet.uint]
min_liquidity = 1000000  # 최소 $1M
fee_percentage = 300     # 3%

[mainnet.bool]
is_testnet = false
use_timelock = true

[sepolia]
endpoint_url = "${SEPOLIA_RPC_URL}"

[sepolia.address]
weth = "0x7b79995e5f793A07Bc00c21412e50Ecae098E7f9"
usdc = "0x94a9D9AC8a22534E3FaCa9F4e7F2E2cf85d5E4C8"
multisig = "${SEPOLIA_MULTISIG}"

[sepolia.uint]
min_liquidity = 1000    # 테스트용 $1k
fee_percentage = 300

[sepolia.bool]
is_testnet = true
use_timelock = false
```

### 2. 배포 스크립트 생성

```solidity
// script/Deploy.s.sol
pragma solidity ^0.8.13;

import {Script} from "forge-std/Script.sol";
import {Config} from "forge-std/Config.sol";
import {console} from "forge-std/console.sol";

// 컨트랙트 임포트
import {TokenFactory} from "../src/TokenFactory.sol";
import {LiquidityPool} from "../src/LiquidityPool.sol";
import {Governance} from "../src/Governance.sol";

contract DeployScript is Script, Config {
    // 배포 아티팩트
    TokenFactory public factory;
    LiquidityPool public pool;
    Governance public governance;

    function run() public {
        // 구성을 로드하고 배포 주소 저장을 위해 쓰기 저장(write-back)을 활성화합니다
        _loadConfig("./deployments.toml", true);

        // 배포 중인 체인 가져오기
        uint256 chainId = block.chainid;
        console.log("Deploying to chain:", chainId);

        // 구성 값 로드
        address weth = config.get("weth").toAddress();
        address usdc = config.get("usdc").toAddress();
        address multisig = config.get("multisig").toAddress();
        uint256 minLiquidity = config.get("min_liquidity").toUint256();
        uint256 feePercentage = config.get("fee_percentage").toUint256();
        bool useTimelock = config.get("use_timelock").toBool();

        // 트랜잭션 브로드캐스트 시작
        vm.startBroadcast();

        // 컨트랙트 배포
        factory = new TokenFactory(multisig);
        console.log("TokenFactory deployed at:", address(factory));

        pool = new LiquidityPool(
            weth,
            usdc,
            minLiquidity,
            feePercentage
        );
        console.log("LiquidityPool deployed at:", address(pool));

        if (useTimelock) {
            governance = new Governance(multisig, 2 days);
            console.log("Governance deployed with timelock at:", address(governance));
        } else {
            governance = new Governance(multisig, 0);
            console.log("Governance deployed without timelock at:", address(governance));
        }

        // 컨트랙트 구성
        factory.setLiquidityPool(address(pool));
        pool.setFactory(address(factory));
        pool.setGovernance(address(governance));

        vm.stopBroadcast();

        // 배포 주소를 구성에 다시 저장
        config.set("token_factory", address(factory));
        config.set("liquidity_pool", address(pool));
        config.set("governance", address(governance));
        config.set("deployed_at", block.timestamp);
        config.set("deployer", msg.sender);

        console.log("\nDeployment complete! Addresses saved to deployments.toml");
    }
}
```

## 고급 패턴

### 멀티 체인 배포

체인별 구성을 사용하여 여러 체인에 동일한 컨트랙트를 배포합니다:

```solidity
contract MultiChainDeployScript is Script, Config {
    struct DeploymentResult {
        address factory;
        address pool;
        address governance;
    }

    mapping(uint256 => DeploymentResult) public deployments;

    function run() public {
        // 구성을 로드하고 모든 체인에 대한 포크 생성
        _loadConfigAndForks("./deployments.toml", true);

        // 구성된 각 체인에 배포
        for (uint256 i = 0; i < chainIds.length; i++) {
            uint256 chainId = chainIds[i];
            deployToChain(chainId);
        }

        // 크로스 체인 구성 검증
        verifyCrossChainSetup();
    }

    function deployToChain(uint256 chainId) internal {
        // 체인의 포크로 전환
        vm.selectFork(forkOf[chainId]);

        console.log("\n========================================");
        console.log("Deploying to chain:", chainId);
        console.log("========================================");

        // Config는 현재 포크의 체인 ID를 자동으로 사용합니다
        address weth = config.get("weth").toAddress();
        bool isTestnet = config.get("is_testnet").toBool();

        vm.startBroadcast();

        // 체인별 구성으로 배포
        TokenFactory factory = new TokenFactory(
            config.get("multisig").toAddress()
        );

        LiquidityPool pool = new LiquidityPool(
            weth,
            config.get("usdc").toAddress(),
            config.get("min_liquidity").toUint256(),
            config.get("fee_percentage").toUint256()
        );

        // 테스트넷 대 메인넷에 대한 다른 구성
        Governance governance;
        if (isTestnet) {
            governance = new Governance(
                config.get("multisig").toAddress(),
                0 // 테스트넷에서는 타임락 없음
            );
        } else {
            governance = new Governance(
                config.get("multisig").toAddress(),
                2 days // 메인넷에서는 타임락
            );
        }

        vm.stopBroadcast();

        // 배포 결과 저장
        deployments[chainId] = DeploymentResult({
            factory: address(factory),
            pool: address(pool),
            governance: address(governance)
        });

        // 구성 파일에 저장
        config.set("token_factory", address(factory));
        config.set("liquidity_pool", address(pool));
        config.set("governance", address(governance));
    }

    function verifyCrossChainSetup() internal view {
        console.log("\n========================================");
        console.log("Cross-chain deployment summary:");
        console.log("========================================");

        for (uint256 i = 0; i < chainIds.length; i++) {
            uint256 chainId = chainIds[i];
            DeploymentResult memory result = deployments[chainId];

            console.log("\nChain", chainId);
            console.log("  Factory:", result.factory);
            console.log("  Pool:", result.pool);
            console.log("  Governance:", result.governance);
        }
    }
}
```

### 이력 추적이 포함된 업그레이드 스크립트

배포 기록을 추적하고 업그레이드를 관리합니다:

```solidity
contract UpgradeScript is Script, Config {
    function run() public {
        _loadConfig("./deployments.toml", true);

        // 현재 배포 읽기
        address currentImpl = config.get("implementation_v1").toAddress();
        address proxy = config.get("proxy").toAddress();

        console.log("Current implementation:", currentImpl);
        console.log("Proxy address:", proxy);

        vm.startBroadcast();

        // 새 구현 배포
        MyContractV2 newImpl = new MyContractV2();

        // 프록시 업그레이드
        IProxy(proxy).upgradeTo(address(newImpl));

        vm.stopBroadcast();

        // 이전 구현 보관 및 새 구현 저장
        config.set("implementation_v1_deprecated", currentImpl);
        config.set("implementation_v2", address(newImpl));
        config.set("upgraded_at", block.timestamp);
        config.set("upgraded_by", msg.sender);

        console.log("Upgrade complete!");
        console.log("New implementation:", address(newImpl));
    }
}
```

## 모범 사례

### 1. 체인 구성

TOML 파일에서 체인을 구성할 때:

**최대 호환성을 위해 숫자 체인 ID를 사용하세요:**
```toml
[1]
endpoint_url = "${MAINNET_RPC}"

[1234]
endpoint_url = "${CUSTOM_CHAIN_RPC}"
```

**검증된 체인에 대해서만 Alloy 별칭을 사용하세요:**
```toml
[mainnet]
[optimism]
[arbitrum-sepolia]
[base-sepolia]
```

**별칭을 사용하기 전에 Alloy 체인을 확인하세요:**
- [Alloy 체인 저장소](https://github.com/alloy-rs/chains/blob/main/src/named.rs) 방문
- 체인의 별칭 검색. 찾을 수 없는 경우 숫자 체인 ID를 사용하거나 PR을 여는 것을 고려하세요.

### 2. 환경 변수

민감한 데이터는 환경 변수에 저장하세요:

```bash
# .env
MAINNET_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY
SEPOLIA_RPC_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY
MAINNET_MULTISIG=0x742d35Cc6634C0532925a3b844Bc9e7595f0fA9b
SEPOLIA_MULTISIG=0x1234567890123456789012345678901234567890
PRIVATE_KEY=0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```

### 3. 구성 유효성 검사

배포 전 구성 유효성 검사:

```solidity
function validateConfig() internal view {
    // 중요한 주소가 설정되었는지 확인
    require(
        config.get("multisig").toAddress() != address(0),
        "Multisig not configured"
    );

    // 매개변수가 예상 범위 내에 있는지 확인
    uint256 fee = config.get("fee_percentage").toUint256();
    require(fee <= 1000, "Fee too high"); // 최대 10%

    // 체인별 요구 사항 확인
    if (!config.get("is_testnet").toBool()) {
        require(config.get("use_timelock").toBool(), "Timelock required for mainnet");
    }
}
```

### 4. 별도의 구성 파일

목적에 따라 다른 구성 파일 사용:

```solidity
// 개발 배포
_loadConfig("./config/dev.toml", true);

// 프로덕션 배포
_loadConfig("./config/prod.toml", true);

// 테스트 구성
_loadConfig("./config/test.toml", false);
```

## 스크립트 실행

구성 기반 스크립트 실행:

```bash
# 드라이 런 (시뮬레이션)
forge script script/Deploy.s.sol:DeployScript \
    --rpc-url $SEPOLIA_RPC_URL

# 테스트넷에 배포
forge script script/Deploy.s.sol:DeployScript \
    --rpc-url $SEPOLIA_RPC_URL \
    --private-key $PRIVATE_KEY \
    --broadcast \
    --verify

# 여러 체인에 배포
forge script script/MultiChainDeploy.s.sol:MultiChainDeployScript \
    --broadcast \
    --verify
```

## 문제 해결

### 일반적인 문제

1. **환경 변수가 해결되지 않음**: 셸에서 변수가 내보내졌거나 `.env`에 정의되었는지 확인하세요.

2. **파일 권한**: `foundry.toml`에서 파일 시스템 액세스 권한 부여:
   ```toml
   fs_permissions = [{ access = "read-write", path = "./" }]
   ```

3. **`InvalidChainKey` 오류**: Alloy 체인 별칭과 일치하지 않는 사용자 정의 체인 이름을 사용할 때 발생합니다.

   **문제:**
   ```toml
   [my_custom_chain]  # 실패합니다!
   endpoint_url = "..."
   ```

   **해결책:**
   - 숫자 체인 ID 사용:
     ```toml
     [1234]  # 모든 체인에서 작동
     endpoint_url = "..."
     ```
   - 또는 정확한 Alloy 체인 별칭 사용 ([지원되는 체인](https://github.com/alloy-rs/chains/blob/main/src/named.rs) 참조):
     ```toml
     [arbitrum-sepolia]  # 지원되는 체인에서 작동
     endpoint_url = "..."
     ```

4. **타입 불일치 오류**: TOML의 값이 선언된 타입과 일치하는지 확인하세요 (예: 주소는 유효한 16진수 문자열이어야 함).

## 함께 보기

- [Config 참조](/reference/forge-std/config.mdx)
- [StdConfig 참조](/reference/forge-std/std-config.mdx)
- [Solidity로 스크립팅하기](/guides/scripting-with-solidity)
- [스크립트 작성 모범 사례](/guides/best-practices/writing-scripts)
