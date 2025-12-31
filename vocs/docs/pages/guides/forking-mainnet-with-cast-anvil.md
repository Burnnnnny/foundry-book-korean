---
description: Anvil을 사용하여 라이브 이더리움 네트워크를 포크하고 테스트 및 실험을 위해 Cast를 사용하여 실제 컨트랙트와 상호 작용합니다.
---

## `Cast`와 `Anvil`로 메인넷 포크하기

[Anvil][anvil]과 [Cast][cast]를 결합하여 실제 네트워크의 컨트랙트와 상호 작용하여 포크하고 테스트할 수 있습니다. 이 가이드의 목표는 DAI를 보유한 사람으로부터 Anvil이 생성한 계정으로 DAI 토큰을 전송하는 방법을 보여주는 것입니다.

다음 단계에 따라 메인넷을 포크하고 DAI 토큰을 전송하세요:

::::steps

### Anvil로 메인넷 포크

메인넷을 포크하는 것부터 시작해 봅시다.

```sh
anvil --fork-url https://mainnet.infura.io/v3/$INFURA_KEY
```

10개의 계정이 공개 키 및 개인 키와 함께 생성되는 것을 볼 수 있습니다. 우리는 `0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266` (이 사용자를 앨리스라고 부릅시다)으로 작업할 것입니다.

### 환경 변수 설정

Etherscan으로 이동하여 DAI 토큰 보유자를 검색합니다 ([여기](https://etherscan.io/token/0x6b175474e89094c44da98b954eedeac495271d0f#balances)). 임의의 계정을 하나 고릅시다. 이 예제에서는 `0xfc2eE3bD619B7cfb2dE2C797b96DeeCbD7F68e46`을 사용합니다. 컨트랙트와 계정을 환경 변수로 내보냅시다:

```sh
export ALICE=0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266
export DAI=0x6b175474e89094c44da98b954eedeac495271d0f
export UNLUCKY_USER=0xfc2eE3bD619B7cfb2dE2C797b96DeeCbD7F68e46
```

### 초기 잔액 확인

[`cast call`][cast-call]을 사용하여 앨리스의 잔액을 확인할 수 있습니다:

```sh
cast call $DAI \
  "balanceOf(address)(uint256)" \
  $ALICE
0
```

마찬가지로 `cast call`을 사용하여 불운한 사용자의 잔액도 확인할 수 있습니다:

```sh
cast call $DAI \
  "balanceOf(address)(uint256)" \
  $UNLUCKY_USER
21840114973524208109322438
```

### DAI 토큰 전송

[`cast send`][cast-send]를 사용하여 불운한 사용자로부터 앨리스에게 토큰을 전송해 봅시다:

```sh
# 이것은 Anvil을 호출하여 불운한 사용자를 가장할 수 있게 해줍니다
cast rpc anvil_impersonateAccount $UNLUCKY_USER
cast send $DAI \
--from $UNLUCKY_USER \
  "transfer(address,uint256)(bool)" \
  $ALICE \
  300000000000000000000000 \
  --unlocked
blockHash               0xbf31c45f6935a0714bb4f709b5e3850ab0cc2f8bffe895fefb653d154e0aa062
blockNumber             15052891
...
```

### 전송 확인

전송이 작동했는지 확인해 봅시다:

```sh
cast call $DAI \
  "balanceOf(address)(uint256)" \
  $ALICE
300000000000000000000000

cast call $DAI \
  "balanceOf(address)(uint256)" \
  $UNLUCKY_USER
21540114973524208109322438
```

::::

[anvil]: /anvil/overview
[cast]: /cast/overview
[cast-call]: /cast/reference/call
[cast-send]: /cast/reference/send
