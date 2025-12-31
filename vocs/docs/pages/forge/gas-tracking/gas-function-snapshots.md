---
description: 최적화 진행 상황을 추적하기 위해 함수의 가스 소비 스냅샷을 캡처하고 비교합니다.
---

## 가스 함수 스냅샷 (Gas Function Snapshots)

Forge는 모든 테스트 함수에 대한 가스 스냅샷을 생성할 수 있습니다. 이는 컨트랙트가 얼마나 많은 가스를 소비할지에 대한 일반적인 느낌을 얻거나, 다양한 최적화 전후의 가스 사용량을 비교하는 데 유용할 수 있습니다.

가스 스냅샷을 생성하려면 [`forge snapshot`](/forge/reference/snapshot)을 실행하세요.

이렇게 하면 기본적으로 `.gas-snapshot`이라는 파일이 생성되며, 여기에는 모든 테스트와 각각의 가스 사용량이 포함됩니다.

```
forge snapshot
cat .gas-snapshot

ERC20Test:testApprove() (gas: 31162)
ERC20Test:testBurn() (gas: 59875)
ERC20Test:testRevertTransferFromInsufficientAllowance() (gas: 81034)
ERC20Test:testRevertTransferFromInsufficientBalance() (gas: 81662)
ERC20Test:testRevertTransferInsufficientBalance() (gas: 52882)
ERC20Test:testInfiniteApproveTransferFrom() (gas: 90167)
ERC20Test:testMetadata() (gas: 14606)
ERC20Test:testMint() (gas: 53830)
ERC20Test:testTransfer() (gas: 60473)
ERC20Test:testTransferFrom() (gas: 84152)
```

### 필터링 (Filtering)

다른 출력 파일을 지정하려면 `forge snapshot --snap <FILE_NAME>`을 실행하세요.

가스 사용량별로 결과를 정렬할 수도 있습니다. `--asc` 옵션을 사용하여 오름차순으로 정렬하거나 `--desc`를 사용하여 내림차순으로 정렬하세요.

마지막으로 모든 테스트에 대한 최소/최대 가스 임계값을 지정할 수도 있습니다.
임계값 이상의 결과만 포함하려면 `--min <VALUE>` 옵션을 사용할 수 있습니다.
같은 방식으로 임계값 미만의 결과만 포함하려면 `--max <VALUE>` 옵션을 사용할 수 있습니다.

변경 사항은 화면에 표시되는 스냅샷이 아니라 스냅샷 파일에 적용된다는 점을 명심하세요.

`forge snapshot --match-path contracts/test/ERC721.t.sol`과 같이 `forge test`의 필터와 함께 사용하여 이 테스트 컨트랙트와 관련된 가스 스냅샷을 생성할 수도 있습니다.

### 가스 사용량 비교

현재 스냅샷 파일과 최신 변경 사항을 비교하려면 `--diff` 또는 `--check` 옵션을 사용할 수 있습니다.

`--diff`는 스냅샷과 비교하여 변경 사항을 표시합니다.

선택적으로 파일 이름(`--diff <FILE_NAME>`)을 받을 수도 있으며, 기본값은 `.gas-snapshot`입니다.

예를 들어:

```
forge snapshot --diff .gas-snapshot2

Running 10 tests for src/test/ERC20.t.sol:ERC20Test
[PASS] testApprove() (gas: 31162)
[PASS] testBurn() (gas: 59875)
[PASS] testRevertTransferFromInsufficientAllowance() (gas: 81034)
[PASS] testRevertTransferFromInsufficientBalance() (gas: 81662)
[PASS] testRevertTransferInsufficientBalance() (gas: 52882)
[PASS] testInfiniteApproveTransferFrom() (gas: 90167)
[PASS] testMetadata() (gas: 14606)
[PASS] testMint() (gas: 53830)
[PASS] testTransfer() (gas: 60473)
[PASS] testTransferFrom() (gas: 84152)
Test result: ok. 10 passed; 0 failed; finished in 2.86ms
testBurn() (gas: 0 (0.000%))
testRevertTransferFromInsufficientAllowance() (gas: 0 (0.000%))
testRevertTransferFromInsufficientBalance() (gas: 0 (0.000%))
testRevertTransferInsufficientBalance() (gas: 0 (0.000%))
testInfiniteApproveTransferFrom() (gas: 0 (0.000%))
testMetadata() (gas: 0 (0.000%))
testMint() (gas: 0 (0.000%))
testTransfer() (gas: 0 (0.000%))
testTransferFrom() (gas: 0 (0.000%))
testApprove() (gas: -8 (-0.000%))
Overall gas change: -8 (-0.000%)
```

`--check`는 스냅샷을 기존 스냅샷 파일과 비교하여 차이점이 있다면 모두 표시합니다. 다른 파일 이름을 제공하여 비교할 파일을 변경할 수 있습니다: `--check <FILE_NAME>`.

예를 들어:

```
forge snapshot --check .gas-snapshot2

Running 10 tests for src/test/ERC20.t.sol:ERC20Test
[PASS] testApprove() (gas: 31162)
[PASS] testBurn() (gas: 59875)
[PASS] testRevertTransferFromInsufficientAllowance() (gas: 81034)
[PASS] testRevertTransferFromInsufficientBalance() (gas: 81662)
[PASS] testRevertTransferInsufficientBalance() (gas: 52882)
[PASS] testInfiniteApproveTransferFrom() (gas: 90167)
[PASS] testMetadata() (gas: 14606)
[PASS] testMint() (gas: 53830)
[PASS] testTransfer() (gas: 60473)
[PASS] testTransferFrom() (gas: 84152)
Test result: ok. 10 passed; 0 failed; finished in 2.47ms
Diff in "ERC20Test::testApprove()": consumed "(gas: 31162)" gas, expected "(gas: 31170)" gas
```
