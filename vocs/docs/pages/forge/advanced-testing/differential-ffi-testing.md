---
description: 차등 테스트와 FFI를 사용하여 여러 구현을 비교하고 참조 구현에 대한 정확성을 검증합니다.
---

## 차등 테스트 (Differential Testing)

Forge는 차등 테스트와 차등 퍼징에 사용할 수 있습니다. `ffi` [치트코드](/reference/cheatcodes/ffi)를 사용하여 비(非) EVM 실행 파일에 대해 테스트할 수도 있습니다.

### 배경

[차등 테스트](https://en.wikipedia.org/wiki/Differential_testing)는 동일한 함수의 여러 구현의 출력을 비교하여 상호 참조합니다. 함수 명세 `F(X)`가 있고, 그 명세의 두 가지 구현 `f1(X)`와 `f2(X)`가 있다고 가정해 봅시다. 적절한 입력 공간에 존재하는 모든 x에 대해 `f1(x) == f2(x)`가 성립할 것으로 예상합니다. 만약 `f1(x) != f2(x)`라면, 적어도 하나의 함수가 `F(X)`를 잘못 구현하고 있음을 알 수 있습니다. 동일성을 테스트하고 불일치를 식별하는 이 과정이 차등 테스트의 핵심입니다.

차등 퍼징은 차등 테스트의 확장입니다. 차등 퍼징은 프로그램적으로 많은 `x` 값을 생성하여 수동으로 선택한 입력으로는 드러나지 않을 수 있는 불일치와 엣지 케이스를 찾습니다.

> 참고: 이 경우 `==` 연산자는 동일성에 대한 사용자 정의일 수 있습니다. 예를 들어 부동 소수점 구현을 테스트하는 경우 특정 허용 오차 내에서 근사적인 동일성을 사용할 수 있습니다.

이러한 유형의 테스트가 실제 생활에서 사용되는 몇 가지 예는 다음과 같습니다:

- 업그레이드된 구현을 이전 버전과 비교
- 코드를 알려진 참조 구현과 비교 테스트
- 타사 도구 및 종속성과의 호환성 확인

아래는 Forge가 차등 테스트에 사용되는 방법에 대한 몇 가지 예시입니다.

### 입문: `ffi` 치트코드

[`ffi`](/reference/cheatcodes/ffi)를 사용하면 임의의 쉘 명령어를 실행하고 출력을 캡처할 수 있습니다. 다음은 모의 예제입니다:

```solidity
import {Test} from "forge-std/Test.sol";

contract TestContract is Test {

    function testMyFFI () public {
        string[] memory cmds = new string[](2);
        cmds[0] = "cat";
        cmds[1] = "address.txt"; // ABI 인코딩된 주소가 포함되어 있다고 가정
        bytes memory result = vm.ffi(cmds);
        address loadedAddress = abi.decode(result, (address));
        // 주소로 무언가 수행
        // ...
    }
}
```

이 주소는 이전에 `address.txt`에 기록되었으며, FFI 치트코드를 사용하여 읽어들입니다. 이 데이터는 이제 테스트 컨트랙트 전체에서 사용할 수 있습니다.

### 예제: Merkle Tree 구현 차등 테스트

[Merkle Trees](https://en.wikipedia.org/wiki/Merkle_tree)는 블록체인 애플리케이션에서 자주 사용되는 암호화 약속(commitment) 체계입니다. 인기로 인해 Merkle Tree 생성기, 증명자(prover), 검증자(verifier)의 다양한 구현이 생겨났습니다. Merkle 루트와 증명은 종종 JavaScript나 Python과 같은 언어를 사용하여 생성되는 반면, 증명 검증은 주로 Solidity에서 온체인으로 수행됩니다.

[Murky](https://github.com/dmfxyz/murky)는 Solidity로 구현된 Merkle 루트, 증명 및 검증의 완전한 구현체입니다. 테스트 스위트에는 OpenZeppelin의 Merkle 증명 라이브러리에 대한 차등 테스트와 참조 JavaScript 구현에 대한 루트 생성 테스트가 포함되어 있습니다. 이러한 테스트는 Foundry의 퍼징 및 `ffi` 기능으로 구동됩니다.

#### 참조 TypeScript 구현에 대한 차등 퍼징

Murky는 `ffi` 치트코드를 사용하여 Forge의 퍼저가 제공한 데이터를 이용해 자체 Merkle 루트 구현을 TypeScript 구현과 비교 테스트합니다:

```solidity
function testMerkleRootMatchesJSImplementationFuzzed(bytes32[] memory leaves) public {
    vm.assume(leaves.length > 1);
    bytes memory packed = abi.encodePacked(leaves);
    string[] memory runJsInputs = new string[](8);

    // ffi 명령어 문자열 빌드
    runJsInputs[0] = 'npm';
    runJsInputs[1] = '--prefix';
    runJsInputs[2] = 'differential_testing/scripts/';
    runJsInputs[3] = '--silent';
    runJsInputs[4] = 'run';
    runJsInputs[5] = 'generate-root-cli';
    runJsInputs[6] = leaves.length.toString();
    runJsInputs[7] = packed.toHexString();

    // 명령어 실행 및 출력 캡처
    bytes memory jsResult = vm.ffi(runJsInputs);
    bytes32 jsGeneratedRoot = abi.decode(jsResult, (bytes32));

    // Murky를 사용하여 루트 계산
    bytes32 murkyGeneratedRoot = m.getRoot(leaves);
    assertEq(murkyGeneratedRoot, jsGeneratedRoot);
}
```

> 참고: `(bytes memory).toHexString()`을 가능하게 하는 라이브러리는 Murky 레포지토리의 [`Strings2.sol`](https://github.com/dmfxyz/murky/blob/main/differential_testing/test/utils/Strings2.sol)을 참조하세요.

Forge는 `npm --prefix differential_testing/scripts/ --silent run generate-root-cli {numLeaves} {hexEncodedLeaves}`를 실행합니다. 이것은 참조 JavaScript 구현을 사용하여 입력 데이터에 대한 Merkle 루트를 계산합니다. 스크립트는 루트를 stdout으로 출력하고, 해당 출력은 `vm.ffi()`의 반환 값에서 `bytes`로 캡처됩니다.

그런 다음 테스트는 Solidity 구현을 사용하여 루트를 계산합니다.

마지막으로 테스트는 두 루트가 정확히 같은지 어설션합니다. 같지 않으면 테스트는 실패합니다.

#### OpenZeppelin의 Merkle Proof 라이브러리에 대한 차등 퍼징

다른 Solidity 구현에 대해 차등 테스트를 사용하고 싶을 수도 있습니다. 이 경우 `ffi`는 필요하지 않습니다. 대신 참조 구현을 테스트로 직접 가져옵니다(import).

```solidity
import {MerkleProof} from "openzeppelin-contracts/contracts/utils/cryptography/MerkleProof.sol";
//...
function testCompatibilityOpenZeppelinProver(bytes32[] memory _data, uint256 node) public {
    vm.assume(_data.length > 1);
    vm.assume(node < _data.length);
    bytes32 root = m.getRoot(_data);
    bytes32[] memory proof = m.getProof(_data, node);
    bytes32 valueToProve = _data[node];
    bool murkyVerified = m.verifyProof(root, proof, valueToProve);
    bool ozVerified = MerkleProof.verify(proof, root, valueToProve);
    assertTrue(murkyVerified == ozVerified);
}
```

#### 알려진 엣지 케이스에 대한 차등 테스트

차등 테스트가 항상 퍼징되는 것은 아닙니다. 알려진 엣지 케이스를 테스트하는 데에도 유용합니다. Murky 코드베이스의 경우, `log2ceil` 함수의 초기 구현은 길이가 2의 거듭제곱(예: 129)에 가까운 특정 배열에서 작동하지 않았습니다. 안전성 검사로서, 이 길이의 배열에 대해 항상 테스트를 실행하고 TypeScript 구현과 비교합니다. 전체 테스트는 [여기](https://github.com/dmfxyz/murky/blob/main/differential_testing/test/DifferentialTests.t.sol#L21)에서 볼 수 있습니다.

#### 참조 데이터에 대한 표준화된 테스트

FFI는 재현 가능하고 표준화된 데이터를 테스트 환경에 주입하는 데에도 유용합니다. Murky 라이브러리에서는 다음과 같이 사용됩니다:

```solidity
bytes32[100] data;
uint256[8] leaves = [4, 8, 15, 16, 23, 42, 69, 88];

function setUp() public {
    string[] memory inputs = new string[](2);
    inputs[0] = "cat";
    inputs[1] = "src/test/standard_data/StandardInput.txt";
    bytes memory result =  vm.ffi(inputs);
    data = abi.decode(result, (bytes32[100]));
    m = new Merkle();
}

function testMerkleGenerateProofStandard() public view {
    bytes32[] memory _data = _getData();
    for (uint i = 0; i < leaves.length; ++i) {
        m.getProof(_data, leaves[i]);
    }
}
```

`src/test/standard_data/StandardInput.txt`는 인코딩된 `bytes32[100]` 배열이 포함된 텍스트 파일입니다. 테스트 외부에서 생성되며 모든 언어의 Web3 SDK에서 사용할 수 있습니다. 다음과 같은 모습입니다:

```bash
0xf910ccaa307836354233316666386231414464306335333243453944383735313..423532
```

표준화된 테스트 컨트랙트는 `ffi`를 사용하여 파일을 읽어들입니다. 데이터를 배열로 디코딩한 다음, 이 예제에서는 8개의 다른 잎(leaves)에 대한 증명을 생성합니다. 데이터가 상수이고 표준이기 때문에, 이 테스트를 사용하여 가스 및 성능 향상을 의미 있게 측정할 수 있습니다.

> 물론 배열을 테스트에 하드 코딩할 수도 있습니다! 하지만 그렇게 하면 컨트랙트, 구현 등에 걸쳐 일관된 테스트를 수행하기가 훨씬 더 어려워집니다.

### 예제: 점진적 더치 경매(Gradual Dutch Auctions) 차등 테스트

Paradigm의 [점진적 더치 경매(Gradual Dutch Auction)](https://www.paradigm.xyz/2022/04/gda) 메커니즘에 대한 참조 구현에는 다수의 차등 퍼징 테스트가 포함되어 있습니다. `ffi`를 사용한 차등 테스트를 더 깊이 탐구하기에 훌륭한 레포지토리입니다.

- [이산 GDA(Discrete GDAs)](https://github.com/FrankieIsLost/gradual-dutch-auction/blob/master/src/test/DiscreteGDA.t.sol#L78)에 대한 차등 테스트
- [연속 GDA(Continuous GDAs)](https://github.com/FrankieIsLost/gradual-dutch-auction/blob/master/src/test/ContinuousGDA.t.sol#L89)에 대한 차등 테스트
- [Python 구현](https://github.com/FrankieIsLost/gradual-dutch-auction/blob/master/analysis/compute_price.py) 참조

### 참조 레포지토리

- [점진적 더치 경매 (Gradual Dutch Auctions)](https://github.com/FrankieIsLost/gradual-dutch-auction)
- [Murky](https://www.github.com/dmfxyz/murky)
- [Solidity 퍼징 템플릿 (Solidity Fuzzing Template)](https://github.com/patrickd-/solidity-fuzzing-boilerplate)

참조로 사용할 만한 다른 레포지토리가 있다면 기여해 주세요!
