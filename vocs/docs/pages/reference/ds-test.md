## DSTest 참조

Dappsys Test(줄여서 DSTest)는 기본적인 로깅 및 어설션 기능을 제공합니다. 이는 Forge 표준 라이브러리에 포함되어 있습니다.

함수에 액세스하려면 `forge-std/Test.sol`을 임포트하고 테스트 컨트랙트에서 `Test`를 상속받으세요:

```solidity
import {Test} from "forge-std/Test.sol";

contract ContractTest is Test {
    // ... 테스트 ...
}
```

### 로깅 (Logging)

사용 가능한 모든 로깅 이벤트에 대한 전체 개요입니다. 자세한 설명과 사용 예시는 아래를 참조하세요.

```solidity
event log                    (string);
event logs                   (bytes);

event log_address            (address);
event log_bytes32            (bytes32);
event log_int                (int);
event log_uint               (uint);
event log_bytes              (bytes);
event log_string             (string);

event log_named_address      (string key, address val);
event log_named_bytes32      (string key, bytes32 val);
event log_named_decimal_int  (string key, int val, uint decimals);
event log_named_decimal_uint (string key, uint val, uint decimals);
event log_named_int          (string key, int val);
event log_named_uint         (string key, uint val);
event log_named_bytes        (string key, bytes val);
event log_named_string       (string key, string val);
```

### 로깅 이벤트 (Logging events)

이 섹션에서는 로깅을 위한 모든 이벤트와 사용 예시를 문서화합니다.

#### `log`

```solidity
event log(string);
```

##### 예시

```solidity
emit log("here");
// here
```

<br></br>

---

#### logs

```solidity
event logs(bytes);
```

##### 예시

```solidity
emit logs(bytes("abcd"));
// 0x6162636400000000000000000000000000000000000000000000000000000000
```

<br></br>

---

#### log\_\<type\>

```solidity
event log_<type>(<type>);
```

여기서 `<type>`은 `address`, `bytes32`, `int`, `uint`, `bytes`, `string`일 수 있습니다.

##### 예시

```solidity
uint256 amount = 1 ether;
emit log_uint(amount);
// 1000000000000000000
```

<br></br>

---

#### log_named\_\<type\>

```solidity
event log_named_<type>(string key, <type> val);
```

여기서 `<type>`은 `address`, `bytes32`, `int`, `uint`, `bytes`, `string`일 수 있습니다.

##### 예시

```solidity
uint256 amount = 1 ether;
emit log_named_uint("Amount", amount);
// amount: 1000000000000000000
```

<br></br>

---

#### log_named_decimal\_\<type\>

```solidity
event log_named_decimal_<type>(string key, <type> val, uint decimals);
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

##### 예시

```solidity
uint256 amount = 1 ether;
emit log_named_decimal_uint("Amount", amount, 18);
// amount: 1.000000000000000000
```

### 어설션 (Asserting)

사용 가능한 모든 어설션 함수에 대한 전체 개요입니다. 자세한 설명과 사용 예시는 아래를 참조하세요.

```solidity
// `condition`이 true인지 확인
function assertTrue(bool condition) internal;
function assertTrue(bool condition, string memory err) internal;

// `a`가 `b`와 같은지 확인
function assertEq(address a, address b) internal;
function assertEq(address a, address b, string memory err) internal;
function assertEq(bytes32 a, bytes32 b) internal;
function assertEq(bytes32 a, bytes32 b, string memory err) internal;
function assertEq(int a, int b) internal;
function assertEq(int a, int b, string memory err) internal;
function assertEq(uint a, uint b) internal;
function assertEq(uint a, uint b, string memory err) internal;
function assertEqDecimal(int a, int b, uint decimals) internal;
function assertEqDecimal(int a, int b, uint decimals, string memory err) internal;
function assertEqDecimal(uint a, uint b, uint decimals) internal;
function assertEqDecimal(uint a, uint b, uint decimals, string memory err) internal;
function assertEq(string memory a, string memory b) internal;
function assertEq(string memory a, string memory b, string memory err) internal;
function assertEq32(bytes32 a, bytes32 b) internal;
function assertEq32(bytes32 a, bytes32 b, string memory err) internal;
function assertEq0(bytes memory a, bytes memory b) internal;
function assertEq0(bytes memory a, bytes memory b, string memory err) internal;

