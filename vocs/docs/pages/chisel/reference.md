## chisel

### 이름

`chisel` - REPL 환경 내에서 Solidity 입력을 테스트하고 상세한 피드백을 받습니다.

### 시놉시스

`chisel` [*options*]

#### 하위 명령 (bin)

1. `chisel list`
   - `~/.foundry/cache/chisel`에 저장된 모든 캐시된 세션을 표시합니다.
1. `chisel load <id>`
   - `id = <id>`인 캐시된 세션이 존재하면 REPL을 실행하고 해당 세션을 로드합니다.
1. `chisel view <id>`
   - `id = <id>`인 캐시된 세션이 존재하면 세션의 REPL 컨트랙트 소스 코드를 표시합니다.
1. `chisel clear-cache`
   - `~/.foundry/cache/chisel` 디렉토리 내의 모든 캐시 파일을 삭제합니다. 이 세션들은 복구할 수 없으므로 이 명령은 주의해서 사용하세요.

#### 플래그

모든 사용 가능한 환경 구성 플래그에 대해서는 `man chisel` 또는 `chisel --help`를 참조하세요.

### 설명

Chisel은 개발자가 Solidity 코드 조각을 작성하고 테스트할 수 있게 해주는 Solidity REPL("read-eval-print loop"의 줄임말)입니다. 이는 Solidity 코드를 작성하고 실행할 수 있는 대화형 환경을 제공하며, 코드 작업 및 디버깅을 위한 내장 명령어 세트도 제공합니다. 따라서 샌드박스 Foundry 테스트 스위트를 구동하지 않고도 Solidity 코드를 빠르게 테스트하고 실험할 수 있는 유용한 도구입니다.

### 사용법

chisel을 열려면 단순히 `chisel` 바이너리를 실행하세요.

거기서 유효한 Solidity 코드를 입력하세요. 명령어 외에도 chisel 프롬프트에는 두 가지 종류의 입력이 있습니다:

1. 표현식 (Expressions)
   - 표현식은 값을 반환하거나 자체적으로 평가될 수 있는 문장입니다. 예를 들어,
     `1 << 8`은 값 `256`을 가진 `uint256`으로 평가되는 표현식입니다. 표현식은
     즉시 평가되며, 평가 후 세션 상태에 유지되지 않습니다.
   - 예시:
     - `address(0).balance`
     - `abi.encode(256, bytes32(0), "Chisel!")`
     - `myViewFunc(128)`
     - ...
1. 문장 (Statements)

   - 문장은 세션의 상태에 유지되도록 의도된 코드 조각입니다. 문장에는
     변수 정의, 값을 반환하는 비상태 변경 함수 호출, 그리고 컨트랙트, 함수,
     이벤트, 에러, 매핑 또는 구조체 정의가 포함됩니다. 표현식을 문장으로 평가하려면
     끝에 세미콜론(`;`)을 추가하면 됩니다.
   - 예시:

     - `uint256 a = 0xa57b`
     - `myStateMutatingFunc(128)` || `myViewFunc(128);`. `;`에 주목하세요.
     - ```solidity
       function hash64(
         bytes32 _a,
         bytes32 _b
       ) internal pure returns (bytes32 _hash) {
           assembly {
               // 해시하려는 64바이트를 스크래치 공간에 저장
               mstore(0x00, _a)
               mstore(0x20, _b)

               // 스크래치 공간의 메모리를 해시하고
               // 결과를 `_hash`에 할당
               _hash := keccak256(0x00, 0x40)
           }
       }
       ```

     - `event ItHappened(bytes32 indexed hash)`
     - `struct Complex256 { uint256 re; uint256 im; }`
     - ...

#### 사용 가능한 명령어

```text
// [!include ~/snippets/output/chisel/help:output]
```

**일반 (General)**

`!help` | `!h`

모든 명령어를 표시합니다.

`!quit` | `!q`

Chisel을 종료합니다.

`!exec <command> [args]` | `!e <command> [args]`

셸 명령어를 실행하고 출력을 인쇄합니다.

예시:

```sh
➜ !e ls
CHANGELOG.md
LICENSE
README.md
TESTS.md
artifacts
cache
contracts
crytic-export
deploy
deploy-config
deployments
dist
echidna.yaml
forge-artifacts
foundry.toml
hardhat.config.ts
layout-lock.json
node_modules
package.json
scripts
slither.config.json
slither.db.json
src
tasks
test-case-generator
tsconfig.build.json
tsconfig.build.tsbuildinfo
tsconfig.json
```

**세션 (Session)**

`!clear` | `!c`

현재 세션 소스를 지웁니다.

내부적으로 각 Chisel 세션에는 문장을 입력함에 따라 변경되는 기본 컨트랙트가 있습니다. 이 명령은 이 컨트랙트를 지우고 세션을 기본 상태로 재설정합니다.

