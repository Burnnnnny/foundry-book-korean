---
description: Foundry는 이더리움 애플리케이션을 빌드, 테스트, 배포 및 상호 작용하기 위한 스마트 컨트랙트 개발 도구 모음입니다.
---

## Foundry 개요

![Foundry banner](/og-image.png)

Foundry는 스마트 컨트랙트 개발 도구 모음입니다.

Foundry는 의존성을 관리하고, 프로젝트를 컴파일하고, 테스트를 실행하고, 배포하며, 커맨드 라인과 Solidity 스크립트를 통해 체인과 상호 작용할 수 있게 해줍니다.

## 문서 살펴보기

### 시작하기

[툴킷 설치](/introduction/installation)를 통해 Foundry를 실행하고 각 도구의 기본을 [시작](/introduction/getting-started)해 보세요.

### 가이드

Foundry로 견고한 스마트 컨트랙트와 개발 워크플로우를 구축하기 위한 포괄적인 튜토리얼과 모범 사례입니다.

- **모범 사례**
  - [컨트랙트 작성](/guides/best-practices/writing-contracts) - 깔끔하고 안전한 스마트 컨트랙트 개발 가이드라인
  - [테스트 작성](/guides/best-practices/writing-tests) - 효과적인 테스트 전략 및 패턴
  - [스크립트 작성](/guides/best-practices/writing-scripts) - 배포 및 자동화 스크립트 모범 사례
  - [보안](/guides/best-practices/security) - 보안 고려 사항 및 취약점 예방
  - [키 관리](/guides/best-practices/key-management) - 개인 키 및 비밀의 안전한 관리
  - [주석 작성](/guides/best-practices/commenting) - 문서화 및 코드 주석 표준
- [Solidity로 스크립팅하기](/guides/scripting-with-solidity) - 고급 배포 및 자동화 기술
- [CREATE2를 사용한 결정론적 배포](/guides/deterministic-deployments-using-create2) - 예측 가능한 컨트랙트 주소
- [Cast와 Anvil로 메인넷 포킹](/guides/forking-mainnet-with-cast-anvil) - 실제 체인 상태에 대해 테스트
- [Docker 내부에서 Foundry 실행](/guides/foundry-in-docker) - 컨테이너화된 개발 환경
- [EIP-712 서명 구현 및 테스트](/guides/eip712)

### 프로젝트 설정

스마트 컨트랙트 코드베이스 확장을 위한 [forge 프로젝트 설정 가이드](/guides/project-setup/creating-a-new-project)로 프로젝트를 구성하는 방법을 배워보세요.

### Forge

컨트랙트 빌드, 테스트, 배포 및 검증을 다루는 [Forge 개요](/forge/overview)를 통해 핵심 스마트 컨트랙트 개발 도구를 마스터하세요.

### Cast

[Cast](/cast/overview)를 사용하여 커맨드 라인에서 블록체인 네트워크와 상호 작용하고, 컨트랙트 호출, 트랜잭션 및 체인 데이터 조회를 수행하는 방법을 배워보세요.

### Anvil

포킹 기능을 갖춘 Foundry의 고성능 이더리움 호환 노드인 [Anvil](/anvil/overview)로 로컬 개발 네트워크를 설정하세요.

### Chisel

빠른 프로토타이핑 및 디버깅을 위한 통합 REPL인 [Chisel](/chisel/overview)로 Solidity를 대화형으로 탐색해 보세요.

### 구성

Foundry 설정을 사용자 정의하고 최적화된 개발 워크플로우를 위해 다른 도구와 통합하세요.

- [`foundry.toml`을 사용한 구성 개요](/config/overview) - 프로젝트 구성 및 설정
- [지속적 통합 (CI)](/config/continuous-integration) - CI/CD 파이프라인 통합
- [VSCode와 통합](/config/vscode) - 에디터 설정 및 확장
- [셸 자동 완성](/config/shell-autocompletion) - 커맨드 라인 생산성 향상
- [정적 분석기](/config/static-analyzers) - 코드 분석 도구 통합
- [Hardhat과 통합](/config/hardhat) - 프레임워크 간 호환성
- [Vyper 지원](/config/vyper) - 대체 스마트 컨트랙트 언어 지원

### 기여하기

Foundry 개선을 위해 기여해 주세요 - 자세한 내용은 [기여 가이드라인](https://github.com/foundry-rs/foundry/blob/master/CONTRIBUTING.md)을 참조하세요.

### 참조

모든 Foundry 도구에 대한 전체 커맨드 참조, 구성 옵션 및 API 문서입니다.

- [FAQ](/misc/faq) - 자주 묻는 질문 및 문제 해결
- **커맨드 참조**
  - [forge 커맨드](/forge/reference/forge) - 전체 forge CLI 참조
  - [cast 커맨드](/cast/reference/cast) - 전체 cast CLI 참조
  - [anvil 커맨드](/anvil/reference) - 전체 anvil CLI 참조
  - [chisel 커맨드](/chisel/reference) - 전체 chisel CLI 참조
- **구성 & API**
  - [구성 참조](/config/reference/overview) - 모든 구성 옵션
  - [Cheatcodes 참조](/reference/cheatcodes/overview) - 테스트 유틸리티 및 도우미
  - [Forge 표준 라이브러리 참조](/reference/forge-std/overview) - 표준 라이브러리 문서
  - [DSTest 참조](/reference/ds-test) - 레거시 테스트 프레임워크 참조

:::tip
Foundry의 멋진 리소스, 가이드, 도구 및 라이브러리 목록인 [Awesome Foundry](https://github.com/crisgarner/awesome-foundry)도 확인해 보세요!
:::
