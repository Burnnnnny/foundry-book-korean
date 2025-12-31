---
description: 상세한 EVM 상태 검사 및 탐색 제어 기능을 갖춘 스마트 컨트랙트 실행 단계를 진행하기 위한 대화형 디버거입니다.
---

## 디버거

Forge는 대화형 디버거와 함께 제공됩니다.

디버거는 [`forge test`](/forge/reference/test), [`forge script`](/forge/reference/script), [`cast run`](/cast/reference/run)에서 접근할 수 있습니다. 한 번에 하나의 함수 또는 하나의 트랜잭션만 선택하여 디버그할 수 있습니다.

`forge test` (또는 `forge script`) 사용:

```sh
forge test --debug --match-test "<REGEX>"
```

여기서 `<REGEX>`는 디버그하려는 파일의 함수 서명입니다. 예를 들어:

```sh
forge test --debug --match-test "test_Increment"
```

일치하는 테스트가 퍼즈 테스트인 경우, 디버거는 실패한 첫 번째 퍼즈 시나리오 또는 (성공한 경우) 마지막 시나리오 중 더 먼저 발생하는 시나리오를 엽니다. 예를 들어:

```sh
forge test --debug --match-test "testFuzz_SetNumber"
```

`cast run` 사용:

```sh
cast run --debug \
  0xd15e0237413d7b824b784e1bbc3926e52f4726e5e5af30418803b8b327b4f8ca
```

### 디버거 레이아웃

![디버거 UI 이미지](/debugger.png)

디버거가 실행되면 터미널이 4개의 사분면으로 나뉩니다:

- **1사분면**: 디버깅 세션의 오퍼코드(opcode)와 현재 하이라이트된 오퍼코드. 또한 현재 계정의 주소, 프로그램 카운터(PC), 누적 가스 사용량이 표시됩니다.
- **2사분면**: 현재 스택(stack)과 스택의 크기
- **3사분면**: 소스 보기(Source view)
- **4사분면**: EVM의 현재 메모리(memory)

코드를 단계별로 진행함에 따라 스택과 메모리의 단어 색상이 변경되는 것을 볼 수 있습니다.

메모리의 경우:

- **빨간색 단어**: 현재 오퍼코드에 의해 쓰여질 예정(write intention)
- **녹색 단어**: 이전 오퍼코드에 의해 쓰여짐
- **청록색 단어**: 현재 오퍼코드에 의해 읽혀짐(read)

스택의 경우, **청록색 단어**는 현재 오퍼코드에 의해 읽히거나 꺼내지는(popped) 항목입니다.

> ⚠️ **참고**
>
> 대부분의 테스트 프레임워크에서는 실패한 첫 번째 어설션(assertion)이 보고됩니다.
> Foundry에서는 실패한 마지막 어설션(DSTest 또는 치트코드에서 발생)이 보고됩니다.

### 탐색

### 일반

- <kbd>q</kbd>: 디버거 종료
- <kbd>h</kbd>: 도움말 보기

### 호출(Call) 탐색

- <kbd>0-9</kbd> + <kbd>k</kbd>: 지정된 횟수만큼 뒤로 단계 이동 (또는 마우스로 위로 스크롤)
- <kbd>0-9</kbd> + <kbd>j</kbd>: 지정된 횟수만큼 앞으로 단계 이동 (또는 마우스로 아래로 스크롤)
- <kbd>g</kbd>: 트랜잭션의 시작으로 이동
- <kbd>G</kbd>: 트랜잭션의 끝으로 이동
- <kbd>c</kbd>: 이전 호출 유형 명령어로 이동(예: [`CALL`][op-call], [`STATICCALL`][op-staticcall], [`DELEGATECALL`][op-delegatecall], [`CALLCODE`][op-callcode])
- <kbd>C</kbd>: 다음 호출 유형 명령어로 이동
- <kbd>a</kbd>: 이전 [`JUMP`][op-jump] 또는 [`JUMPI`][op-jumpi] 명령어로 이동
- <kbd>s</kbd>: 다음 [`JUMPDEST`][op-jumpdest] 명령어로 이동
- <kbd>'</kbd> + <kbd>a-z</kbd>: [`vm.breakpoint`][cheat-breakpoint] 치트코드로 설정된 `<char>` 중단점으로 이동

### 메모리 탐색

- <kbd>Ctrl</kbd> + <kbd>j</kbd>: 메모리 뷰 아래로 스크롤
- <kbd>Ctrl</kbd> + <kbd>k</kbd>: 메모리 뷰 위로 스크롤
- <kbd>m</kbd>: 메모리를 UTF8로 보기

### 스택 탐색

- <kbd>J</kbd>: 스택 뷰 아래로 스크롤
- <kbd>K</kbd>: 스택 뷰 위로 스크롤
- <kbd>t</kbd>: 스택에 라벨을 표시하여 현재 op가 어떤 항목을 소비할지 확인

[op-call]: https://www.evm.codes/#f1
[op-staticcall]: https://www.evm.codes/#fa
[op-delegatecall]: https://www.evm.codes/#f4
[op-callcode]: https://www.evm.codes/#f2
[op-jumpdest]: https://www.evm.codes/#5b
[op-jump]: https://www.evm.codes/#f1
[op-jumpi]: https://www.evm.codes/#57
[cheat-breakpoint]: /reference/cheatcodes/breakpoint
