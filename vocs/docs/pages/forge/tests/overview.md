---
description: forge test를 사용하여 스마트 컨트랙트를 테스트합니다.
---

## 테스트

Forge는 [`forge test`](/forge/reference/test) 명령어로 테스트를 실행할 수 있습니다. 모든 테스트는 Solidity로 작성됩니다.

Forge는 소스 디렉토리 어디에서나 테스트를 찾습니다. `test`로 시작하는 함수가 있는 모든 컨트랙트는 테스트로 간주됩니다. 일반적으로 테스트는 관례에 따라 `test/`에 위치하며 `.t.sol`로 끝납니다.

다음은 기본 테스트만 있는 새로 생성된 프로젝트에서 `forge test`를 실행하는 예제입니다:

```sh
// [!include ~/snippets/output/hello_foundry/forge-test:all]
```

필터를 전달하여 특정 테스트만 실행할 수도 있습니다:

```sh
// [!include ~/snippets/output/test_filters/forge-test-match-contract-and-test:all]
```

이 명령어는 이름에 `testDeposit`이 포함된 `ComplicatedContractTest` 테스트 컨트랙트의 테스트를 실행합니다.
이러한 플래그의 반대 버전(`--no-match-contract` 및 `--no-match-test`)도 존재합니다.

`--match-path`를 사용하여 glob 패턴과 일치하는 파일 이름의 테스트를 실행할 수 있습니다.

```sh
// [!include ~/snippets/output/test_filters/forge-test-match-path:all]
```

`--match-path` 플래그의 반대는 `--no-match-path`입니다.

### 로그와 트레이스

`forge test`의 기본 동작은 통과 및 실패한 테스트의 요약만 표시하는 것입니다. 상세도(verbosity)를 높임으로써( `-v` 플래그 사용) 이러한 동작을 제어할 수 있습니다. 각 상세도 단계는 더 많은 정보를 추가합니다:

- **레벨 2 (`-vv`)**: 테스트 중에 출력된 로그도 표시됩니다. 여기에는 예상 값 대 실제 값과 같은 정보를 보여주는 테스트의 어설션 오류가 포함됩니다.
- **레벨 3 (`-vvv`)**: 실패한 테스트에 대한 스택 트레이스도 표시됩니다.
- **레벨 4 (`-vvvv`)**: 모든 테스트에 대한 스택 트레이스가 표시되고, 실패한 테스트에 대한 설정(setup) 트레이스가 표시됩니다.
- **레벨 5 (`-vvvvv`)**: 스택 트레이스와 설정 트레이스가 항상 표시됩니다.

### 감시 모드 (Watch mode)

Forge는 `forge test --watch`를 사용하여 파일 변경 시 테스트를 다시 실행할 수 있습니다.

기본적으로 변경된 테스트 파일만 다시 실행됩니다. 변경 시 모든 테스트를 다시 실행하려면 `forge test --watch --run-all`을 사용할 수 있습니다.