// `a`가 `b`보다 큰지 확인
function assertGt(uint a, uint b) internal;
function assertGt(uint a, uint b, string memory err) internal;
function assertGt(int a, int b) internal;
function assertGt(int a, int b, string memory err) internal;
function assertGtDecimal(int a, int b, uint decimals) internal;
function assertGtDecimal(int a, int b, uint decimals, string memory err) internal;
function assertGtDecimal(uint a, uint b, uint decimals) internal;
function assertGtDecimal(uint a, uint b, uint decimals, string memory err) internal;

// `a`가 `b`보다 크거나 같은지 확인
function assertGe(uint a, uint b) internal;
function assertGe(uint a, uint b, string memory err) internal;
function assertGe(int a, int b) internal;
function assertGe(int a, int b, string memory err) internal;
function assertGeDecimal(int a, int b, uint decimals) internal;
function assertGeDecimal(int a, int b, uint decimals, string memory err) internal;
function assertGeDecimal(uint a, uint b, uint decimals) internal;
function assertGeDecimal(uint a, uint b, uint decimals, string memory err) internal;

// `a`가 `b`보다 작은지 확인
function assertLt(uint a, uint b) internal;
function assertLt(uint a, uint b, string memory err) internal;
function assertLt(int a, int b) internal;
function assertLt(int a, int b, string memory err) internal;
function assertLtDecimal(int a, int b, uint decimals) internal;
function assertLtDecimal(int a, int b, uint decimals, string memory err) internal;
function assertLtDecimal(uint a, uint b, uint decimals) internal;
function assertLtDecimal(uint a, uint b, uint decimals, string memory err) internal;

// `a`가 `b`보다 작거나 같은지 확인
function assertLe(uint a, uint b) internal;
function assertLe(uint a, uint b, string memory err) internal;
function assertLe(int a, int b) internal;
function assertLe(int a, int b, string memory err) internal;
function assertLeDecimal(int a, int b, uint decimals) internal;
function assertLeDecimal(int a, int b, uint decimals, string memory err) internal;
function assertLeDecimal(uint a, uint b, uint decimals) internal;
function assertLeDecimal(uint a, uint b, uint decimals, string memory err) internal;

// `a`가 절대값의 델타(delta) 내에서 `b`와 거의 같은지 확인
function assertApproxEqAbs(uint256 a, uint256 b, uint256 maxDelta) internal;
function assertApproxEqAbs(uint256 a, uint256 b, uint256 maxDelta, string memory err) internal;

// `a`가 퍼센트 델타 내에서 `b`와 거의 같은지 확인 (`1e18`은 100%)
function assertApproxEqRel(uint256 a, uint256 b, uint256 maxPercentDelta) internal;
function assertApproxEqRel(uint256 a, uint256 b, uint256 maxPercentDelta, string memory err) internal;
```

### 어설션 함수 (Assertion functions)

이 섹션에서는 어설션을 위한 모든 함수와 사용 예시를 문서화합니다.

#### `assertTrue`

```solidity
function assertTrue(bool condition) internal;
```

`condition`이 true인지 확인합니다.

##### 예시

```solidity
bool success = contract.fun();
assertTrue(success);
```

<br></br>

---

#### `assertEq`

```solidity
function assertEq(<type> a, <type> b) internal;
```

여기서 `<type>`은 `address`, `bytes32`, `int`, `uint`일 수 있습니다.

`a`가 `b`와 같은지 확인합니다.

##### 예시

```solidity
uint256 a = 1 ether;
uint256 b = 1e18 wei;
assertEq(a, b);
```

<br></br>

---

#### `assertEqDecimal`

```solidity
function assertEqDecimal(<type> a, <type> b, uint decimals) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 `b`와 같은지 확인합니다.

##### 예시

```solidity
uint256 a = 1 ether;
uint256 b = 1e18 wei;
assertEqDecimal(a, b, 18);
```

<br></br>

---

#### `assertEq32`

