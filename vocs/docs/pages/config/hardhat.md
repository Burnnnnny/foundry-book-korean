## Hardhat과 통합하기

[Hardhat](https://hardhat.org/)과 함께 Foundry 프로젝트를 운영할 수 있습니다. 이 문서는 시스템에 Foundry와 node가 설치되어 있다고 가정합니다. 또한 Foundry와 Hardhat 모두에 익숙하다고 가정합니다.

### 왜 바로 작동하지 않나요?

Hardhat은 기본적으로 모든 NodeJS 의존성의 기본 폴더인 `node_modules`에 라이브러리가 설치되기를 기대합니다. Foundry는 `lib`에 있기를 기대합니다. 물론 [Foundry를 구성](/config/overview)할 수 있지만, `node_modules` 디렉토리 구조로 쉽게 맞출 수는 없습니다.

이러한 이유로 [hardhat-foundry](https://www.npmjs.com/package/@nomicfoundation/hardhat-foundry)를 사용하는 것을 권장합니다. hardhat-foundry를 올바르게 설치하고 사용하면 Hardhat은 Foundry가 사용하는 것과 동일한 contracts 디렉토리를 사용하며, forge install로 설치된 의존성을 사용할 수 있게 됩니다.

이 문서에서는 두 가지 시나리오를 다룹니다:

1. Foundry 프로젝트에 Hardhat 추가하기
2. Hardhat 프로젝트에 Foundry 추가하기

### 예제 저장소만 보여주세요!

[여기 있습니다!](https://github.com/foundry-rs/HardhatInFoundry)

이미 보유한 Foundry 프로젝트에 적용하거나 작동 방식을 배우고 싶다면 아래 내용을 읽어보세요:

### Foundry 프로젝트에 Hardhat 추가하기

Foundry 프로젝트 작업 디렉토리 내부에서:

1. `npm init -y` - `package.json` 파일을 설정합니다.
2. `npm i --save-dev hardhat` - 프로젝트에 Hardhat을 개발 의존성으로 설치합니다.
3. `npx hardhat init` - 동일한 디렉토리 내에서 Hardhat 프로젝트를 초기화하고 "**Create an empty hardhat.config.js**" 옵션을 선택합니다. 이렇게 하면 기본 `hardhat.config.js` 파일이 생성됩니다.
4. `npm i --save-dev @nomicfoundation/hardhat-foundry @nomicfoundation/hardhat-toolbox` - hardhat-foundry 플러그인과 Hardhat 테스트를 실행하는 데 필요한 모든 기본 의존성의 조합인 Hardhat toolbox 플러그인을 설치합니다.

플러그인이 작동하도록 `hardhat.config.js` 파일은 다음과 같아야 합니다:

```javascript
require("@nomicfoundation/hardhat-toolbox");
require("@nomicfoundation/hardhat-foundry");
/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.19",
};
```

5. 기본적으로 Foundry 프로젝트는 간단한 `Counter.sol` 컨트랙트와 몇 가지 테스트와 함께 제공됩니다. 기본 `Counter.t.sol` 파일과 같은 위치인 `test` 디렉토리 내에 `Counter.t.js` 파일을 생성하세요.
6. `Counter.t.js` 파일에 다음 코드를 추가하세요:

```javascript
const { expect } = require("chai");
const hre = require("hardhat");
const { loadFixture } = require("@nomicfoundation/hardhat-toolbox/network-helpers");

describe("Counter contract", function () {
  async function CounterLockFixture() {
    const counter = await ethers.deployContract("Counter");
    await counter.setNumber(0);

    return { counter };
  }

  it("Should increment the number correctly", async function () {
    const { counter } = await loadFixture(CounterLockFixture);
    await counter.increment();
    expect(await counter.number()).to.equal(1);
  });

  // This is not a fuzz test because Hardhat does not support fuzzing yet.
  it("Should set the number correctly", async function () {
    const { counter } = await loadFixture(CounterLockFixture);
    await counter.setNumber(100);
    expect(await counter.number()).to.equal(100);
  });
});
```

이 코드는 기본 `Counter.t.sol` 파일과 동일한 테스트를 실행합니다.

이제 끝입니다!
동일한 `test` 디렉토리 내에서 Hardhat 및 Foundry 테스트를 생성하고 각각 `npx hardhat test` 및 `forge test`로 실행할 수 있습니다.
더 자세한 내용은 [Hardhat 문서](https://hardhat.org/docs)를 확인하세요.

### Hardhat 프로젝트에 Foundry 추가하기

Hardhat 프로젝트 작업 디렉토리 내부에서:

1. `npm i --save-dev @nomicfoundation/hardhat-foundry`- hardhat-foundry 플러그인을 설치합니다.
2. `hardhat.config.js` 파일 상단에 `require("@nomicfoundation/hardhat-foundry");`를 추가하세요.

> ℹ️ **Note**
> 3단계는 디렉토리가 초기화된 git 저장소인 경우에만 작동합니다. 아직 하지 않았다면 `git init`을 실행하세요.

3. 터미널에서 `npx hardhat init-foundry`를 실행하세요. 이렇게 하면 Hardhat 프로젝트의 기존 구성을 기반으로 `foundry.toml` 파일이 생성되고 `forge-std` 라이브러리가 설치됩니다.

이제 Hardhat은 Foundry가 컨트랙트, 테스트 및 의존성을 찾을 위치를 알 수 있도록 `foundry.toml` 파일 내에 몇 가지 구성을 포함하여 동일한 디렉토리 내에 기본 Foundry 프로젝트를 설정합니다. 언제든지 `foundry.toml` 파일을 편집하여 이러한 구성을 변경할 수 있습니다.