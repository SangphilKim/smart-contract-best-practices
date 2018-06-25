# 이더리움 스마트 컨트렉트 보안 습관들

이 문서는 중급 솔리디티 프로그래머들을 위해 보안 고려사항들의 기초 지식을 제공합니다.
이것은 [ConsenSys Diligence](https://media.consensys.net/introducing-consensys-diligence-cf38f83948c)와 폭넓은 이더리움 커뮤니티들에 의해 유지됩니다.

[![Join the chat at https://gitter.im/ConsenSys/smart-contract-best-practices](https://badges.gitter.im/ConsenSys/smart-contract-best-practices.svg)](https://gitter.im/ConsenSys/smart-contract-best-practices?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## 어디서부터 시작하죠?

* [일반적인 철학 (General Philosophy)](./general_philosophy.md) 스마트 컨트렉트 보안 사고방식에 대해 설명합니다.
* [솔리디티 추천들(Solidity Recommendations)](./recommendations.md) 바람직한 코드 패턴의 예시를 담았습니다.
* [Known Attacks](./known_attacks.md) 다른 클래스들 간의 취약성을 피하는 방법을 설명합니다.
* [Software Engineering](./software_engineering.md) 몇개의 아키텍쳐의 개요와 위험완화를 위한 디자인 접근법의 개요입니다.
* [Documentation and Procedures](./documentation_procedures.md) 외부 개발자들과 감사원들(Auditors)을 위한 시스템 문서 작성법 연습 개요입니다.
* [Security Tools](./security_tools.md) 취약성 탐지와 코드 수준 발전을 위한 툴 리스트 입니다.
* [Security Notifications](./security_notifications.md) 최신 상태를 유지하기 위한 정보 소스 리스트 입니다.
* [Tokens](./tokens.md) 토큰과 관련된 연습 개요 입니다.

## 기여자(Contributions)를 환영합니다!

작은 수정이나, 새로운 섹션 추가를 자유롭게 풀 리퀘스트(pull request)해주세요. 만약에 새로운 컨텐츠 내용을 작성하신다면, [contributing](./about/contributing.md)페이지 안에 스타일 안내 방식으로 언급해주세요.  

필요한 보완이나 업데이트 주제들은 [issues](https://github.com/ConsenSys/smart-contract-best-practices/issues)에서 볼 수 있습니다. 만약 당신의 아이디어에 대해 토론하기 원한다면, [Gitter](https://gitter.im/ConsenSys/smart-contract-best-practices)에서 우리와 채팅 할 수 있습니다.

글이나 블로그 포스팅을 하신다면, [bibliography](./bibliography.md)에 추가 해 주세요.


## 일반적인 철학 (General Philosophy)
이더리움과 복잡한 블록체인 프로그램들은 새롭고 아주 실험적입니다. 그러므로, 당신은 새로운 버그들이나 보안 위험을 발견하는 것과 같은 보안 환경에 대해 끊임없는 변화를 예상하셔야 합니다. 이 문서의 보안 연습은 기초적인 단계의 보안이므로 스마트 컨트렉트 개발자로서는 더 공부하셔야 합니다.

스마트 컨트렉트 프로그래밍은 당신이 해오던 방식과 다른 엔지니어링 사고 방식이 필요합니다. 실패 비용(The cost of failure)이 높을 수도 있고, 변경하기 까다롭고, 하드웨어 프로그래밍 또는 금융 서비스 프로그래밍과 유사한 방법으로 웹이나 모바일 환경에서 만들어야 합니다. 그러니 알려진 취약성 방어기법으로는 충분하지 않아서 새로운 개발 철학에 대해 학습 필요성을 느끼실 것 입니다:

## 실패를 대비

컨트렉트 안에 에러 중 사소하지 않은 건 없습니다. 그러므로 당신의 코드는 버그 대응과 취약성을 유연하게 대처해야 합니다.

  - 컨트렉트에 무언가 잘못 되어가고 있으면 멈춤 ('써킷 브레이커')
  - 위험이 있는 자금의 액수를 관리 (비율 제한, 최대 사용량)
  - 버그들 수정과 개선을 위한 효과적인 업그레이드 방법

## 신중한 공개

전체 제품 공개 전에 항상 버그를 잡을 수 있도록 하세요.

  - 철저하게 테스트 컨트렉트를 작성하고, 새로운 공격 벡터를 발견하면 테스트 항목에 추가
  - [버그 바운티](#bounties)를 제공하고 알파 테스트넷 공개 때 부터 시작
  - 단계별 공개, 각 단계별 테스트와 사용량 증가

## 컨트렉트의 단순함 유지

복잡성은 에러 발생 가능성을 증가시킵니다.

  - 컨트렉트 구조를 반드시 간단하게 구성
  - 코드를 모듈화해 컨트렉트와 함수를 작게 유지
  - 이미 사용된 툴들 또는 이용 가능한 코드 사용(예. 랜덤 숫자 생성을 스스로 돌리지 마세요)
  - 가능한 퍼포먼스 명확성 선호  
  - 분산화가 필요한 시스템 부분은 블록체인만 사용

## 최신 버전으로 유지

자원 리스트의 다음 부분으로 새로운 보안 개발 관심 유지를 한다.

  - 발견된 새로운 버그에 대한 컨트렉트 확인
  - 가능한 툴이나 라이브러리를 최신버전으로 업그레이드
  - 유용하다고 인정된 새로운 보안 기술 채택

## 블록체인의 특징을 이해

대부분의 프로그래밍은 이더리움 프로그래밍과 관련된 경험일텐데, 몇가지 위험을 아셔야 합니다.

  - 외부(external) 컨트렉트 호출(call)을 아주 주의, 악성코드가 실행되거나 통제 흐름이 변경될 수도 있음
  - 공공(public) 함수는 공공의 성질을 가지고 있으며, 악의적으로 호출 될 수 있음을 이해. 개인적인(private) 데이터도 아무나 볼 수 있음
  - 가스 비용과 블록 가스 제한을 명심

## 펀더멘탈 트레이드오프 (Fundamental Tradeoffs) : 단순함과 복잡함 경우

스마트 컨트렉트 시스템 보안과 구조를 평가할때 몇가지 펀더멘탈 트레이드오프가 고려됩니다. 스마트 컨트렉트 시스템의 상충 관계에 대한 적절한 균형을 확인하기 위해 일반적으로 추천 합니다.

소프트웨어 엔지니어링 측면에서 이상적인 스마트 컨트렉트 시스템은 모듈화이며, 복제 대신 코드를 재사용하며, 컴포넌트가 업그레이드 가능하게 하는 것입니다. 안전한 아키텍쳐(secure architecture)측면에서 이상적인 스마트 컨트렉트 시스템은 유난히 복잡한 스마트 컨트렉트 시스템의 경우를 포함해 이런 사고방식을 공유하는 것입니다.

하지만, 중요한 예외사항으로 소프트웨어 엔지니어링 모범 규칙과 보안을 동일시 하지 않는 것입니다. 모든 경우에, 아래 컨트렉트 시스템 관점 사이에서 최적의 방법을 섞어 적절한 균형을 얻는 것입니다:

- 업그레이드 불가능과 업그레이드 가능
- 단일 방식과 모듈 방식
- 복제와 재사용

### 업그레이드 불가능과 업그레이드 가능

여러 리소스들을 사용하는 동안에 Killable과 같이 유연성이 강조된 형태거나, 업그레이드가 가능하거나 수정가능한 패턴은 *펀더멘탈 트레이드오프* 의 유연성(malleability)과 보안성(security) 사이에 있습니다.

유연하게 정의된 패턴은 복잡성과 잠재적인 공격면을 추가하게 됩니다. 단순함은 스마트 컨트렉트 시스템의 짧은 시간동안 사전 정의된 아주 제한적인 기능의 복잡한 성능을 뛰어넘는데 효과적입니다. 예를 들면, 한정된 시간동안 관리자 없이 토큰 세일을 하는 컨트렉트 시스템과 같습니다.

### 단일 방식과 모듈 방식

단일 방식 컨트렉트는 지역적으로 정의된 식별가능와 조회가능이 모두 포함 되어 있습니다. 단일 방식으로 존재하는 몇개의 스마트 컨트렉트 시스템은 높은 관심을 가지게 되며, 극도로 지역적인 데이타와 흐름을 가지는 인수(argument)가 생깁니다 - 예를 들면, 코드 최적화(optimizing code) 효율성 검토 같은 경우 입니다.

여기서 고려되는 다른 상충관계(tradeoffs)와 마찬가지로, 보안 사례들(best practices)은 간단하면서 오래가지 못하는 소프트웨어 엔지니어링 사례와 멀어지는 경향이 있고, 더 복잡하면서 빈번하게 계속되는 컨트렉트 시스템 경우의 소프트웨어 엔지니어링 사례 쪽으로 기울게 됩니다.

### 복제와 재사용

소프트 엔지니어링 스마트 컨트렉트 시스템은 합당한 재사용을 최대화하는 것을 바라고 있습니다. 솔리디티 컨트렉트 코드 재사용 방법은 여러가지가 있습니다. 전에 디플로이(deployed) 된 *당신이 가지고 있는* 증명된 컨트렉트를 사용하는 것은 일반적으로 코드를 재사용하기 위한 안전한 방법 입니다.

복제는 당신이 가지고 있는 컨트렉트 중 전에 디플로이 되었던 컨트렉트를 사용할 수 없는 경우에 빈번하게 의존하게 됩니다. 복제 없이 재사용 가능한 안전한 코드 패턴을 찾아 제공하기 위해 [라이브 립스(Live Libs)](https://github.com/ConsenSys/live-libs)와 [제플린 솔리디티(Zeppelin Solidity)](https://github.com/OpenZeppelin/zeppelin-solidity)는 노력하고 있는 중 입니다. 모든 컨트렉트 보안 분석은 스마트 컨트렉트 시스템 목표의 자금 위험과 신뢰할만한 수준으로 사전에 인증(established)받지 못한 재사용 코드도 모두 포함 되어야 합니다.


## 솔리디티 스마트 컨트렉트 보안 권장사항들 (Recommendations for Smart Contract Security in Solidity)
이 페이지는 스마트 컨트렉트를 작성할때 일반적으로 따르는 솔리디티 패턴 몇가지를 보여줍니다.

## 프로토콜 별 권장사항들

아래 권장사항들은 이더리움의 컨트렉트 시스템 개발에 적용됩니다.

## 외부 호출(External Calls)

### 외부 호출를 구성할때 주의하세요

믿을 수 없는 컨트렉트의 호출은 몇가지 기대하지 않은 위험이나 에러를 야기할 수 있습니다. 외부 호출은 다른 컨트렉트에 의존하는 컨트렉트 _또는_ 컨트렉트 내부의 악성 코드를 실행 시킬 수 있습니다. 보통 말하는 그런 모든 외부 호출은 잠재적인 보안 위험으로 처리해야 합니다. 그게 불가능해 지거나 외부 호출을 제거할 수 없다면, 이 섹션의 뒷부분에 있는 권장사항을 이용해 위험을 최소화 하세요.

### 믿을 수 없는 컨트렉트 표시 //Mark untrusted contracts

When interacting with external contracts, name your variables, methods, and contract interfaces in a way that makes it clear that interacting with them is potentially unsafe. This applies to your own functions that call external contracts.

```sol
// bad
Bank.withdraw(100); // 신뢰할 수 있는지 없는지 불명확함

function makeWithdrawal(uint amount) { // 이 함수가 잠재적으로 불안한지에 대해 불명확함
    Bank.withdraw(amount);
}

// good
UntrustedBank.withdraw(100); // 신뢰할수 없는 외부 호출
TrustedBank.withdraw(100); // 외부에 있지만 XYZ 회사에서 유지하고 있는 믿을 수 있는 은행 컨트렉트

function makeUntrustedWithdrawal(uint amount) {
    UntrustedBank.withdraw(amount);
}
```


### Avoid state changes after external calls

Whether using *raw calls* (of the form `someAddress.call()`) or *contract calls* (of the form `ExternalContract.someMethod()`), assume that malicious code might execute. Even if `ExternalContract` is not malicious, malicious code can be executed by any contracts *it* calls.

One particular danger is malicious code may hijack the control flow, leading to race conditions. (See [Race Conditions](./known_attacks#race-conditions) for a fuller discussion of this problem).

If you are making a call to an untrusted external contract, *avoid state changes after the call*. This pattern is also sometimes known as the [checks-effects-interactions pattern](http://solidity.readthedocs.io/en/develop/security-considerations.html?highlight=check%20effects#use-the-checks-effects-interactions-pattern).


### Be aware of the tradeoffs between `send()`, `transfer()`, and `call.value()()`

When sending ether be aware of the relative tradeoffs between the use of
`someAddress.send()`, `someAddress.transfer()`, and `someAddress.call.value()()`.

- `someAddress.send()`and `someAddress.transfer()` are considered *safe* against [reentrancy](./known_attacks#reentrancy).
    While these methods still trigger code execution, the called contract is
    only given a stipend of 2,300 gas which is currently only enough to log an
    event.
- `x.transfer(y)` is equivalent to `require(x.send(y));`, it will automatically revert if the send fails.
- `someAddress.call.value(y)()` will send the provided ether and trigger code execution.  
    The executed code is given all available gas for execution making this type of value transfer *unsafe* against reentrancy.

Using `send()` or `transfer()` will prevent reentrancy but it does so at the cost of being incompatible with any contract whose fallback function requires more than 2,300 gas. It is also possible to use `someAddress.call.value(ethAmount).gas(gasAmount)()` to forward a custom amount of gas.

One pattern that attempts to balance this trade-off is to implement both
a [*push* and *pull*](#favor-pull-over-push-for-external-calls) mechanism, using `send()` or `transfer()`
for the *push* component and `call.value()()` for the *pull* component.

It is worth pointing out that exclusive use of `send()` or `transfer()` for value transfers
does not itself make a contract safe against reentrancy but only makes those
specific value transfers safe against reentrancy.


### Handle errors in external calls

Solidity offers low-level call methods that work on raw addresses: `address.call()`, `address.callcode()`, `address.delegatecall()`, and `address.send()`. These low-level methods never throw an exception, but will return `false` if the call encounters an exception. On the other hand, *contract calls* (e.g., `ExternalContract.doSomething()`) will automatically propagate a throw (for example, `ExternalContract.doSomething()` will also `throw` if `doSomething()` throws).

If you choose to use the low-level call methods, make sure to handle the possibility that the call will fail, by checking the return value.

```sol
// bad
someAddress.send(55);
someAddress.call.value(55)(); // this is doubly dangerous, as it will forward all remaining gas and doesn't check for result
someAddress.call.value(100)(bytes4(sha3("deposit()"))); // if deposit throws an exception, the raw call() will only return false and transaction will NOT be reverted

// good
if(!someAddress.send(55)) {
    // Some failure code
}

ExternalContract(someAddress).deposit.value(100);
```


### Favor *pull* over *push* for external calls

External calls can fail accidentally or deliberately. To minimize the damage caused by such failures, it is often better to isolate each external call into its own transaction that can be initiated by the recipient of the call. This is especially relevant for payments, where it is better to let users withdraw funds rather than push funds to them automatically. (This also reduces the chance of [problems with the gas limit](./known_attacks#dos-with-block-gas-limit).)  Avoid combining multiple `send()` calls in a single transaction.

```sol
// bad
contract auction {
    address highestBidder;
    uint highestBid;

    function bid() payable {
        require(msg.value >= highestBid);

        if (highestBidder != 0) {
            highestBidder.transfer(highestBid); // if this call consistently fails, no one else can bid
        }

       highestBidder = msg.sender;
       highestBid = msg.value;
    }
}

// good
contract auction {
    address highestBidder;
    uint highestBid;
    mapping(address => uint) refunds;

    function bid() payable external {
        require(msg.value >= highestBid);

        if (highestBidder != 0) {
            refunds[highestBidder] += highestBid; // record the refund that this user can claim
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }

    function withdrawRefund() external {
        uint refund = refunds[msg.sender];
        refunds[msg.sender] = 0;
        msg.sender.transfer(refund);
    }
}
```

## Don't assume contracts are created with zero balance

An attacker can send wei to the address of a contract before it is created.  Contracts should not assume that its initial state contains a zero balance.  See [issue 61](https://github.com/ConsenSys/smart-contract-best-practices/issues/61) for more details.

## Remember that on-chain data is public

Many applications require submitted data to be private up until some point in time in order to work. Games (eg. on-chain rock-paper-scissors) and auction mechanisms (eg. sealed-bid second-price auctions) are two major categories of examples. If you are building an application where privacy is an issue, take care to avoid requiring users to publish information too early.

Examples:

* In rock paper scissors, require both players to submit a hash of their intended move first, then require both players to submit their move; if the submitted move does not match the hash throw it out.
* In an auction, require players to submit a hash of their bid value in an initial phase (along with a deposit greater than their bid value), and then submit their action bid value in the second phase.
* When developing an application that depends on a random number generator, the order should always be (1) players submit moves, (2) random number generated, (3) players paid out. The method by which random numbers are generated is itself an area of active research; current best-in-class solutions include Bitcoin block headers (verified through http://btcrelay.org), hash-commit-reveal schemes (ie. one party generates a number, publishes its hash to "commit" to the value, and then reveals the value later) and [RANDAO](http://github.com/randao/randao).
* If you are implementing a frequent batch auction, a hash-commit scheme is also desirable.

## In 2-party or N-party contracts, beware of the possibility that some participants may "drop offline" and not return

Do not make refund or claim processes dependent on a specific party performing a particular action with no other way of getting the funds out. For example, in a rock-paper-scissors game, one common mistake is to not make a payout until both players submit their moves; however, a malicious player can "grief" the other by simply never submitting their move - in fact, if a player sees the other player's revealed move and determines that they lost, they have no reason to submit their own move at all. This issue may also arise in the context of state channel settlement. When such situations are an issue, (1) provide a way of circumventing non-participating participants, perhaps through a time limit, and (2) consider adding an additional economic incentive for participants to submit information in all of the situations in which they are supposed to do so.

## Solidity specific recommendations

### <!-- -->

The following recommendations are specific to Solidity, but may also be instructive for developing smart contracts in other languages.

## Enforce invariants with `assert()`

An assert guard triggers when an assertion fails - such as an invariant property changing. For example, the token to ether issuance ratio, in a token issuance contract, may be fixed. You can verify that this is the case at all times with an `assert()`. Assert guards should often be combined with other techniques, such as pausing the contract and allowing upgrades. (Otherwise, you may end up stuck, with an assertion that is always failing.)

Example:

```sol
contract Token {
    mapping(address => uint) public balanceOf;
    uint public totalSupply;

    function deposit() public payable {
        balanceOf[msg.sender] += msg.value;
        totalSupply += msg.value;
        assert(this.balance >= totalSupply);
    }
}
```

Note that the assertion is *not* a strict equality of the balance because the contract can be [forcibly sent ether](#remember-that-ether-can-be-forcibly-sent-to-an-account) without going through the `deposit()` function!


## Use `assert()` and `require()` properly

In Solidity 0.4.10 `assert()` and `require()` were introduced. `require(condition)` is meant to be used for input validation, which should be done on any user input, and reverts if the condition is false. `assert(condition)` also reverts if the condition is false but should be used only for invariants: internal errors or to check if your contract has reached an invalid state. Following this paradigm allows formal analysis tools to verify that the invalid opcode can never be reached: meaning no invariants in the code are violated and that the code is formally verified.

## Beware rounding with integer division

All integer division rounds down to the nearest integer. If you need more precision, consider using a multiplier, or store both the numerator and denominator.

(In the future, Solidity will have a fixed-point type, which will make this easier.)

```sol
// bad
uint x = 5 / 2; // Result is 2, all integer divison rounds DOWN to the nearest integer
```

Using a multiplier prevents rounding down, this multiplier needs to be accounted for when working with x in the future:

```sol
// good
uint multiplier = 10;
uint x = (5 * multiplier) / 2;
```

Storing the numerator and denominator means you can calculate the result of `numerator/denominator` off-chain:
```sol
// good
uint numerator = 5;
uint denominator = 2;
```

## Remember that Ether can be forcibly sent to an account

Beware of coding an invariant that strictly checks the balance of a contract.

An attacker can forcibly send wei to any account and this cannot be prevented (not even with a fallback function that does a `revert()`).

The attacker can do this by creating a contract, funding it with 1 wei, and invoking
`selfdestruct(victimAddress)`.  No code is invoked in `victimAddress`, so it
cannot be prevented.

## Be aware of the tradeoffs between abstract contracts and interfaces

Both interfaces and abstract contracts provide one with a customizable and re-usable approach for smart contracts. Interfaces, which were introduced in Solidity 0.4.11, are similar to abstract contracts but cannot have any functions implemented. Interfaces also have limitations such as not being able to access storage or inherit from other interfaces which generally makes abstract contracts more practical. Although, interfaces are certainly useful for designing contracts prior to implementation. Additionally, it is important to keep in mind that if a contract inherits from an abstract contract it must implement all non-implemented functions via overriding or it will be abstract as well.

## Keep fallback functions simple

[Fallback functions](http://solidity.readthedocs.io/en/latest/contracts.html#fallback-function) are called when a contract is sent a message with no arguments (or when no function matches), and only has access to 2,300 gas when called from a `.send()` or `.transfer()`. If you wish to be able to receive Ether from a `.send()` or `.transfer()`, the most you can do in a fallback function is log an event. Use a proper function if a computation or more gas is required.

```sol
// bad
function() payable { balances[msg.sender] += msg.value; }

// good
function deposit() payable external { balances[msg.sender] += msg.value; }

function() payable { LogDepositReceived(msg.sender); }
```

## Explicitly mark visibility in functions and state variables

Explicitly label the visibility of functions and state variables. Functions can be specified as being `external`, `public`, `internal` or `private`. Please understand the differences between them, for example, `external` may be sufficient instead of `public`. For state variables, `external` is not possible. Labeling the visibility explicitly will make it easier to catch incorrect assumptions about who can call the function or access the variable.

```sol
// bad
uint x; // the default is internal for state variables, but it should be made explicit
function buy() { // the default is public
    // public code
}

// good
uint private y;
function buy() external {
    // only callable externally
}

function utility() public {
    // callable externally, as well as internally: changing this code requires thinking about both cases.
}

function internalAction() internal {
    // internal code
}
```

## Lock pragmas to specific compiler version

Contracts should be deployed with the same compiler version and flags that they have been tested the most with. Locking the pragma helps ensure that contracts do not accidentally get deployed using, for example, the latest compiler which may have higher risks of undiscovered bugs. Contracts may also be deployed by others and the pragma indicates the compiler version intended by the original authors.

```sol
// bad
pragma solidity ^0.4.4;


// good
pragma solidity 0.4.4;
```

### Exception

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

## Differentiate functions and events

Favor capitalization and a prefix in front of events (we suggest *Log*), to prevent the risk of confusion between functions and events. For functions, always start with a lowercase letter, except for the constructor.

```sol
// bad
event Transfer() {}
function transfer() {}

// good
event LogTransfer() {}
function transfer() external {}
```

## Prefer newer Solidity constructs

Prefer constructs/aliases such as `selfdestruct` (over `suicide`) and `keccak256` (over `sha3`).  Patterns like `require(msg.sender.send(1 ether))` can also be simplified to using `transfer()`, as in `msg.sender.transfer(1 ether)`.

## Be aware that 'Built-ins' can be shadowed

It is currently possible to [shadow](https://en.wikipedia.org/wiki/Variable_shadowing) built-in globals in Solidity. This allows contracts to override the functionality of built-ins such as `msg` and `revert()`. Although this [is intended](https://github.com/ethereum/solidity/issues/1249), it can mislead users of a contract as to the contract's true behavior.

```sol
contract PretendingToRevert {
    function revert() internal constant {}
}

contract ExampleContract is PretendingToRevert {
    function somethingBad() public {
        revert();
    }
}
```

Contract users (and auditors) should be aware of the full smart contract source code of any application they intend to use.

## Avoid using `tx.origin`

Never use `tx.origin` for authorization, another contract can have a method which will call your contract (where the user has some funds for instance) and your contract will authorize that transaction as your address is in `tx.origin`.

```
pragma solidity 0.4.18;

contract MyContract {

    address owner;

    function MyContract() public {
        owner = msg.sender;
    }

    function sendTo(address receiver, uint amount) public {
        require(tx.origin == owner);
        receiver.transfer(amount);
    }

}

contract AttackingContract {

    MyContract myContract;
    address attacker;

    function AttackingContract(address myContractAddress) public {
        myContract = MyContract(myContractAddress);
        attacker = msg.sender;
    }

    function() public {
        myContract.sendTo(attacker, msg.sender.balance);
    }

}
```

You should use `msg.sender` for authorization (if another contract calls your contract `msg.sender` will be the address of the contract and not the address of the user who called the contract).

You can read more about it here: [Solidity docs](https://solidity.readthedocs.io/en/develop/security-considerations.html#tx-origin)

Besides the issue with authorization, there is a chance that `tx.origin` will be removed from the Ethereum protocol in the future, so code that uses `tx.origin` won't be compatible with future releases [Vitalik: 'Do NOT assume that tx.origin will continue to be usable or meaningful.'](https://ethereum.stackexchange.com/questions/196/how-do-i-make-my-dapp-serenity-proof/200#200)

It's also worth mentioning that by using `tx.origin` you're limiting interoperability between contracts because the contract that uses tx.origin cannot be used by another contract as a contract can't be the `tx.origin`.

## Timestamp Dependence

There are three main considerations when using a timestamp to execute a critical function in a contract, especially when actions involve fund transfer.

### Gameability

Be aware that the timestamp of the block can be manipulated by a miner. Consider this [contract](https://etherscan.io/address/0xcac337492149bdb66b088bf5914bedfbf78ccc18#code):

```sol

uint256 constant private salt =  block.timestamp;

function random(uint Max) constant private returns (uint256 result){
    //get the best seed for randomness
    uint256 x = salt * 100/Max;
    uint256 y = salt * block.number/(salt % 5) ;
    uint256 seed = block.number/3 + (salt % 300) + Last_Payout + y;
    uint256 h = uint256(block.blockhash(seed));

    return uint256((h / x)) % Max + 1; //random number between 1 and Max
}
```

When the contract uses the timestamp to seed a random number, the miner can actually post a timestamp within 30 seconds of the block being validating, effectively allowing the miner to precompute an option more favorable to their chances in the lottery. Timestamps are not random and should not be used in that context.

### *30-second Rule*
A general rule of thumb in evaluating timestamp usage is:
#### If the contract function can tolerate a [30-second](https://ethereum.stackexchange.com/questions/5924/how-do-ethereum-mining-nodes-maintain-a-time-consistent-with-the-network/5931#5931) drift in time, it is safe to use `block.timestamp`
If the scale of your time-dependent event can vary by 30-seconds and maintain integrity, it is safe to use a timestamp. This includes things like ending of auctions, registration periods, etc.

### Caution using `block.number` as a timestamp

When a contract creates an `auction_complete` modifier to signify the end of a token sale such as [so]((https://github.com/SpankChain/old-sc_auction/blob/master/contracts/Auction.sol))
```sol
modifier auction_complete {
    require(auctionEndBlock <= block.number     ||
          currentAuctionState == AuctionState.success ||
          currentAuctionState == AuctionState.cancel)
        _;}
```
`block.number` and *[average block time](https://etherscan.io/chart/blocktime)* can be used to estimate time as well, but this is not future proof as block times may change (such as [fork reorganisations](https://blog.ethereum.org/2015/08/08/chain-reorganisation-depth-expectations/) and the [difficulty bomb](https://github.com/ethereum/EIPs/issues/649)). In a sale spanning days, the 12-minute rule allows one to construct a more reliable estimate of time.

## Multiple Inheritance Caution

When utilizing multiple inheritance in Solidity, it is important to understand how the compiler composes the inheritance graph.

```sol

contract Final {
    uint public a;
    function Final(uint f) public {
        a = f;
    }
}

contract B is Final {
    int public fee;

    function B(uint f) Final(f) public {
    }
    function setFee() public {
        fee = 3;
    }
}

contract C is Final {
    int public fee;

    function C(uint f) Final(f) public {
    }
    function setFee() public {
        fee = 5;
    }
}

contract A is B, C {
  function A() public B(3) C(5) {
      setFee();
  }
}
```
When A is deployed, the compiler will *linearize* the inheritance from left to right, as:

**C -> B -> A**

The consequence of the linearization will yield a `fee` value of 5, since C is the most derived contract. This may seem obvious, but imagine scenarios where C is able to shadow crucial functions, reorder boolean clauses, and cause the developer to write exploitable contracts. Static analysis currently does not raise issue with overshadowed functions, so it must be manually inspected.

For more on security and inheritance, check out this [article](https://pdaian.com/blog/solidity-anti-patterns-fun-with-inheritance-dag-abuse/)

To help contribute, Solidity's Github has a [project](https://github.com/ethereum/solidity/projects/9#card-8027020) with all inheritance-related issues.

## Deprecated/historical recommendations

These are recommendations which are no longer relevant due to changes in the protocol or improvements to solidity. They are recorded here for posterity and awareness.

### Beware division by zero (Solidity < 0.4)

Prior to version 0.4, Solidity [returns zero](https://github.com/ethereum/solidity/issues/670) and does not `throw` an exception when a number is divided by zero. Ensure you're running at least version 0.4.