```solidity
function assertEq32(bytes32 a, bytes32 b) internal;
```

`a`가 `b`와 같은지 확인합니다.

##### 예시

```solidity
assertEq(bytes32("abcd"), 0x6162636400000000000000000000000000000000000000000000000000000000);
```

<br></br>

---

#### `assertEq0`

```solidity
function assertEq0(bytes a, bytes b) internal;
```

`a`가 `b`와 같은지 확인합니다.

##### 예시

```solidity
string memory name1 = "Alice";
string memory name2 = "Bob";
assertEq0(bytes(name1), bytes(name2)); // [FAIL]
```

<br></br>

---

#### `assertGt`

```solidity
function assertGt(<type> a, <type> b) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 `b`보다 큰지 확인합니다.

##### 예시

```solidity
uint256 a = 2 ether;
uint256 b = 1e18 wei;
assertGt(a, b);
```

<br></br>

---

#### `assertGtDecimal`

```solidity
function assertGtDecimal(<type> a, <type> b, uint decimals) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 `b`보다 큰지 확인합니다.

##### 예시

```solidity
uint256 a = 2 ether;
uint256 b = 1e18 wei;
assertGtDecimal(a, b, 18);
```

<br></br>

---

#### `assertGe`

```solidity
function assertGe(<type> a, <type> b) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 `b`보다 크거나 같은지 확인합니다.

##### 예시

```solidity
uint256 a = 1 ether;
uint256 b = 1e18 wei;
assertGe(a, b);
```

<br></br>

---

#### `assertGeDecimal`

```solidity
function assertGeDecimal(<type> a, <type> b, uint decimals) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 `b`보다 크거나 같은지 확인합니다.

##### 예시

```solidity
uint256 a = 1 ether;
uint256 b = 1e18 wei;
assertGeDecimal(a, b, 18);
```

<br></br>

---

#### `assertLt`

```solidity
function assertLt(<type> a, <type> b) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 `b`보다 작은지 확인합니다.

##### 예시

```solidity
uint256 a = 1 ether;
uint256 b = 2e18 wei;
assertLt(a, b);
```

<br></br>

---

#### `assertLtDecimal`

```solidity
function assertLtDecimal(<type> a, <type> b, uint decimals) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 `b`보다 작은지 확인합니다.

##### 예시

```solidity
uint256 a = 1 ether;
uint256 b = 2e18 wei;
assertLtDecimal(a, b, 18);
```

<br></br>

---

#### `assertLe`

```solidity
function assertLe(<type> a, <type> b) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 `b`보다 작거나 같은지 확인합니다.

##### 예시

```solidity
uint256 a = 1 ether;
uint256 b = 1e18 wei;
assertLe(a, b);
```

<br></br>

---

#### `assertLeDecimal`

```solidity
function assertLeDecimal(<type> a, <type> b, uint decimals) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 `b`보다 작거나 같은지 확인합니다.

##### 예시

```solidity
uint256 a = 1 ether;
uint256 b = 1e18 wei;
assertLeDecimal(a, b, 18);
```

<br></br>

---

#### `assertApproxEqAbs`

```solidity
function assertApproxEqAbs(<type> a, <type> b, uint256 maxDelta) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 절대값 델타 내에서 `b`와 거의 같은지 확인합니다.

##### 예시

```solidity
function testRevert () external {
    uint256 a = 100;
    uint256 b = 200;

    assertApproxEqAbs(a, b, 90);
}
```

<br></br>

---

#### `assertApproxEqRel`

```solidity
function assertApproxEqRel(<type> a, <type> b, uint256 maxPercentDelta) internal;
```

여기서 `<type>`은 `int`, `uint`일 수 있습니다.

`a`가 퍼센트 델타 내에서 `b`와 거의 같은지 확인합니다. 여기서 `1e18`은 100%입니다.

##### 예시

```solidity
function testRevert () external {
    uint256 a = 100;
    uint256 b = 200;
    assertApproxEqRel(a, b, 0.4e18);
}
```

<br></br>

> ℹ️ **정보**
>
> 추가 매개변수 `string err`을 제공하여 위 함수들에 사용자 정의 오류 메시지를 전달할 수 있습니다.
