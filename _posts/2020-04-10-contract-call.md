---
layout: post
title:  "silidity contract call"
author: violetstair
categories: [ Programming ]
tags: [ethereum, blockchain, programming, solidity]
image: assets/images/solidity.svg
description: "다른 컨트랙트 호출하기"
featured: true
hidden: true
---

smart contract에서 다른 smart contract 호출하기

#### 테스트 컨트랙트

```javascript
contract Target {

    address _last_sender;

    constructor () public {
        _last_sender = msg.sender;
    }

    function set() public returns (address) {
        _last_sender = msg.sender;
        return msg.sender;
    }

    function get() public view returns (address) {
        return _last_sender;
    }
}
```

`set()`를 호출하면 `msg.sender`의 주소를 `_last_sender`에 저장하고 `msg.sender`를 반환하는 간단한 contract를 통해 호출 구조를 확인할 수 있다

코드 테스트를 통해 동작 확인

```javascript
describe("target contract 실행", () => {
    it("set", async () => {
        let last_sender = await this.target.get()
        assert(last_sender.should.be.equal(owner1))

        await this.target.set({from: user1})

        last_sender = await this.target.get()
        assert(last_sender.should.be.equal(user1))
    })
})
```

#### .call() / .delegatecall() / .staticcall()

컨트랙트를 호출하는 방법으로 `.call()` / `.delegatecall()` / `.staticcall()` 세가지 방법이 있으며

기본적인 호출 형태는 다음과 같다

```javascript
(bool success, bytes memory data) = contract_address.call(abi.encodeWithSignature("function(data_type)", [pamrms]));
```

컨트랙트의 기능을 실행하면 `bool`과 `bytes` type의 두개의 데이터가 반환되며 각각 실행 성공 여부와 반환되는 데이터를 의미한다

`bytes` 형태로 반환된 데이터는 `abi.decode()`를 통해 디코딩 후 사용 가능하다

```javascript
function get() public view returns (address, address[], uint256) {}
```

위오 같은 함수를 호출한 결과를 디코딩 하려면 다음과 같이 변환해 사용할 수 있다

```javascript
(address a, address[] b, uint256 c) = abi.decode(data, (address, address[], uint256));
```

`Target` 컨트랙트를 호출하는 `Caller` 컨트랙트를 통해 `.call()` / `.delegatecall()` / `.staticcall()`의 차이점을 확인할 수 있다

```javascript
contract Caller {
    address _sender;
    function remotecall(address _contract) public {
        (bool success, bytes memory data) = _contract.call(abi.encodeWithSignature("set()"));
        require(success, "contract call failed");
        (address a) = abi.decode(data, (address));
        _sender = a;
    }

    function remotedelegatecall(address _contract) public {
        (bool success, bytes memory data) = _contract.delegatecall(abi.encodeWithSignature("set()"));
        require(success, "contract delegatecall failed");
        (address a) = abi.decode(data, (address));
        _sender = a;
    }

    function get() public view returns (address) {
        return _sender;
    }

    function remotestaticcall(address _contract) public view returns (address) {
        (bool success, bytes memory data) = _contract.staticcall(abi.encodeWithSignature("get()"));
        require(success, "contract staticcall failed");
        (address a) = abi.decode(data, (address));
        return a;
    }
}
```

##### .call()

`.call()`은 호출 대상 컨트랙트를 실행하고 상태를 변경할 수 있다.

`Target` 컨트랙트에서 조회되는 `msg.sender`는 `.call()`을 실행시킨 컨트랙트의 주소 정보를 가지게 된다

```javascript
await this.caller.remotecall(this.target.address, {from: user1})
const address1 = await this.target.get()
assert(address1.should.be.equal(this.caller.address))

const address2 = await this.caller.get()
assert(address2.should.be.equal(this.caller.address))
```

* `.call()`를 호출하면 `Target`의 `_last_sender`를 변경시킬 수 있다
* `.call()`를 통해 호출한 `set()`의 `msg.sender`는 `remotecall()`를 실행시킨 컨트랙트의 주소 정보이다

##### .delegatecall()

`.delegatecall()`는 호출 대상 컨트랙트를 실행은 가능하지만, 호출 대상 컨트랙트의 상태는 변경할 수 없다

`Target` 컨트랙트에서 `msg.sender`, `msg.data`는 `Caller`의 `msg.sender`, `msg.data`의 정보와 같다

```javascript
await this.caller.remotedelegatecall(this.target.address, {from: user1})
const address1 = await this.target.get()
assert(address1.should.be.equal(owner1))

const address2 = await this.caller.get()
assert(address2.should.be.equal(user1))
```

* `.delegatecall()`를 호출해도 `Target`의 `_last_sender`는 변경되지 않는다
* `.delegatecall()`를 통해 호출한 `set()`의 `msg.sender`는 `remotedelegatecall()`를 실행시킨 `msg.sender`, 즉 트랜젝션을 실행시킨 `address`라는 것을 확인할 수 있다

##### .staticcall()

`.staticcall()`은 상태 변경을 하지 않고 조회만 가능하다 `view`, `pure` 속성의 함수에서만 사용할 수 있다.

#### 주의사항

다른 컨트랙트의 인터페이스를 정의할 때 띄어쓰기를 할 경우 오류가 발생한다

```javascript
contract_address.call(abi.encodeWithSignature("set(uint256, address)", _amoumt, address)); // 에러 발생

contract_address.call(abi.encodeWithSignature("set(uint256,address)", _amoumt, address)); // 정상 동작
```
