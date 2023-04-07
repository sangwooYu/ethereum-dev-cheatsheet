# ethereum-dev-cheatsheet
A cheatsheet for developing on the Ethereum platform

* [Ethereum](#cb): A description of how Ethereum works.
	* [Accounts](#cb-a) 
	* [Transactions](#cb-t)
* [Dapp Architecture](#da): A summary of Dapp architecture.
	* [Blockchain](#da-b)
	* [Nodes](#da-n)
	* [Frontend](#da-f)
* [Interacting with Contracts](#ic): Notes about ABIs
* [Solidity Cheatsheet](#sc): Summary of common things in Solidity
	* [Notes](#sc-g)
	* [Sending Messages](#sc-s)
* [Web3 Cheatsheet](#wc): Summary of common things in Web3
* [Truffle Cheatsheet](#tc): Summary of common things in Truffle
	* [Notes](#tc-n)
	* [Compiling/Building](#tc-c)
	* [Deploying/Testing](#tc-d) 

# <a name="cb"></a> Ethereum

이더리움은 실행 코드(컨트랙트라고 함)를 블록체인에 저장하고 다른 모든 계정에서 상호작용할 수 있는 블록체인입니다.
## <a name="cb-a"></a>Accounts

계정에는 두 가지 유형이 있습니다:

* 외부 소유 계정(일명 지갑)
* 컨트랙트

**두 계정 유형 모두**

* 잔액 보유(이더리움)
* 다른 계좌로 잔액 이체 가능

**외부 계정**

* 블록체인에 트랜잭션을 전송할 수 있습니다.
	* 트랜잭션은 단순히 이더를 전송하거나
	* 또는 컨트랙트 함수를 호출할 수 있습니다.
	* 또는 컨트랙트를 생성할 수 있습니다.
* 개인 키로 제어

**계약서**

* 는 '외부 계정'의 트랜잭션을 통해서만 작업을 수행합니다.
* 다른 컨트랙트에 메시지를 보낼 수 있습니다.
* 트랜잭션과 메시지를 통해 함수를 실행할 수 있습니다.
* 개인키가 없음(첫 번째 글머리 기호 참조)

## <a name="cb-t"></a>Transactions

### 중요

**블록체인의 모든 일은 '외부 계정'의 트랜잭션에 의해 시작됩니다.** 모든 일은 '외부 계정'이 블록체인에 트랜잭션을 전송하는 것에서 시작됩니다.

### 프로세스

#### 1) 트랜잭션 생성

'외부 계정'(발신자)이 트랜잭션을 생성할 때 다음을 지정합니다:

* **발신자**: 이 트랜잭션을 생성하는 계정
  **받는 사람**: 받는 사람: 트랜잭션의 대상 수신자
  **데이터**: 'To'가 컨트랙트인 경우, 어떤 함수를 어떤 파라미터로 호출할 것인가?
* **가스 가격**: 가스 1단위당 지불할 의향이 있는 금액(웨이 단위)
  **gasLimit**: 지불하고자 하는 총 가스 금액
  **서명**: 이 트랜잭션이 **발신자**의 거래임을 증명합니다.

참고: 발생하는 최대 거래 수수료는 **가스 가격**에 **가스 한도**를 곱한 금액입니다.  발신자는 최소한 이 '잔액'을 가지고 있어야 합니다.  트랜잭션에 적절한 서명이 있는지, 모든 것이 올바르게 인코딩되었는지 등 다른 유효성 검사도 진행됩니다.

트랜잭션이 대기열로 전송됩니다.  이 트랜잭션을 나타내는 `트랜잭션 해시`가 생성됩니다.

#### 2) 보류 중(채굴 대기 중)

마이너는 다음과 같이 작동합니다:

* 보류 중인 모든 트랜잭션을 확인합니다.
* 가스 가격이 가장 높은 트랜잭션을 찾습니다(가장 많은 수익을 올릴 것이기 때문에).
* 현재 블록에 들어갈 수 있는 최대한 많은 트랜잭션을 찾습니다(가스 및 메모리 제한이 있습니다).
* 해당 트랜잭션을 실행합니다.  각각에 대해
	* 트랜잭션이 실행될 때마다 얼마나 많은 가스가 사용되는지 계산합니다.  가스 제한`을 초과하면 즉시 중지하고 트랜잭션을 블록체인에 실패로 저장합니다.
	* 코드를 실행하고, 스토리지를 업데이트하고, 새로운 컨트랙트 데이터를 블록에 넣고, 내부 트랜잭션을 수행하는 등의 작업을 수행합니다.
	* 트랜잭션 수수료를 `발신자`에게 부과합니다(가스사용량×가스가격).
	* 모든 관련 정보를 블록에 추가합니다.
	* 블록에 `txReceipt`도 추가합니다.
* 블록 채굴을 시작합니다!

마이너가 채굴에 성공하면 이를 동료들에게 브로드캐스트합니다.  동료들은 결과가 정확한지 확인하기 위해 모든 트랜잭션을 실행하는 동일한 프로세스를 수행합니다.  블록을 수락하면 다음 블록에 대한 작업을 시작합니다.

이것이 첫 번째 '확인'입니다.  이 블록 이후에 추가로 채굴되는 블록은 모두 확인 블록입니다.

#### 3) Done

이 시점에서 `txHash`는 블록체인에 있으며 무슨 일이 일어났는지에 대한 모든 정보를 포함해야 합니다.  얼마나 많은 가스가 사용되었는지, 내부 메시지 등.

# <a name="da"></a>Dapp Architecture

"Dapp"은 정말 잘못된 이름이라고 생각합니다.  여러분에게는 세 가지 레이어가 있습니다:

## <a name="da-b"></a>Blockchain
블록체인에 배포된 스마트 컨트랙트는 작업을 수행하고 데이터를 저장합니다. 보통 솔리디티로 작성되지만, 사용성을 제외한 모든 것에 집착하는 지저분하고 느리고 매우 제한된 어셈블리 같은 언어인 'EVM'으로 컴파일되는 모든 언어로 작성할 수 있습니다.

## <a name="da-n"></a>Nodes
노드는 세상과 블록체인을 연결하는 미들웨어입니다.  노드는 RPC를 통해 세상에 노출됩니다. 따라서 노드에 HTTP 호출을 수행하여 다음과 같은 쿼리에 대한 답을 얻을 수 있습니다:

* 모든 블록 조회
* 모든 주소의 잔액 찾기
* 어떤 컨트랙트가 무엇을 저장하고 있는지 확인
* '상수' 컨트랙트 함수 호출(상태를 변경하지 않으므로)

내부적으로 노드는 이더리움 채굴자 네트워크의 일부이며, 새로운 블록이 채굴될 때 알림을 받습니다. 노드는 전체 블록체인을 포함합니다.

개발 시에는 블록이 채굴되는 것처럼 가장하고 자체 블록체인을 포함하는 가짜 노드인 TestRPC를 사용하게 됩니다. 매우 편리합니다.

## <a name="da-f"></a>FrontEnd
이것은 일부 노드에 HTTP 호출을 수행하고 그 결과를 사용하여 블록체인에 무엇이 있는지에 대한 UI를 표시하는 평범한 오래된 JS입니다.

브라우저와 존재하는 확장 기능에 따라 UI에는 "이 가스 한도와 이 가스 가격으로 이 매개변수를 전달하여 이 컨트랙트 함수를 호출하세요"와 같이 사용자에게 특정 세부 정보가 포함된 트랜잭션을 생성하라는 메시지가 포함될 수 있습니다.

이렇게 하면 메타마스크와 미스트는 사용자가 트랜잭션을 확인할 수 있는 UI를 표시합니다.

### 메타마스크 관련

프론트엔드에서 메타마스크가 확인할 트랜잭션을 생성할 때, 메타마스크는 다음 정보 중 어느 것도 표시하지 않습니다:

* 전체 주소(또는 이더스캔에서 주소로 연결되는 링크)
* 컨트랙트 함수 이름
* 함수 매개변수

기본적으로 "이 웹사이트에서 이 거래를 하고 싶어하는데, 이 거래에 대해서는 아무 말도 하지 않겠습니다.  괜찮으세요?"

저는 미스트를 사용해 보지 않았기 때문에 그렇게 어리석은지는 모르겠습니다.

# <a name="ic"></a>ABI 및 계약과의 상호 작용

컨트랙트가 블록체인에 배포될 때, 컨트랙트는 어떻게 상호작용할 수 있는지에 대한 정보가 전혀 포함되어 있지 않습니다.  기본적으로 컨트랙트는 거대한 데이터 덩어리이며, 원하는 결과를 얻으려면 이를 '적절하게' 호출해야 합니다.

이는 사용자 친화적이지 않은 가학적인 접근 방식이지만, 블록당 몇 KB를 줄이거나 마이닝 시간을 몇 마이크로초 단축하는 등 "좋은" 이유가 있다고 생각합니다.

어쨌든 콘트랙트와 제대로 상호작용하기 위해서는 ABI를 알아야 합니다.  ABI는 유효한 호출을 수행할 수 있도록 사용 가능한 모든 함수(및 해당 매개변수 유형)를 포함합니다.  ABI는 컨트랙트 내부에 있는 스키마라고 생각하면 됩니다.

내부적으로 클라이언트는 사용자가 수행하려는 작업을 트랜잭션의 '데이터' 부분으로 변환합니다.  이에 대한 전체 사양은 이더리움 문서 어딘가에서 찾을 수 있지만, 알고 싶지 않으실 수도 있습니다.

# <a name="sc"></a>Solidity

## <a name="sc-g"></a>Gotchas

* 문자열로 행운을 빕니다.
* 부동 소수점이 없습니다.  실제로 `a/b`를 수행하지 않습니다.  대신 `(일부 값 * a)/b`를 수행합니다. 개 멍청하죠.
* 외부 트랜잭션은 반환 값을 받을 수 없습니다.  네, 진심입니다.
* 함수는 동적 배열을 반환할 수 없습니다.

## <a name="sc-s"></a>Sending Messages (Internal Transactions)

아래 예에서 `addr`은 수신자 주소입니다.

---

`bool _success = addr.send(valueInWei)`

* 'addr'이 컨트랙트인 경우 폴백 함수를 호출합니다.
* 성공 여부에 대해 bool을 반환합니다.
* 소량의 가스(2300)를 전송합니다.

---

`addr.transfer(valueInWei)`

* 'addr'이 컨트랙트인 경우 폴백 함수를 호출합니다.
* 실패 시 throw
* 소량의 가스(2300)를 전송합니다.

---

`bool _succees = addr.call.value(valueInWei).gas(uint)();`

* 'addr'이 컨트랙트인 경우 폴백 함수를 호출합니다.
* 성공 여부에 대해 bool을 반환합니다.
* 사용자 지정 가스 양을 전송합니다(기본값은 0: 무제한).
* 사용자 지정 값 전송

---

`bytes4 _signature = bytes4(sha3("fnName(param1type,param2type)"))`
`bool _success = addr.call.value().gas()(_signature, param1, param2...)`

* 참 또는 거짓을 반환합니다.
* 사용자 지정 가스 양을 전송합니다(기본값은 0: 무제한).
* 사용자 지정 값 전송

---

`var _returnedValue = addr.someFunction(param1, param2,...);`

* 함수가 반환하는 모든 것을 반환합니다.
* 무제한 가스 전송
* 값을 설정할 수 없음
* !! 참고: 솔리디티는 `addr`의 유형과 `someFunction`이 있다는 것을 알아야 합니다.
---

`var _returnedValue = addr.someFunction.value(valueInWei).gas(uint)(param1, param2, ...);`

* 함수가 반환하는 모든 것을 반환합니다.
* 사용자 지정 가스 양을 전송합니다(기본값은 0: 무제한).
* 사용자 지정 값 전송
* !! 참고: 솔리디티는 `addr`의 유형과 `payable someFunction`이 있다는 것을 알아야 합니다.

---

여러 반환 값을 처리하는 방법을 아는 것이 유용할 수 있습니다.

여기:

```
var (return1, return2) = ...
```

또는 유형을 알고 있는 경우

```
(uint _re1, bytes32 _ret2) = ...
```



# Web3 / truffle-contract

트러플 컨트랙트는 기본적으로 Web3이지만 약속을 반환합니다.  다른 기능은 잘 모르겠습니다.

## Transactions

### Manually

여러분이 처리하는 대부분의 모든 것은 트랜잭션이 될 것이며, 어떤 식으로든 `sendTransaction`을 호출할 것입니다.  [Web3 문서는 여기에서 찾을 수 있습니다](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethsendtransaction).

트랜잭션을 수동으로 생성하는 방법은 다음과 같습니다:

예시:

```
var options = {
	// String - address this is from (and will be signed by)
	from: "0xabc...",
	// String - address of who its to
	to: "0xdef...",
	// Number|String|BigNumber - how much wei to send
	value: 1000,
	// Number|String|BigNumber - maximum gas to be used
	gas: 1e10,
	// Number|String|BigNumber - price of gas, defaults to mean network gas price
	gasPrice: 22e12 // (22 gwei),
	// String - optional byte string of contract call with params, or contract creation code
	data: "0x3832...",
	// Number - allows you to overwrite your own pending transactions
	nonce: 123
}
var txHash = web3.eth.sendTransaction(options);
```

Web3가 txHash를 반환합니다... 사전 채굴된 것인지 사후 채굴된 것인지 잘 모르겠습니다.

트러플 컨트랙트는 객체로 이행된 약속을 반환합니다:

```
{
	tx: "0x123...",
	receipt: {
		transactionHash: '0x4b0cb3d24b374b27eb05dda5343e3435208c18171774afdd0c7147b9c80894cf',
    	transactionIndex: 0,
    	blockHash: '0x4513c0bb872b86f5c741d6b71f4373d7fda94ee4b865ce39423c94e29d03b3c5',
    	blockNumber: 2556,
    	gasUsed: 830927,
    	cumulativeGasUsed: 830927,
    	contractAddress: null,
    	logs: [ [Object], [Object], [Object] ]
	}
	
	// Only when called via a contract instance
	// These will be logs specific to this object.
	logs: [{ ...log1... }, { ...log2... }],
}
```

**참고: 이더리움은 매우 훌륭하기 때문에 (반어법) 트랜잭션을 수행할 때 반환값을 얻을 수 없습니다.**

**참고: 전송하는 컨트랙트 이외의 주소가 주제 이름과 일치하는 경우 로그에 일치하는 항목이 포함됩니다.  이는 웹3.0의 버그입니다. 예를 들어, 컨트랙트에 "Foo"라는 이벤트가 있고 다른 컨트랙트가 "Foo" 이벤트를 로그하는 경우, 모든 "Foo"가 로그에 표시됩니다 **.

### Using ABIs

Web3와 트러플은 ABI를 사용하여 일부 트랜잭션 매개변수를 채웁니다.

```
myContract.doStuff(arg1, arg2, options).then( ... )
```

내부적으로 이것은 sendTransaction을 수행하지만 `to` 및 `data`를 채웁니다.

Web3와 트러플 컨트랙트는 안타깝게도 명명된 매개변수를 사용하지 않습니다.  그리고 일부 인수를 생략하면 상황이 정말 엉망이 됩니다.  그러니 조심하세요.

### Calls

통화를 하는 것은 TX와 다릅니다.  네트워크를 건드리지 않으므로 상태가 변경되지 않습니다.  하지만 최소한 반환값은 돌려받을 수 있습니다.

_(Side note: I'm not sure what happens if you do `if (addr.send()) { return 1; } else { return 0; } ` inside of a call)_

어쨌든, `.call`을 붙이고 거기에 매개 변수를 전달하는 것을 제외하고는 위와 거의 동일합니다.  왜 그냥 `.asCall()`에 붙일 수 없는지 모르겠습니다.

## Events

### Watching block events

[Official docs on filters.](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethfilter)  These docs are pretty good.

Todo: put code samples.

### Watching contract instance events

[Official event docs here.](https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events)  These docs are pretty good.

특정 객체 인스턴스에서 이벤트를 감시할 수 있습니다.

Use either `instance.<EventName>(eventFilter, filterOpts)` or `instance.allEvents(filterOps);`.  They both return the same thing.

```
// a filter object.  see above section for details
// note:  I'm not sure what the default fromBlock is
var filterOpts = { ... };

// an object whose keys match the args of events
// and whose values will be used to filter
var eventFilter = {
	arg1: "value must match this",
	arg2: ["can be this", "or this"]
};
var eventFilter = null;

var watcherForEventName = myContract.EventName(eventFilter, filterOpts);
var watcherForAll = myContract.allEvents(filterOpts);
console.log(watcherForEventName.get());

/*
logs the following in truffle-contract:
[
	{
		logIndex: 0,
    	transactionIndex: 0,
    	transactionHash: '0x36caf6b32b87a5145581df63d02fb02857d8935977721d324e8daadb6f2663f3',
   		blockHash: '0x6cec9e3a2ec6972291f71650758001e258f3f973b7fc10a0f81de24b468306f6',
    	blockNumber: 2168,
    	address: '0x8f7d11d92a76d10107f14f42142c4409e2e0a37c',
    	type: 'mined',
    	event: 'EventName',
    	args: {
    		arg1: <value>,
    		arg2: <value>
    	}
    }, {...}    
]
*/

```

## Functions

# <a name="tc"></a>Truffle

## <a name="tc-n"></a>Notes

* node8 async/await을 사용하면 배포와 테스트 모두에서 훨씬 더 즐겁게 작업할 수 있습니다.

```
module.exports = function(deployer, network, accounts) {
	deployer.then(async function(){
		console.log("Deploying first thing...");
		await deployer.deploy(FirstContract);
		firstContract = deployer.at(FirstContract.address);
		
		console.log("Deploying the second thing...");
		await deployer.deploy(SecondContract);
		...
```

## <a name="tc-c"></a>Compiling / Building

* 파일 `B.sol`로 가져온 파일 `A.sol`을 편집하는 경우, `truffle compile`은 `B.sol`을 다시 컴파일하지 않습니다.  트러플 컴파일 --all`을 사용하는 습관을 들이는 것이 좋습니다.
* If you experience a `solc` exception about `5 5`, it's because there is a syntax error somewhere in _any_ of your files.  You're fucked.
* 트러플-컴파일`의 포크를 사용 중이라면 `truffle compile --parse`를 실행하면 구문 분석하려는 파일에서 구문 오류를 확인할 수 있습니다.  (이 기능은 현재 풀 리퀘스트입니다.)

## <a name="tc-d"></a>Deploying / Migrating

* 배포 함수에서 프로미스를 반환할 수 없으며 무시됩니다.  트러플은 `deployer` 프로미스 체인이 완료되는 즉시 단일 마이그레이션이 완료된 것으로 간주합니다.  모든 것을 deployer.then() 안에 넣는 것을 고려하세요:

```
// incorrect -- truffle will continue to next deployment because it thinks this is finished
module.exports = async function(deployer, network, accounts) {
	await deployer.deploy(SomeContract);
	// as soon as the last deployer.deploy() or .then() is done
	// truffle thinks deployment is done.  anything below will
	// be run _after_... which can screw stuff up.
	var c = SomeContract.at(SomeContract.address);
	await c.doSomeCall();
	
// correct -- truffle will wait for this to finish before continuing
module.exports = function(deployer, network, accounts) {
	// note: async function always returns a promise
	deployer.then(async function(){
		await deployer.deploy(SomeContract);
		var c = SomeContract.at(SomeContract.address);
		await c.doSomeCall();
```

* `deployer.deploy` 버그: 정확한 생성자 개수를 전달하지 않으면 생성자를 하나도 전달하지 않습니다.  이에 대한 유효성 검사도 수행하지 않습니다.
* 
```
// assume SomeContract takes 3 args
// incorrect - This creates SomeContract with empty args.
deployer.deploy(SomeContract, 5, 5);
// correct
deployer.deploy(SomeContract, 5, 5, 5);
```

* `deployer.deploy` returns a promise fulfilled with _nothing_... not even the address.  To get the address of something deployed:

```
await deployer.deploy(SomeContract);
var c = SomeContract.at(SomeContract.address);

// or using promises
deployer.deploy(SomeContract).then(function(){
	var c = SomeContract.at(SomeContract.address);
})
```

# Testing

## Fast Forwarding TestRPC
작성하는 많은 계약이 시간 기반일 수 있으므로 계약이 설계된 대로 작동하는지 확인하기 위해 테스트RPC를 '빨리 감기'하고 싶을 수 있습니다.  저는 이 방법을 찾기 위해 얼마나 열심히 검색해야 했는지 놀랐습니다.

아무튼, 웹3를 사용하여 이를 수행하는 방법은 다음과 같습니다:
```
function fastForward(timeInSeconds){
	if (!Number.isInteger(timeInSeconds))
		throw new Error("Passed a non-number: " + timeInSeconds);
	if (timeInSeconds <= 0)
		throw new Error("Can not fastforward a negative amount: " + timeInSeconds);
	
	// move time forward.
	web3.currentProvider.send({
        jsonrpc: "2.0",
        method: "evm_increaseTime",
        params: [timeInSeconds],
        id: new Date().getTime()
    });
	// mine a block to make sure future calls use updated time.
	web3.currentProvider.send({
        jsonrpc: "2.0",
        method: "evm_mine",
        params: null,
        id: new Date().getTime()
    });
}
```

# Patterns

## Upgrading Smart Contracts

저는 개인적으로 싱글톤 컨트랙트에 대한 `이름=>주소` 매핑을 포함하는 `레지스트리` 컨트랙트를 배포한 적이 있습니다.  

제 컨트랙트들이 서로 통신해야 할 때, 그들은 _항상_ 레지스트리에 먼저 요청합니다.

싱글톤을 업그레이드해야 할 때는 새 인스턴스를 배포하고 해당 인스턴스의 레지스트리 이름을 업그레이드합니다.  모든 컨트랙트가 항상 레지스트리에 최신 이름을 요청하기 때문에 추가 조치가 필요하지 않습니다.

그러나 이것은 싱글톤 컨트랙트가 많은 상태를 저장하지 않고 새 버전을 배포할 때 상태를 복사하기 때문에 작동합니다.

매핑과 같이 방대하거나 알 수 없는 상태를 복사해야 하는 경우에는 '릴레이'(일명 '델리게이트콜') 패턴을 사용해야 합니다.

[For more on upgrading contracts, go here](https://github.com/ConsenSys/smart-contract-best-practices#eng-techniques)

## Handling errors:  Returning vs Throwing

솔리디티에서 `throw`를 하면 아무것도 반환되지 않습니다.  오류 메시지가 없습니다.  흔적도 없습니다.  아무것도.  이건 정말 멍청한 짓이고, 테스트를 악몽으로 만듭니다.

For example:

```
contract Foo {
	function doStuff(){
		require(msg.sender == "0xABC...");
		require(msg.value > 100);
		... other code ...
	}
}
```


```
it("fails when wrong amount is sent", function(done){
	myFoo.doStuff({from: "0xBCA...", 90})
		.then(function(){ done("We expected this call to fail"); )
		.catch(done);
});
```

이 테스트는 통과하지만 잘못된 이유로 실패했습니다.  잘못된 주소를 전달했기 때문에 실패했습니다.  잘못된 금액을 전송했기 때문에 테스트가 실패한 것이 아닙니다.
[Anyway, read more about returning vs throwing here.](https://ethereum.stackexchange.com/questions/10046/throw-vs-return)