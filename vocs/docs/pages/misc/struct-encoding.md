---
description: Solidity 구조체가 ABI 튜플에 매핑되는 방식과 중첩 구조가 있는 컨트랙트 상호 작용을 위한 인코딩을 처리하는 방법을 알아보세요.
---

## 구조체 인코딩 (Struct Encoding)

구조체(struct)는 여러 변수를 그룹화할 수 있는 사용자 정의 타입입니다:

```solidity
struct MyStruct {
    address addr;
    uint256 amount;
}
```

새로운 [ABI coder v2](https://docs.soliditylang.org/en/latest/layout-of-source-files.html#abi-coder-pragma)만이 임의로 중첩된 배열과 구조체를 인코딩 및 디코딩할 수 있습니다. Solidity 0.8.0부터는 기본적으로 활성화되어 있으며, 그 이전에는 `pragma experimental ABIEncoderV2`를 통해 활성화해야 합니다.

Solidity 구조체는 ABI 타입 "튜플(tuple)"에 매핑됩니다. Solidity 타입이 ABI 타입에 매핑되는 방식에 대한 자세한 내용은 Solidity 문서의 [Solidity 타입을 ABI 타입에 매핑](https://docs.soliditylang.org/en/latest/abi-spec.html#mapping-solidity-to-abi-types)을 참조하세요.

따라서 구조체는 튜플로 인코딩 및 디코딩됩니다. 위에서 정의한 구조체 `MyStruct`는 ABI 관점에서 튜플 `(address,uint256)`에 매핑됩니다.

컨트랙트에서 이것이 어떻게 작동하는지 살펴봅시다:

```solidity
pragma solidity =0.8.15;


contract Test {
    struct MyStruct {
        address addr;
        uint256 amount;
    }
    function f(MyStruct memory t) public pure {}
}
```

이 컨트랙트의 `f` 함수의 ABI는 다음과 같습니다:

```json
{
  "inputs": [
    {
      "components": [
        {
          "internalType": "address",
          "name": "addr",
          "type": "address"
        },
        {
          "internalType": "uint256",
          "name": "amount",
          "type": "uint256"
        }
      ],
      "internalType": "struct Test.MyStruct",
      "name": "t",
      "type": "tuple"
    }
  ],
  "name": "f",
  "outputs": [],
  "stateMutability": "pure",
  "type": "function"
}
```

이는 다음과 같이 읽힙니다: 함수 `f`는 `address`와 `uint256` 타입의 두 구성 요소를 가진 `tuple` 타입의 입력 1개를 받습니다.

**중첩 구조체 인코딩:**
다음은 중첩된 구조체가 있는 더 복잡한 예시입니다:

```solidity
pragma solidity 0.8.21;

contract Test {
    struct nestedStruct {
        address addr;
        uint256 amount;
    }

    struct MyStruct {
        string nestedStructName;
        uint256 nestedStructCount;
        nestedStruct _nestedStruct;
    }

    function f(MyStruct memory t) public pure {}
}
```

이 컨트랙트의 `f` 함수의 ABI는 다음과 같습니다:

```json
{
  "inputs": [
    {
      "name": "t",
      "type": "tuple",
      "internalType": "struct Test.MyStruct",
      "components": [
        {
          "name": "nestedStructName",
          "type": "string",
          "internalType": "string"
        },
        {
          "name": "nestedStructCount",
          "type": "uint256",
          "internalType": "uint256"
        },
        {
          "name": "_nestedStruct",
          "type": "tuple",
          "internalType": "struct Test.nestedStruct",
          "components": [
            {
              "name": "addr",
              "type": "address",
              "internalType": "address"
            },
            {
              "name": "amount",
              "type": "uint256",
              "internalType": "uint256"
            }
          ]
        }
      ]
    }
  ],
  "name": "f",
  "outputs": [],
  "stateMutability": "pure",
  "type": "function"
}
```

이는 다음과 같이 읽힙니다: 함수 `f`는 세 가지 구성 요소를 가진 튜플 타입의 입력 1개를 받습니다. 세 가지 구성 요소는 문자열, uint256, 그리고 address 타입의 addr과 uint256 타입의 amount 구성 요소를 가진 중첩 구조체를 나타내는 또 다른 튜플입니다.

`MyStruct`를 인코딩하여 함수 `f`의 매개변수로 전달하려면:

```bash
cast abi-encode "f((string,uint256,(address,uint256)))" "(example,1,(0x...,1))"
```

`MyStruct`를 인수로 받는 컨트랙트를 배포하려면:

```bash
forge create src/Test.sol:Test --constructor-args "(example,1,(0x...,1))"
```
