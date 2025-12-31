---
description: Foundry 프로젝트에서 git 서브모듈, 리매핑, 패키지 설치를 사용하여 스마트 컨트랙트 종속성을 관리합니다.
---

## 종속성 (Dependencies)

Forge는 기본적으로 [git 서브모듈](https://git-scm.com/book/en/v2/Git-Tools-Submodules)을 사용하여 종속성을 관리합니다. 즉, 스마트 컨트랙트가 포함된 모든 GitHub 저장소와 함께 작동합니다.

### 종속성 추가하기

종속성을 추가하려면 [`forge install`](/forge/reference/install)을 실행하세요:

```sh
// [!include ~/snippets/output/hello_foundry/forge-install:all]
```

이 명령어는 `solady` 라이브러리를 가져오고, git에서 `.gitmodules` 파일을 스테이징한 다음 `Installed solady`라는 메시지로 커밋을 생성합니다.

이제 `lib` 폴더를 확인해 보면:

```sh
// [!include ~/snippets/output/hello_foundry/tree:all]
```

Forge가 `solady`를 설치한 것을 볼 수 있습니다!

기본적으로 `forge install`은 최신 master 브랜치 버전을 설치합니다. 특정 태그나 커밋을 설치하려면 다음과 같이 할 수 있습니다:

```sh
forge install vectorized/solady@v0.1.3
```

### 종속성 리매핑하기

Forge는 종속성을 더 쉽게 임포트할 수 있도록 리매핑할 수 있습니다. Forge는 자동으로 몇 가지 리매핑을 추론하려고 시도합니다:

```sh
// [!include ~/snippets/output/hello_foundry/forge-remappings:all]
```

이러한 리매핑의 의미는 다음과 같습니다:

- `forge-std`에서 임포트하려면: `import "forge-std/Contract.sol";`
- `solady`에서 임포트하려면: `import "solady/Contract.sol";`

프로젝트 루트에 `remappings.txt` 파일을 생성하여 이러한 리매핑을 사용자 정의할 수 있습니다.

`solady` 저장소의 `utils` 폴더를 가리키는 `solady-utils`라는 리매핑을 만들어 보겠습니다!

```sh
@solady-utils/=lib/solady/src/utils/
```

`foundry.toml`에서도 리매핑을 설정할 수 있습니다.

```toml
remappings = [
    "@solady-utils/=lib/solady/src/utils/",
]
```

이제 다음과 같이 `solady` 저장소의 `src/utils`에 있는 모든 컨트랙트를 임포트할 수 있습니다:

```solidity
import {LibString} from "@solady-utils/LibString.sol";
```

### 리매핑 충돌

경우에 따라 두 개 이상의 git 서브모듈이 동일한 네임스페이스를 가진 서로 다른 종속성을 포함할 때 종속성 충돌이 발생할 수 있습니다. 예를 들어, `org/lib_1`과 `org/lib_2`를 모두 설치했고 각각 고유한 버전의 `@openzeppelin`을 참조한다고 가정해 보겠습니다. 이러한 시나리오에서 `forge remappings`는 해당 네임스페이스에 대해 단일 리매핑 항목을 생성하며, 이는 두 `@openzeppelin` 라이브러리 중 하나만 가리키게 됩니다.

```sh
forge remappings
@openzeppelin/=lib/lib_1/node_modules/@openzeppelin/
```

이러한 상황은 임포트 문제를 야기하여 `forge build` 실패를 유발하거나 컨트랙트에 예기치 않은 동작을 초래할 수 있습니다. 이를 해결하려면 `remappings.txt` 파일에 리매핑 컨텍스트(context)를 추가하면 됩니다. 이는 컴파일러에게 별개의 컴파일 컨텍스트에서 서로 다른 리매핑을 사용하도록 지시하여 충돌을 해결합니다. 예를 들어 `lib_1`과 `lib_2` 간의 충돌을 해결하려면 `remappings.txt`를 다음과 같이 업데이트합니다:

```sh
lib/lib_1/:@openzeppelin/=lib/lib_1/node_modules/@openzeppelin/
lib/lib_2/:@openzeppelin/=lib/lib_2/node_modules/@openzeppelin/
```

이 접근 방식은 각 종속성이 적절한 라이브러리 버전에 매핑되도록 하여 잠재적인 문제를 방지합니다. 리매핑에 대한 자세한 내용은 [Solidity 언어 문서](https://docs.soliditylang.org/en/latest/path-resolution.html#import-remapping)를 참조하세요.

### 종속성 업데이트하기

[`forge update <dep>`](/forge/reference/update)를 사용하여 지정한 버전의 최신 커밋으로 특정 종속성을 업데이트할 수 있습니다. 예를 들어 이전에 설치한 `solady`의 master 버전에서 최신 커밋을 가져오려면 다음을 실행합니다:

```sh
forge update lib/solady
```

또는 `forge update`만 실행하여 모든 종속성을 한 번에 업데이트할 수도 있습니다.

### 종속성 제거하기

[`forge remove <deps>...`](/forge/reference/remove)를 사용하여 종속성을 제거할 수 있습니다. 여기서 `<deps>`는 종속성의 전체 경로 또는 이름입니다. 예를 들어 `solady`를 제거하려면 다음 두 명령어는 동일합니다:

```sh
forge remove solady
# ... 아래와 동일합니다 ...
forge remove lib/solady
```

### Hardhat 호환성

Forge는 종속성이 npm 패키지(`node_modules`에 저장됨)이고 컨트랙트가 `src`가 아닌 `contracts`에 저장되는 Hardhat 스타일 프로젝트도 지원합니다.

Hardhat 호환 모드를 활성화하려면 `--hh` 플래그를 전달하세요.
