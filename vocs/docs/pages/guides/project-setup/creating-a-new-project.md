---
description: forge init과 기본 템플릿을 사용하여 새로운 Foundry 프로젝트를 초기화하고 프로젝트 구조를 설정합니다.
---

## 새 프로젝트 생성하기

::::steps

#### 프로젝트 초기화

Foundry로 새 프로젝트를 시작하려면 [`forge init`](/forge/reference/init)을 사용하세요:

```sh
// [!include ~/snippets/output/hello_foundry/forge-init:command]
```

이는 기본 템플릿으로 `hello_foundry`라는 새 디렉토리를 생성합니다. 또한 새로운 `git` 저장소를 초기화합니다.

:::info
다른 템플릿을 사용하여 새 프로젝트를 생성하려면 다음과 같이 `--template` 플래그를 전달하세요:
```sh
forge init --template https://github.com/foundry-rs/forge-template hello_template
```
:::

#### 프로젝트 디렉토리로 이동

새로 생성된 프로젝트로 이동하세요:

```sh
cd hello_foundry
```

#### 프로젝트 구조 살펴보기

기본 템플릿이 어떻게 생겼는지 확인해 봅시다:

```sh
// [!include ~/snippets/output/hello_foundry/tree:all]
```

기본 템플릿에는 하나의 종속성이 설치되어 있습니다: Forge 표준 라이브러리(Forge Standard Library). 이는 Foundry 프로젝트에 선호되는 테스트 라이브러리입니다. 또한, 템플릿에는 빈 시작 컨트랙트와 간단한 테스트도 함께 제공됩니다.

#### 프로젝트 빌드

컨트랙트 컴파일:

```sh
// [!include ~/snippets/output/hello_foundry/forge-build:all]
```

#### 테스트 실행

테스트 스위트 실행:

```sh
// [!include ~/snippets/output/hello_foundry/forge-test:all]
```

`out`과 `cache`라는 두 개의 새 디렉토리가 생긴 것을 확인할 수 있습니다.

`out` 디렉토리에는 ABI와 같은 컨트랙트 아티팩트가 포함되며, `cache`는 `forge`가 필요한 것만 다시 컴파일하는 데 사용됩니다.

::::