`!source` | `!so`

현재 세션의 소스 코드를 표시합니다.

위에서 언급했듯이, 각 Chisel 세션에는 기본 컨트랙트가 있습니다. 이 명령은 이 컨트랙트의 소스 코드를 표시합니다.

`!save [id]` | `!s [id]`

현재 세션을 캐시에 저장합니다.

Chisel은 세션 캐싱을 허용하는데, 이는 Chisel에서 더 복잡한 로직을 테스트하거나 나중에 세션으로 돌아가고 싶을 때 매우 유용할 수 있습니다. 모든 캐시된 Chisel 세션은 `~/.foundry/cache/chisel`에 저장됩니다.

`id` 인수가 제공되지 않으면 Chisel은 저장하는 세션에 자동으로 숫자 ID를 할당합니다.

`!load <id>` | `!l <id>`

캐시에서 이전 세션 ID를 로드합니다.

이 명령은 이전에 캐시된 세션을 캐시에서 로드합니다. 세션의 소스와 함께 모든 환경 설정도 로드됩니다. `id` 인수는 `~/.foundry/cache/chisel` 디렉토리에 있는 기존 캐시된 세션과 일치해야 합니다.

`!list` | `!ls`

모든 캐시된 세션을 나열합니다.

이 명령은 `~/.foundry/cache/chisel` 디렉토리 내의 모든 캐시된 chisel 세션을 표시합니다.

`!clearcache` | `!cc`

저장된 모든 세션의 chisel 캐시를 지웁니다.

`~/.foundry/cache/chisel` 디렉토리 내의 모든 캐시 파일을 삭제합니다. 이 세션들은 복구할 수 없으므로 이 명령은 주의해서 사용하세요.

`!export` | `!ex`

현재 세션 소스를 스크립트 파일로 내보냅니다.

Foundry 프로젝트의 루트 디렉토리에서 `chisel`이 실행된 경우, 현재 세션을 프로젝트의 `scripts` 디렉토리에 있는 foundry 스크립트로 내보낼 수 있습니다.

`!fetch <addr> <name>` | `!fe <addr> <name>`

Etherscan에서 검증된 컨트랙트의 인터페이스를 가져옵니다.

이 명령은 Etherscan API에서 `<addr>`에 있는 검증된 컨트랙트의 인터페이스를 파싱하려고 시도합니다. 성공하면 인터페이스가 `<name>`이라는 이름으로 세션 소스에 삽입됩니다.

현재는 이더리움 메인넷에서 검증된 컨트랙트의 인터페이스만 가져올 수 있습니다. 추후 Chisel은 여러 Etherscan 지원 체인에서 인터페이스를 가져오는 것을 지원할 예정입니다.

`!edit`

현재 세션의 `run()` 함수를 편집기에서 엽니다.

chisel은 `$EDITOR` 환경 변수에 정의된 편집기를 사용합니다.

**환경 (Environment)**

`!fork <url>` | `!f <url>`

현재 세션에 대해 RPC를 포크합니다. 로컬 네트워크로 돌아가려면 인수를 0개 제공하세요.

제공된 RPC의 상태를 포크하려고 시도합니다. URL이 제공되지 않으면 빈 로컬 devnet 상태를 사용하는 것으로 돌아갑니다.

`!traces` | `!t`

현재 세션에 대한 트레이스를 활성화/비활성화합니다.

트레이싱이 활성화되면 각 문장이 삽입된 후 foundry 스타일의 호출 트레이싱 및 로그가 인쇄됩니다.

**디버그 (Debug)**

`!memdump` | `!md`

현재 상태의 원시 메모리를 덤프합니다.

REPL 컨트랙트의 `run` 함수의 마지막 명령어 실행이 완료된 후 기계 상태의 원시 메모리를 덤프하려고 시도합니다.

`!stackdump` | `!sd`

현재 상태의 원시 스택을 덤프합니다.

REPL 컨트랙트의 `run` 함수의 마지막 명령어 실행이 완료된 후 기계 상태의 원시 스택을 덤프하려고 시도합니다.

`!rawstack <var>` | `!rs <var>`

변수의 스택 할당의 원시 값을 표시합니다. 길이가 32바이트를 초과하는 변수의 경우 메모리 포인터를 표시합니다.

이 명령은 길이가 32바이트 미만인 변수에 대한 전체 원시 스택 할당을 보고 싶을 때 유용합니다.

예시:

```sh
➜ address addr
➜ assembly {
    addr := not(0)
}
➜ addr
Type: address
└ Data: 0xffffffffffffffffffffffffffffffffffffffff
➜ !rs addr
Type: bytes32
└ Data: 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
➜
```
