---
description: CREATE2 오퍼코드를 사용하여 반사실적(counterfactual) 상호 작용을 위한 예측 가능한 주소로 여러 네트워크에 스마트 컨트랙트를 배포합니다.
---

## `CREATE2`를 사용한 결정론적 배포

2019년 [Constantinople 포크](https://ethereum.org/en/history/#constantinople)의 일부로 EVM에 포함된 `CREATE2`는 [EIP-1014](https://eips.ethereum.org/EIPS/eip-1014)로 여정을 시작한 오퍼코드입니다.
`CREATE2`를 사용하면 배포자가 제어하는 매개변수를 기반으로 결정론적 주소에 스마트 컨트랙트를 배포할 수 있습니다.

결과적으로 `CREATE2`는 알려진 코드가 해당 주소에 배치될 것임을 보장하므로, 아직 생성되지 않은 주소와 상호 작용할 수 있는 "반사실적(counterfactual)" 배포를 가능하게 한다고 자주 언급됩니다.

이는 배포된 컨트랙트의 주소가 배포자의 논스(nonce)에 대한 함수인 `CREATE` 오퍼코드와 대조됩니다.
`CREATE2`를 사용하면 주소의 논스가 다르더라도 동일한 배포자 계정을 사용하여 여러 네트워크의 동일한 주소에 컨트랙트를 배포할 수 있습니다.

최상의 사용자 경험을 위해서는 여러 EVM 체인에서 동일한 배포에 대해 서로 다른 주소를 갖지 않는 것이 좋습니다.

:::note
이 가이드는 `CREATE2`를 사용하여 결정론적 배포를 구성하는 데 도움을 주기 위한 것입니다.
기본적으로 `new Counter{salt: salt}()`는 [`0x4e59b44847b379578588920ca78fbf26c0b4956c`](https://github.com/Arachnid/deterministic-deployment-proxy)에 있는 결정론적 배포자를 사용합니다. 배포자는 모든 EVM 체인에서 사용 가능하지 않을 수 있습니다.
다른 배포자 주소는 `foundry.toml`에서 `create2_deployer`를 설정하거나 `--create2-deployer` 인수를 사용하여 구성할 수 있습니다.
:::

결정론적 배포를 설정하려면 다음 단계를 따르세요:

::::steps

### `foundry.toml` 구성하기

결정론적 주소에 신뢰할 수 있게 배포하려면 바이트코드가 동일하게 유지되어야 합니다. 이를 위해 `foundry.toml`을 다음과 같이 구성하세요:

```toml
[profile.default]
solc = "<SOLC_VERSION>"
evm_version = "<EVM_VERSION>"
bytecode_hash = "none"
cbor_metadata = false
```

### Solc 버전 고정

`solc` (Solidity) 버전을 고정해야 합니다. 일반적으로 최신 버전이나 선호하는 경우 가장 최신 버전을 사용하는 것이 좋습니다.

```toml
solc = "<SOLC_VERSION>"
```

### EVM 버전 설정

다음으로 `evm_version`을 구성하세요. 일반적으로 최신 하드포크를 사용하는 것이 좋지만, 배포 대상에 따라 오퍼코드 비호환성으로 인해 이전 하드포크를 사용해야 할 수도 있습니다.

```toml
evm_version = "<EVM_VERSION>"
```

### 메타데이터 및 바이트코드 설정 구성

기본적으로 Solidity 컴파일러는 바이트코드 끝에 메타데이터 파일의 해시를 추가합니다. 이 바이트코드에는 컴파일러 버전 및 ABI와 같은 내용이 포함됩니다.

소스 파일 해시가 메타데이터 파일에 포함되므로 소스 파일의 단일 바이트라도 변경되면 메타데이터 해시도 변경됩니다. 즉, 주어진 소스 파일로 컨트랙트를 컴파일할 수 있고 바이트코드 + 추가된 메타데이터 해시가 온체인 컨트랙트와 정확히 동일하다면, 이는 동일한 소스 파일과 동일한 컴파일 설정의 바이트 단위 일치임을 확신할 수 있습니다.

메타데이터 파일은 다음과 같을 수 있습니다:

```json
{
  "compiler": {
    "version": "0.8.28+commit.7893614a"
  },
  "language": "Solidity",
  "output": {
    "abi": [
      {
        "inputs": [],
        "stateMutability": "nonpayable",
        "type": "function",
        "name": "increment"
      },
      {
        "inputs": [],
        "stateMutability": "view",
        "type": "function",
        "name": "number",
        "outputs": [
          {
            "internalType": "uint256",
            "name": "",
            "type": "uint256"
          }
        ]
      },
      {
        "inputs": [
          {
            "internalType": "uint256",
            "name": "newNumber",
            "type": "uint256"
          }
        ],
        "stateMutability": "nonpayable",
        "type": "function",
        "name": "setNumber"
      }
    ],
    "devdoc": {
      "kind": "dev",
      "methods": {},
      "version": 1
    },
    "userdoc": {
      "kind": "user",
      "methods": {},
      "version": 1
    }
  },
  "settings": {
    "remappings": ["forge-std/=lib/forge-std/src/"],
    "optimizer": {
      "enabled": false,
      "runs": 200
    },
    "metadata": {
      "bytecodeHash": "ipfs"
    },
    "compilationTarget": {
      "src/Counter.sol": "Counter"
    },
    "evmVersion": "cancun",
    "libraries": {}
  },
  "sources": {
    "src/Counter.sol": {
      "keccak256": "0x09277f949d59a9521708c870dc39c2c434ad8f86a5472efda6a732ef728c0053",
      "urls": [
        "bzz-raw://94cd5258357da018bf911aeda60ed9f5b130dce27445669ee200313cd3389200",
        "dweb:/ipfs/QmNbEfWAqXCtfQpk6u7TpGa8sTHXFLpUz7uebz2FVbchSC"
      ],
      "license": "UNLICENSED"
    }
  },
  "version": 1
}
```

메타데이터 파일에 대해 자세히 알아보려면 [여기](https://playground.sourcify.dev/)를 클릭하세요.

다음과 같이 메타데이터를 비활성화하면:

```toml
bytecode_hash = "none"
cbor_metadata = false
```

바이트코드의 일부로 메타데이터 해시를 포함하지 않습니다. 즉, 바이트코드는 이제 결정론적일 수 있지만 컨트랙트를 검증할 때 [`완전 일치(full match)`](https://docs.sourcify.dev/docs/full-vs-partial-match/#full-perfect-matches)가 아닌 [`부분 일치(partial match)`](https://docs.sourcify.dev/docs/full-vs-partial-match/#partial-matches)만 가능합니다. 요구 사항에 따라 이는 허용될 수 있습니다.

### 최적화 도구 구성

`optimizer`를 활성화하는 경우 `optimizer_runs`가 일관되게 유지되도록 하세요.

### CREATE2 팩토리 설정

기본적으로 컨트랙트는 기본(또는 `create2_deployer` 구성으로 지정된) create2 팩토리를 사용하지 않으며, 실행되는 컨트랙트에서 create2 오퍼코드를 실행하는 것으로 기본 설정됩니다. 예를 들어 개인 키 없이 테스트를 실행하거나 스크립트를 실행할 때 이러한 동작이 발생합니다.

항상 create2 팩토리를 사용하려면 다음 구성을 사용할 수 있습니다:

```toml
always_use_create_2_factory = true
```

예를 들어 테스트에서 create2 팩토리 배포 주소를 사용하려는 경우 유용합니다.

::::

## 컨트랙트 배포

Solidity의 기본 `CREATE`를 사용할 때 컨트랙트의 새 주소는 `sender` 주소의 `hash`와 `sender`의 `nonce`를 사용하여 결정됩니다:

```
new_contract_address = keccak256(rlp_encode([sender, nonce]))[12:]
```

```solidity
// 기본 CREATE 오퍼코드 사용
Counter counter = new Counter();
```

`nonce`는 각 체인에서 한 번만 사용할 수 있기 때문에 동일한 컨트랙트 주소에 배포하는 신뢰할 수 없는 방법입니다.

대신 `CREATE2`의 `salt` 매개변수를 사용해 보겠습니다.

`CREATE2`의 `salt` 매개변수는 최종 배포된 컨트랙트 주소를 결정하는 핵심 구성 요소입니다. 이를 통해 결정론적 배포에서 유연성과 고유성을 확보할 수 있습니다. 배포된 컨트랙트의 주소는 다음 공식을 사용하여 파생됩니다:

```
new_contract_address = keccak256(0xff ++ deployer ++ salt ++ keccak256(init_code))
```

```solidity
// CREATE2 오퍼코드에 `salt` 매개변수 전달
Counter counter = new Counter{salt: salt}();
```

- `0xff`는 고유성을 보장하는 고정 접두사입니다.
- `deployer`는 CREATE2 작업을 실행하는 주소입니다.
- `salt`는 배포자가 선택한 32바이트 값입니다.
- `keccak256(bytecode)`는 컨트랙트 생성 바이트코드의 해시입니다.

`0xff`는 고정되어 있고, `deployer`는 결정론적 배포자([0x4e59b44847b379578588920ca78fbf26c0b4956c](https://github.com/Arachnid/deterministic-deployment-proxy))이며 바이트코드가 고정되어 있으므로, `salt` 매개변수를 사용하여 새 컨트랙트 주소를 완전히 제어할 수 있습니다.

## 추가 리소스

- [컨트랙트 메타데이터](https://docs.soliditylang.org/en/latest/metadata.html)
- [초기화 코드와 무관한 결정론적 배포](https://github.com/Vectorized/solady/blob/main/src/utils/CREATE3.sol)
