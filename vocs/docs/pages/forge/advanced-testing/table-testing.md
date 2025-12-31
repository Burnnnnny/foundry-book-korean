---
description: 테스트 케이스 입력을 테이블 형식(입력, 예상 출력 및 잠재적으로 다른 관련 매개변수에 대한 열 포함)으로 구성하여 스마트 컨트랙트의 일반적인 동작과 엣지 케이스를 테스트하는 속성 기반 테스트입니다.
---

## 테이블 테스트 (Table Testing)

Foundry v1.3.0부터는 테이블 테스트 지원이 포함되어 있어 데이터셋("테이블")을 정의하고 해당 데이터셋의 각 항목에 대해 테스트 함수를 실행할 수 있습니다. 이 접근 방식은 입력과 조건의 특정 조합이 테스트되도록 보장하는 데 도움이 됩니다.

#### 테스트 정의

Forge에서 테이블 테스트는 `table` 접두사가 붙은 함수로 명명되며, 하나 이상의 데이터셋을 인수로 받습니다:

```solidity
function tableSumsTest(TestCase memory sums) public
```

```solidity
function tableSumsTest(TestCase memory sums, bool enable) public
```

데이터셋은 Forge 픽스처(fixture)로 정의되며 다음과 같을 수 있습니다:

- `fixture` 접두사와 데이터셋 이름이 뒤따르는 스토리지 배열
- `fixture` 접두사와 데이터셋 이름이 뒤따르는 함수. 함수는 값의 배열(고정 크기 또는 동적)을 반환해야 합니다.

#### 예제

- 단일 데이터셋. 다음 예제에서 `tableSumsTest` 테스트는 `fixtureSums` 데이터셋의 입력을 사용하여 두 번 실행됩니다: 한 번은 `TestCase(1, 2, 3)`으로, 한 번은 `TestCase(4, 5, 9)`로 실행됩니다.

```solidity
struct TestCase {
    uint256 a;
    uint256 b;
    uint256 expected;
}

function fixtureSums() public returns (TestCase[] memory) {
    TestCase[] memory entries = new TestCase[](2);
    entries[0] = TestCase(1, 2, 3);
    entries[1] = TestCase(4, 5, 9);
    return entries;
}

function tableSumsTest(TestCase memory sums) public pure {
    require(sums.a + sums.b == sums.expected, "wrong sum");
}
```

매개변수 이름이 사용 가능한 픽스처(`fixtureSums`)에 대해 해결되므로 `tableSumsTest`의 `TestCase` 매개변수 이름을 `sums`로 지정해야 합니다. 이 예제에서 매개변수 이름이 `sums`가 아니면 `[FAIL: Table test should have fixtures defined]` 오류가 발생합니다.

- 다중 데이터셋. `tableSwapTest` 테스트는 `fixtureWallet` 및 `fixtureSwap` 데이터셋의 같은 위치에 있는 값을 사용하여 두 번 실행됩니다.

```solidity
struct Wallet {
    address owner;
    uint256 amount;
}

struct Swap {
    bool swap;
    uint256 amount;
}

Wallet[] public fixtureWallet;
Swap[] public fixtureSwap;

function setUp() public {
    // 첫 번째 테이블 테스트 입력
    fixtureWallet.push(Wallet(address(11), 11));
    fixtureSwap.push(Swap(true, 11));

    // 두 번째 테이블 테스트 입력
    fixtureWallet.push(Wallet(address(12), 12));
    fixtureSwap.push(Swap(false, 12));
}

function tableSwapTest(Wallet memory wallet, Swap memory swap) public pure {
    require(
        (wallet.owner == address(11) && swap.swap) || (wallet.owner == address(12) && !swap.swap), "not allowed"
    );
}
```

위에서 언급한 것과 동일한 명명 요구 사항이 여기서도 관련됩니다.
