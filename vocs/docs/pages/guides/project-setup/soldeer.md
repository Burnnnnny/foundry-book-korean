---
description: 중앙 저장소 및 git 지원을 통해 종속성을 관리하기 위해 Soldeer를 기본 Solidity 패키지 관리자로 사용합니다.
---

## 패키지 관리자로서의 Soldeer

[여기](/guides/project-setup/dependencies)에서 설명한 대로, Foundry는 지금까지 종속성을 처리하기 위해 git 서브모듈을 사용해 왔습니다.

프로젝트가 복잡해짐에 따라 기본 패키지 관리자의 필요성이 대두되었습니다.

새로운 접근 방식인 [soldeer.xyz](https://soldeer.xyz)가 만들어졌습니다. 이는 Rust로 구축되고 오픈 소스로 공개된 Solidity 네이티브 종속성 관리자입니다 ([https://github.com/mario-eth/soldeer](https://github.com/mario-eth/soldeer) 저장소 확인).

#### Soldeer의 전체 명령어와 사용법을 보려면 [USAGE.md](https://github.com/mario-eth/soldeer/blob/main/USAGE.md)를 방문하세요.

### 새 프로젝트 초기화

Foundry 프로젝트에서 Soldeer를 처음 사용하는 경우, `init` 명령어를 사용하여 필요한 구성과 최신 버전의 `forge-std`가 포함된 새로운 Soldeer 인스턴스를 설치할 수 있습니다.

```bash
forge soldeer init
```

### 종속성 추가

#### 중앙 저장소에 저장된 종속성 추가

종속성을 추가하려면 [soldeer.xyz](https://soldeer.xyz)를 방문하여 추가하려는 종속성(예: openzeppelin 5.0.2)을 검색할 수 있습니다.

![image](https://i.postimg.cc/Hm6R8MTs/Unknown-413.png)

그런 다음 forge 명령어를 실행하세요:

```bash
forge soldeer install @openzeppelin-contracts~5.0.2
```

이 명령어는 중앙 저장소에서 종속성을 다운로드하고 `dependencies` 디렉토리에 설치합니다.

Soldeer는 두 가지 유형의 종속성 구성을 관리할 수 있습니다: `soldeer.toml`을 사용하거나 `foundry.toml`에 내장하는 방식입니다. Foundry와 함께 작업하려면 `foundry.toml`에 `[dependencies]` 구성을 정의해야 합니다. 이렇게 하면 `soldeer CLI`가 설치된 종속성을 그곳에 정의하도록 지시합니다.
예:

```toml
# 전체 참조 https://github.com/foundry-rs/foundry/tree/master/crates/config

[profile.default]
auto_detect_solc = false
bytecode_hash = "none"
fuzz = { runs = 1_000 }
libs = ["dependencies"] # <= 이것을 추가하는 것이 중요합니다
gas_reports = ["*"]

[dependencies] # <= 종속성은 이 설정 아래에 추가됩니다
"@openzeppelin-contracts" = { version = "5.0.2" }
"@uniswap-universal-router" = { version = "1.6.0" }
"@prb-math" = { version = "4.0.2" }
forge-std = { version = "1.8.1" }
```

#### 특정 링크에 저장된 종속성 추가

중앙 저장소에 특정 종속성이 없는 경우 zip 아카이브 링크를 제공하여 설치할 수 있습니다.

예:

```bash
forge soldeer install @custom-dependency~1.0.0 --url https://my-website.com/custom-dependency-1-0-0.zip
```

위 명령어는 제공된 링크에서 종속성을 다운로드하고 일반 종속성처럼 설치하려고 시도합니다. 이를 위해 구성에 `url`이라는 추가 필드가 표시됩니다.

예:

```toml
[dependencies]
"@custom-dependency" = { version = "1.0.0", url = "https://my-website.com/custom-dependency-1-0-0.zip" }
```

#### GIT에 저장된 종속성 추가

종속성 소스로 Git을 사용하기로 선택한 경우(Git은 종속성 관리자로 설계되지 않았으므로 일반적으로 권장하지 않음), Git 저장소 링크를 추가 인수로 제공할 수 있습니다. 그러면 Soldeer가 Git 하위 프로세스를 사용하여 설치를 자동으로 처리합니다.
예:

```bash
forge soldeer install forge-std~1.9.2 --git https://github.com/foundry-rs/forge-std.git
```

특정 리비전, 브랜치 또는 태그를 사용하려면 명령어에 `--rev/--tag/--branch` 인수를 추가하여 사용할 수 있습니다.

예:

```bash
forge soldeer install forge-std~1.9.2 --git https://github.com/foundry-rs/forge-std.git --rev 4695fac44b2934aaa6d7150e2eaf0256fdc566a7
```

몇 가지 git 예시:

```bash
forge soldeer install test-project~v1 --git git@github.com:test/test.git
forge soldeer install test-project~v1 --git git@gitlab.com:test/test.git
```

```bash
forge soldeer install test-project~v1 --git https://github.com/test/test.git
forge soldeer install test-project~v1 --git https://gitlab.com/test/test.git
```

```bash
forge soldeer install test-project~v1 --git git@github.com:test/test.git --rev 345e611cd84bfb4e62c583fa1886c1928bc1a464
forge soldeer install test-project~v1 --git git@github.com:test/test.git --branch dev
forge soldeer install test-project~v1 --git git@github.com:test/test.git --tag v1
```

### 종속성 업데이트

Soldeer는 구성 파일(foundry 또는 soldeer TOML)에 종속성을 명시하므로 팀 내에서 종속성 구성을 공유하기가 훨씬 쉽습니다.

예를 들어 git 저장소에 Foundry 구성 파일이 있는 경우, 저장소를 풀(pull)하고 `forge soldeer update`를 실행할 수 있습니다. 이 명령어는 `[dependencies]` 태그 아래에 지정된 모든 종속성을 자동으로 설치합니다.

```toml
# 전체 참조 https://github.com/foundry-rs/foundry/tree/master/crates/config

[profile.default]
auto_detect_solc = false
bytecode_hash = "none"
fuzz = { runs = 1_000 }
libs = ["dependencies"] # <= 이것을 추가하는 것이 중요합니다
gas_reports = ["*"]

[dependencies] # <= 종속성은 이 설정 아래에 추가됩니다
"@openzeppelin-contracts" = { version = "5.0.2" }
"@uniswap-universal-router" = { version = "1.6.0" }
"@prb-math" = { version = "4.0.2" }
forge-std = { version = "1.8.1" }
```

### 종속성 제거

`forge soldeer uninstall DEPENDENCY`를 사용할 수 있습니다.

예: `forge soldeer uninstall @openzeppelin-contracts`. 이 작업은 다음을 제거합니다:

- 구성 항목
- `dependencies` 아티팩트
- `soldeer.lock` 항목
- `remappings` 항목 (txt 또는 config 리매핑)

또한 아티팩트(종속성 파일, 구성 항목, 리매핑 항목)를 제거하여 종속성을 수동으로 제거할 수도 있습니다.

### 아티팩트 정리 (Cleaning Artifacts)

`forge soldeer clean` 명령어는 `dependencies` 폴더와 그 안의 모든 내용을 제거합니다. 이는 구성 파일이나 잠금 파일에서 새로 설치하기 전이나 단순히 디스크 공간을 확보하는 데 유용할 수 있습니다.

### 리매핑 (Remappings)

이제 리매핑을 완전히 구성할 수 있습니다. 구성 TOML 파일(foundry.toml)은 다음 옵션을 가진 `[soldeer]` 테이블을 허용합니다.

```toml
[soldeer]
# soldeer가 리매핑을 관리할지 여부
remappings_generate = true

# 종속성을 설치, 업데이트 또는 제거할 때 soldeer가 모든 리매핑을 재생성할지 여부
remappings_regenerate = false

# 리매핑에 버전을 접미사로 붙일지 여부: `name-a.b.c`
remappings_version = true

# 리매핑에 추가할 접두사 ("@"는 `@name`이 됨)
remappings_prefix = ""

# 리매핑을 저장할 위치 (`remappings.txt`의 경우 "txt", `foundry.toml`의 경우 "config")
# `soldeer.toml`이 구성으로 사용되는 경우 무시됨 (`remappings.txt` 사용)
remappings_location = "txt"
```

### 종속성의 종속성 설치 (하위 종속성)

종속성을 설치할 때 해당 종속성이 설치해야 할 다른 종속성을 가지고 있을 수 있습니다. 현재는 구성 파일의 구성 항목으로 `recursive_deps` 필드를 지정하거나 설치 또는 업데이트 명령을 실행할 때 `--recursive-deps` 인수를 전달하여 이를 처리할 수 있습니다. 이렇게 하면 필요한 모든 하위 종속성이 자동으로 가져와집니다.
예:

```toml
[soldeer]
recursive_deps = true
```

### 중앙 저장소에 새 버전 푸시

Soldeer는 npmjs/crates.io처럼 작동하며, 모든 개발자가 자신의 프로젝트를 중앙 저장소에 게시하도록 장려합니다.

그렇게 하려면 [soldeer.xyz](https://soldeer.xyz)에 가서 계정을 생성하고 인증한 다음,

![image](https://i.postimg.cc/G3VDpN2S/s1.png)

새 프로젝트를 추가하세요.

![image](https://i.postimg.cc/rsBRYd3L/s2.png)

프로젝트가 생성되면 프로젝트 소스로 이동하여 다음을 수행할 수 있습니다:

- `.soldeerignore` 파일을 생성하여 필요하지 않은 파일을 제외하세요(`.gitignore`와 유사). `.gitignore` 파일도 존중됩니다.
- `forge soldeer login`을 실행하여 계정에 로그인하세요.
- 프로젝트 `my-project`의 버전 `1.0.0`과 연결된 중앙 저장소에 푸시하려는 디렉토리의 터미널에서 `forge soldeer push my-project~1.0.0`을 실행하세요.

터미널이 있는 현재 디렉토리가 아닌 특정 디렉토리를 푸시하려면 `forge soldeer push my-project~1.0.0 /path/to/directory`를 사용할 수 있습니다.

**경고** ⚠️

모든 사람이 볼 수 있는 민감한 파일을 중앙 저장소에 푸시할 위험이 있습니다. `.soldeerignore` 또는 `.gitignore` 파일에서 민감한 파일을 제외했는지 확인하세요.
또한 `.dot` 파일/디렉토리가 포함된 프로젝트를 푸시하려고 하면 경고가 발생하도록 구현했습니다.
이 경고를 건너뛰려면 다음을 사용하면 됩니다:

```bash
forge soldeer push my-project~1.0.0 --skip-warnings
```

#### 드라이 런 (Dry-run)

버전을 푸시하면 어떻게 될지 시뮬레이션하고 싶은 경우 `--dry-run` 플래그를 사용할 수 있습니다. 이렇게 하면 중앙 저장소에 푸시하기 전에 검사할 수 있는 zip 파일이 생성됩니다.

```bash
forge soldeer push my-project~1.0.0 --dry-run
```

#### 로그인 데이터

API 액세스 토큰을 검색하거나 제공하려면 `forge soldeer login` 명령어를 사용해야 합니다.
CLI 토큰은 [soldeer.xyz](https://soldeer.xyz)에서 생성할 수 있으며, 이메일 로그인은 향후 Soldeer 버전에서 제거될 예정이므로 CLI에서 이메일과 비밀번호를 사용하는 것보다 선호됩니다.
또는 `SOLDEER_API_TOKEN` 환경 변수를 통해 유효한 CLI 토큰을 제공할 수 있습니다. `forge soldeer login --help`를 사용하여 사용 가능한 옵션을 확인하세요.

> **경고** ⚠️
>
> - 프로젝트가 생성되면 삭제할 수 없습니다.
> - 버전이 푸시되면 삭제할 수 없습니다.
> - 동일한 버전을 두 번 푸시할 수 없습니다.
> - 터미널에서 실행하는 명령어의 프로젝트 이름은 Soldeer 웹사이트에서 생성한 프로젝트 이름과 일치해야 합니다.
> - 컨트랙트로 임포트할 때 버전 고정(version pinning)을 사용하는 것을 권장합니다. 이렇게 하면 사용 중인 종속성의 버전을 정확히 알 수 있어 코드를 보호하는 데 도움이 됩니다. 또한 보안 연구원들의 작업에도 도움이 됩니다.
> - 드라이 런을 실행하는 경우 버전을 푸시하기 전에 zip 파일을 삭제하세요.
>   예: `import '@openzeppelin-contracts/token/ERC20.sol'` 대신
>   `import '@openzeppelin-contracts-5.0.2/token/ERC20.sol'`을 사용해야 합니다.

### 특정 패키지가 중앙 저장소에 없으면 어떻게 되나요?

- 특정 패키지가 중앙 저장소에 없는 경우 [Soldeer 저장소](https://github.com/mario-eth/soldeer/issues)에 이슈를 열면 팀에서 추가를 검토할 것입니다.
- 사용하고 싶은 패키지가 있는데 중앙 저장소에 없는 경우, 위의 단계에 따라 직접 중앙 저장소에 푸시할 수 있습니다.

## 리매핑 주의사항

git 서브모듈이나 npm과 같은 다른 종속성 관리자를 사용하는 경우, soldeer와 다른 관리자 간에 종속성이 중복되지 않도록 하세요.

Soldeer 없이 설치된 종속성을 대상으로 하는 리매핑은 `--regenerate-remappings` 플래그가 지정되거나 `remappings_regenerate = true` 옵션이 설정되지 않는 한, Soldeer 명령어를 사용할 때 수정되거나 제거되지 않습니다.

## 종속성 유지 보수

Soldeer의 비전은 OpenZeppelin, Solady, Uniswap과 같은 주요 프로젝트가 자체 패키지를 Soldeer 레지스트리에 게시하여 커뮤니티가 쉽게 포함하고 적시에 업데이트를 받을 수 있도록 하는 것입니다.

이러한 일이 발생할 때까지 Soldeer 유지 관리 팀(현재 m4rio.eth)은 npmjs 또는 GitHub 버전에 의존하여 가장 인기 있는 종속성을 저장소에 푸시할 것입니다. 우리는 [오픈 소스 크롤러 도구](https://github.com/mario-eth/soldeer-crawler)를 사용하여 종속성을 크롤링하고 `soldeer` 조직 아래에 푸시하고 있습니다.

보안을 강화하고자 하는 분들을 위해 `soldeer.lock` 파일은 각 다운로드된 ZIP 파일과 해당 압축 해제된 폴더에 대해 `SHA-256` 해시를 저장합니다 (생성 방식은 `soldeer_core::utils::hash_folder` 참조).
이를 공식 릴리스와 비교하여 파일이 조작되지 않았는지 확인할 수 있습니다.

**프로젝트 유지 관리자님**
프로젝트를 Soldeer 조직에서 이동시켜 직접 Soldeer에 버전을 푸시하고 싶으시면 GitHub에 이슈를 열거나 [X (구 트위터)](https://twitter.com/m4rio_eth)에서 m4rio.eth에게 연락주세요.
