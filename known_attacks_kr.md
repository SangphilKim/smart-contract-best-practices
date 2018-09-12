다음은 당신이 스마트 컨트랙트를 작성할 때 인지하고 방지해야 하는 known attacks의 목록입니다.

## 경합조건(Race Conditions)<sup><a href='#footnote-race-condition-terminology'>\*</a></sup>

컨트랙트를 외부(external)에서 호출 시 발생하는 가장 큰 위험 중 하나는 그것이 통제 흐름을 뺏을 수 있다는 것이며, 호출한 함수가 예상하지 못한 당신의 데이터를 변경할 수 있다는 것입니다. 이러한 종류의 버그는 많은 형태를 가지고 있을 수 있으며, DAO의 붕괴를 만들어낸 두 개의 주요 버그들은 이런 종류의 버그였습니다.

### 재진입(Reentrancy)

이미 알려진 이 버그의 첫 번째 버전은 함수의 첫 번째 호출이 종료되기 전에 재귀 호출되는 함수를 포함하고 있습니다. 이것은 파괴적인 방향으로 상호작용하는 다른 함수의 호출을 야기할 수 있습니다.

```sol
// 안전하지 않음
mapping (address => uint) private userBalances;

function withdrawBalance() public {
    uint amountToWithdraw = userBalances[msg.sender];
    require(msg.sender.call.value(amountToWithdraw)()); // 이 부분에서 호출자의 코드가 실행되며, withdrawBalance를 다시 호출할 수 있습니다
    userBalances[msg.sender] = 0;
}
```

사용자의 잔액은 함수의 마지막까지 0이 되지 않기 때문에, 두 번째(그리고 이후) 호출은 계속해서 성공할 것이며, 잔액을 다시 반복해서 인출(withdraw)하게 됩니다. 매우 유사한 버그는 DAO 공격의 취약성들 중 하나였습니다.

예시에서 주어진 것처럼, 이 문제를 피하기 위한 가장 좋은 방법은 [`call.value()()` 대신 `send()`를 사용하는 것]입니다. 이렇게 하면 외부 코드가 호출되는 것을 막을 수 있습니다.

그러나 만약 당신이 외부 호출을 제거하지 못한다면, 이 공격을 막기위한 가장 간단한 방법은 당신이 해야하는 모든 내부(internal) 작업을 마치기 전까지 외부 함수를 호출하지 않는 것입니다:

```sol
mapping (address => uint) private userBalances;

function withdrawBalance() public {
    uint amountToWithdraw = userBalances[msg.sender];
    userBalances[msg.sender] = 0;
    require(msg.sender.call.value(amountToWithdraw)()); // 사용자의 잔액은 이미 0이기 때문에 이후의 호출들은 아무것도 인출하지 못합니다.
}
```

만약 당신이 `withdrawBalance()`를 호출하는 다른 함수를 가지고 있다면, 같은 공격의 잠재적인 목표가 될 수 있음에 유의하세요. 그러므로 당신은 신뢰할 수 없는 컨트랙트를 호출하는 함수들 그 자체를 신뢰할 수 없는 것으로 취급해야 합니다. 가능성 있는 해결책(solution)들에 대한 자세한 내용은 아래를 참조하십시오.

### 함수간 경합 조건(Cross-function Race Conditions)

또한 공격자는 같은 상태(state)를 공유하는 두 가지 다른 함수들을 사용해 유사한 공격을 할 수 있습니다.

```sol
// 안전하지 않음
mapping (address => uint) private userBalances;

function transfer(address to, uint amount) {
    if (userBalances[msg.sender] >= amount) {
       userBalances[to] += amount;
       userBalances[msg.sender] -= amount;
    }
}

function withdrawBalance() public {
    uint amountToWithdraw = userBalances[msg.sender];
    require(msg.sender.call.value(amountToWithdraw)()); // 이 부분에서 호출자의 코드가 실행되며, transfer()를 호출할 수 있습니다.
    userBalances[msg.sender] = 0;
}
```

이 경우, 공격자는 코드가 `withdrawBalance` 내에서 외부 호출로 실행되었다면 `transfer()`를 호출합니다. 공격자들의 잔액은 0이 되지 않기 때문에, 그들은 이미 인출을 받았음에도 불구하고 토큰을 전송할 수 있습니다. 이 취약성 또한 DAO 공격에 사용되었습니다.

같은 주의 사항의 같은 해결책으로 해결이 가능합니다. 또한 이 예시에서는 두 함수 모두 같은 컨트랙트의 일부임을 유의하세요. 그러나 여러 컨트랙트들이 상태를 공유한다면 같은 버그가 컨트랙트들 간에 발생할 수 있습니다.

### 경합 조건 해결책의 위험요소(Pitfalls in Race Condition Solutions)

경합 조건은 여러 함수들, 심지어는 여러 컨트랙트들 간에 발생할 수 있기 때문에 재진입을 막는 것을 목표로 하는 해결책들은 충분하지 않습니다.

대신 우리는 모든 내부 작업을 우선으로 마친 후, 외부 함수를 호출하는 것을 추천합니다. 만약 당신이 이 규칙을 잘 지킨다면, 당신은 경합 조건을 피할 수 있습니다. 그러나 당신은 외부 함수를 너무 빨리 호출하는 것을 피하는 것 뿐만 아니라 외부 함수를 호출하는 함수를 호출하는 것도 피해야 합니다. 다음 예시는 안전하지 못한 경우입니다:

```sol
// 안전하지 않음
mapping (address => uint) private userBalances;
mapping (address => bool) private claimedBonus;
mapping (address => uint) private rewardsForA;

function withdraw(address recipient) public {
    uint amountToWithdraw = userBalances[recipient];
    rewardsForA[recipient] = 0;
    require(recipient.call.value(amountToWithdraw)());
}

function getFirstWithdrawalBonus(address recipient) public {
    require(!claimedBonus[recipient]); // 받는 사람들은 각자 보너스를 한번만 요구할 수 있습니다.

    rewardsForA[recipient] += 100;
    withdraw(recipient); // 이 부분에서 호출자는 getFirstWithdrawalBonus를 다시 실행할 수 있습니다.
    claimedBonus[recipient] = true;
}
```

`getFirstWithdrawalBonus()`가 외부 컨트랙트를 직접 호출하지 않음에도 불구하고 `withdraw()`내의 호출은 경합 조건을 취약하게 만들기에 충분합니다. 그러므로 당신은 `withdraw()` 또한 신뢰할 수 없는 것처럼 취급해야 합니다.

```sol
mapping (address => uint) private userBalances;
mapping (address => bool) private claimedBonus;
mapping (address => uint) private rewardsForA;

function untrustedWithdraw(address recipient) public {
    uint amountToWithdraw = userBalances[recipient];
    rewardsForA[recipient] = 0;
    require(recipient.call.value(amountToWithdraw)());
}

function untrustedGetFirstWithdrawalBonus(address recipient) public {
    require(!claimedBonus[recipient]); // 받는 사람들은 각자 보너스를 한번만 요구할 수 있습니다.

    claimedBonus[recipient] = true;
    rewardsForA[recipient] += 100;
    untrustedWithdraw(recipient); // claimedBonus를 true로 설정합니다. 그래서 재진입(reentry)이 불가능합니다.
}
```

재진입을 불가능하도록 수정하는 것 뿐만 아니라, [신뢰할 수 없는 함수들은 표시가 되었습니다](./recommendations#mark-untrusted-contracts). 이것과 같은 양식은 모든 수준에서 반복됩니다: `untrustedGetFirstWithdrawalBonus()` 함수가 외부 컨트랙트를 호출하는 `untrustedWithdraw()`를 호출하기 때문에, 당신은 또한 반드시 `untrustedGetFirstWithdrawalBonus()`를 안전하지 않은 것으로 대해야 합니다.

종종 추천되는 또 다른 해결책은 [뮤텍스(mutex)](https://en.wikipedia.org/wiki/Mutual_exclusion)입니다. 이것은 당신이 몇몇 상태를 "잠금"할 수 있도록 함으로써 "잠금"의 소유자에 의해서만 변경하는 것이 가능하게 합니다. 쉬운 예시는 다음과 같습니다.

```sol
// 주석(Note): 이것은 가장 기본적인 예시이며 뮤텍스는 큰 규모의(substantial) 로직 그리고/또는 공유된 상태에서 특히 유용합니다.
mapping (address => uint) private balances;
bool private lockBalances;

function deposit() payable public returns (bool) {
    require(!lockBalances);
    lockBalances = true;
    balances[msg.sender] += msg.value;
    lockBalances = false;
    return true;
}

function withdraw(uint amount) payable public returns (bool) {
    require(!lockBalances && amount > 0 && balances[msg.sender] >= amount);
    lockBalances = true;

    if (msg.sender.call(amount)()) { // 일반적으로는 안전하지 않지만, 뮤텍스가 이것을 안전하게 합니다.
      balances[msg.sender] -= amount;
    }

    lockBalances = false;
    return true;
}
```

만약 사용자가 첫 번째 호출이 끝나기 전에 `withdraw()`를 다시 호출하려고 한다면, "잠금"은 이것이 영향을 미치기 전에 막을 것입니다. 이것은 효율적인 패턴이 될 수 있지만, 당신이 함께 동작해야 하는 여러 컨트랙트들을 가지고 있다면 난해하게 됩니다. 다음 예시는 안전하지 못합니다:

```sol
// 안전하지 않음
contract StateHolder {
    uint private n;
    address private lockHolder;

    function getLock() {
        require(lockHolder == 0);
        lockHolder = msg.sender;
    }

    function releaseLock() {
        require(msg.sender == lockHolder);
        lockHolder = 0;
    }

    function set(uint newState) {
        require(msg.sender == lockHolder);
        n = newState;
    }
}
```

공격자는 `getLock()` 함수를 호출할 수 있지만, 그런 다음 `releaseLock()`을 호출할 수 없습니다. 만약 그들이 그렇게 한다면, 컨트랙트는 영원히 잠기게 됩니다. 그리고 추가 변경은 일어나지 않을 것입니다. 만약 당신이 경합 조건으로부터 보호하기 위해 뮤텍스를 사용한다면, 당신은 "잠금"을 요청할 수 있는 방법이 없다는 것과 다시는 풀 수 없다는 것을 주의해서 보장해야 합니다.

<a name='footnote-race-condition-terminology'></a>

<div style='font-size: 80%; display: inline;'>* 몇몇 사람들은 이더리움이 현재에는 진정한 병행성을 가지고 있지 않기 때문에 <i>경합 조건</i>이라는 단어의 사용에 반대할 수도 있습니다. 하지만 자원(resources)을 차지하기 위한 논리적으로 서로 다른 프로세스(processes)의 기본적인 특징들이 여전히 존재하며 같은 종류의 위험들과 잠재적인 해결책이 적용되어 있습니다.</div>

## 트랜잭션-순서 의존성(TOD : Transaction-Ordering Dependence) / 선행매매(Front Running)

위의 예시들은 공격자가 악의적인 코드를 *단일 트랜잭션 내*에서 실행하는 것을 포함하고 있는 경합 조건들의 예시입니다. 다음은 블록체인에 내재되어 있는 다른 종류의 경합 조건입니다: 실제로 *트랜잭션들의 순서*(블록 내의)는 쉽게 조작될 수 있습니다.

트랜잭션은 멤풀(Mempool)내에 잠시 머무르게 되기 때문에, 누군가는 트랜잭션이 블록에 포함되기 전에 어떤 동작이 일어나게 될지 알 수 있습니다. 이것은 몇 개의 토큰을 구매하기 위한 트랜잭션이 보일 수 있는 탈중앙화되된 시장(market)이나, 다른 트랜잭션이 포함되기 전에 시장 주문이 먼저 수행되는 것과 같은 문제를 만들어 낼 수 있습니다. 이것을 방지하는것은 특정 컨트랙트 그 자체에 이르기 때문에 어렵습니다. 예를 들면 시장에서는 집단 경매(batch auction)을 구현하는 것이 더 낫습니다(또한 이것은 잦은 빈도의 구매에 대한 걱정을 하지 않아도 됩니다). 또 다른 방법은 선-커밋(pre-commit) 구조를 사용하는 것입니다("자세한 내용은 나중에 제출하겠습니다").

## 타임스탬프 의존성(Timestamp Dependence)
블록의 타임스탬프는 채굴자에 의해 조작될 수 있음을 알고 있어야 합니다. 그리고 모든 직, 간접적인 타임스탬프의 사용은 고려해야 합니다.

타임스탬프 의존성에 관련된 디자인 고려사항들을 위한 [권장 사항](./recommendations/#timestamp-dependence) 부분을 참조하세요.

## 정수 오버플로우와 언더플로우

간단한 토큰 전송을 살펴보겠습니다:

```sol
mapping (address => uint256) public balanceOf;

// 안전하지 않음
function transfer(address _to, uint256 _value) {
    /* 보내는 사람이 충분한 잔액을 가지고 있는지를 확인합니다. */
    require(balanceOf[msg.sender] >= _value);
    /* 새로운 잔액을 더하고 뺍니다. */
    balanceOf[msg.sender] -= _value;
    balanceOf[_to] += _value;
}

// 안전함
function transfer(address _to, uint256 _value) {
    /* 보내는 사람의 잔액이 충분한지와 오버플로우를 확인합니다. */
    require(balanceOf[msg.sender] >= _value && balanceOf[_to] + _value >= balanceOf[_to]);

    /* 새로운 잔액을 더하고 뺍니다. */
    balanceOf[msg.sender] -= _value;
    balanceOf[_to] += _value;
}
```

만약 잔액이 최대 uint 값(2^256)에 다다르면, 이것은 0으로 돌아가게 됩니다. 이것은 이런 조건을 확인합니다. 이것이 관련이 있든, 없든 구현에 따르게 됩니다. 어떻게 됐든 uint 값이 매우 큰 숫자에 근접할 경우를 생각해봅시다. 어떻게 uint 변수가 상태를 변경시키고, 그리고 누가 이와 같은 변경을 할 수 있는 권한을 가지고 있는지 생각해봅시다. 만약 아무나 uint 값을 변경하는 함수를 호출할 수 있다면, 이것은 공격에 더욱 취약해집니다. 만약 관리자만이 변수의 상태를 변경할 수 있도록 접근할 수 있다면, 당신은 안전할 것입니다. 또한 만약 사용자가 한번에 1 밖에 증가시킬 수 없다면, 이런 한계값에 근접할 수 있는 실현 가능한 방법이 없기 때문에 아마 당신은 안전할 것입니다.

언더플로우도 마찬가지입니다. 만약 uint가 0보다 작아진다면, 이것은 언더플로우를 발생시킬 것이고 uint의 최대값으로 설정될 것입니다.

uint8, uint16, uint24 등과 같은 더 작은 데이터의 종류에 주의해야합니다: 이 데이터 종류들은 심지어는 더 쉽게 최대값을 만들어 낼 수 있습니다.

[20가지 이상의 오버플로우와 언더플로우](https://github.com/ethereum/solidity/issues/796#issuecomment-253578925)가 있다는 것을 알고 있어야 합니다.

### 언더플로우 심화: 저장소 조작(Storage Manipulation)
 2017 underhanded solidity contest에서 [Doug Hoyte의 출품작](https://github.com/Arachnid/uscc/tree/master/submissions-2017/doughoyte)은 선외 가작에 들었습니다. 출품작은 매우 흥미롭습니다. 왜냐하면 이것은 C언어 스타일의 언더플로우가 솔리디티 저장소에 어떻게 영향을 미칠지에 대한 문제를 제기했기 때문입니다. 이것은 위 출품작의 간단한 버전입니다:

 ```sol
contract UnderflowManipulation {
    address public owner;
    uint256 public manipulateMe = 10;
    function UnderflowManipulation() {
        owner = msg.sender;
    }
    
    uint[] public bonusCodes;
    
    function pushBonusCode(uint code) {
        bonusCodes.push(code);
    }
    
    function popBonusCode()  {
        require(bonusCodes.length >=0);  // 항상 진실인 명제(Tautology)입니다.
        bonusCodes.length--; // 이 부분에서 언더플로우가 발생할 수 있습니다.
    }
    
    function modifyBonusCode(uint index, uint update)  { 
        require(index < bonusCodes.length);
        bonusCodes[index] = update; // bonusCodes.length보다 작은 인덱스에 입력합니다.
    }
    
}
```

일반적으로, 변수 `manipulateMe`의 위치는 `keccak256`을 거치지 않는다면 영향을 줄 수 없습니다. 즉, 실행이 불가능합니다. 하지만 동적 배열은 순차적으로 저장하기 때문에, 만약 악의적인 이용자가 `manipulateMe`를 변경하고자 한다면 그들은 해야하는 행동은 다음과 같습니다:

* 언더플로우를 위해 `popBonusCode`를 호출합니다(주의 사항: 솔리디티는 [내장된 pop 메서드가 없습니다](https://github.com/ethereum/solidity/pull/3743)).
* `manipulateMe`의 저장소 위치를 계산합니다.
* `modifyBonusCode`를 이용해 `manipulateMe`의 값을 조작하거나 갱신합니다.

실제로 이 배열은 즉시 수상하다고 지적되겠지만, 더 복잡한 스마트 컨트랙트 구조에 뒤덮여 있어 임의로 상수 변수에 대해 악의적인 변경을 허용할 수 있습니다.

동적 배열 사용을 고려중이라면 컨테이너 데이터 구조는 종은 예제입니다. 솔리디티 CRUD [파트 1](https://medium.com/@robhitchens/solidity-crud-part-1-824ffa69509a)와 [파트 2](https://medium.com/@robhitchens/solidity-crud-part-2-ed8d8b4f74ec) 기사는 좋은 참고 자료입니다.

<a name="dos-with-unexpected-revert"></a>

## (예상 밖의) 회귀(revert)가 있는 DoS(DoS with Unexpected revert)

간단한 경매(auction) 컨트랙트를 살펴보겠습니다:

```sol
// 안전하지 않음
contract Auction {
    address currentLeader;
    uint highestBid;

    function bid() payable {
        require(msg.value > highestBid);

        require(currentLeader.send(highestBid)); // 이전의 리더에게 환불을 해줍니다. 그리고 만약 환불에 실패하면 회귀됩니다.

        currentLeader = msg.sender;
        highestBid = msg.value;
    }
}
```

이전의 리더에게 환불해주려 할 때, 환불이 실패한다면 회귀하게 됩니다. 이것은 그들의 주소로 환불하려는 것을 *항상* 실패하게 함으로써 악의적인 가격 제시자가 리더가 될 수 있다는 것을 의미합니다. 이런 경우 그들은 `bid()` 함수를 호출하는 다른 사람들을 막을 수 있으며, 계속 리더로 남아있을 수 있습니다. 추천하는 방법은 이전에 설명했던 것처럼 [pull payment system](./recommendations#favor-pull-over-push-for-external-calls)을 설정하는 것입니다.

또 다른 예시는 컨트랙트가 배열을 통해 사용자들에게 결제하는 것을 반복하는 것입니다(예를 들자면 크라우드펀딩 컨트랙트의 서포터들). 이것은 각각의 결제가 성공하는 것을 확실히 하는 것을 원한다면 일반적으로 사용되는 방법입니다. 만약 그렇지 않다면 그 하나는 회귀됩니다. 문제는 하나가 실패한다면 당신은 전체 지불 시스템을 회귀시켜야 한다는 것입니다. 이것은 반복이 절대 끝나지 않는다는 것을 의미합니다. 한 개의 주소가 에러를 만들어내기 때문에 그 누구도 지불을 받지 못합니다.

```sol
address[] private refundAddresses;
mapping (address => uint) public refunds;

// 좋지 않은 코드
function refundAll() public {
    for(uint x; x < refundAddresses.length; x++) { // 얼마나 많은 주소들이 참가했는지에 기반한 임의의 길이만큼 반복
        require(refundAddresses[x].send(refunds[refundAddresses[x]])) // 두 배로 나쁨, 지금 send에서 발생한 단 하나의 실패는 모든 자금을 지연시키킵니다.
    }
}
```

다시 말하지만 추천하는 해결책으로는 [pull over push payments를 장려](./recommendations#favor-pull-over-push-for-external-calls)합니다.

## 블록 가스 한도가 있는 DoS(DoS with Block Gas Limit)

아마 당신은 이전의 예제에서 또 다른 문제가 있다는 것을 눈치챘을 것입니다: 모두에게 한번에 지불한다면, 당신은 블록 가스 한도에 도달할 위험이 있습니다. 각각의 이더리움 블록은 특정 최대 계산량을 처리할 수 있습니다. 만약 당신이 그 이상을 시도한다면, 당신의 트랜잭션은 실패할 것입니다.

이것은 의도적인 공격이 없어도 문제를 일으킬 수 있습니다. 그러나 공격자가 필요한 가스량을 조작할 수 있다면 특히 나쁩니다. 이전의 예제에서 공격자는 각각 소량의 환불만을 요구하는 많은 주소들을 추가할 수 있습니다. 그러므로 공격자의 각 주소들에 환불하는데 드는 가스 비용은 가스 한도 이상일 수 있으며 환불 트랜잭션이 전혀 발생하지 않습니다.

이것은 [pull over push payments를 장려](./recommendations#favor-pull-over-push-for-external-calls)하는 또 다른 이유입니다.

만약 당신이 반드시 꼭 알 수 없는 크기의 배열을 반복해야 한다면, 잠재적으로 여러 블록을 사용할 수 있도록 계획해야 합니다. 즉, 여러개의 트랜잭션이 필요합니다. 다음 예시와 같이 당신은 얼마나 진행했는지 추적할 수 있어야 하며, 그 지점으로부터 다시 시작할 수 있어야 합니다:

```sol
struct Payee {
    address addr;
    uint256 value;
}

Payee[] payees;
uint256 nextPayeeIndex;

function payOut() {
    uint256 i = nextPayeeIndex;
    while (i < payees.length && msg.gas > 200000) {
      payees[i].addr.send(payees[i].value);
      i++;
    }
    nextPayeeIndex = i;
}
```

`payOut()` 함수의 다음 반복을 기다리는 동안 다른 트랜잭션이 처리되더라도 별다른 일이 일어나지 않도록 해야합니다. 그러니 이 패턴은 반드시 필요한 경우에만 사용하세요.

## 강제로 컨트랙트에 이더를 전송(Forcibly Sending Ether to a Contract)

컨트랙트의 폴백 함수(fallback function)를 트리거링하지 않으면서 강제로 컨트랙트에 이더를 전송하는 것이 가능합니다. 이것은 중요한 로직을 폴백 함수에 위치시키거나 컨트랙트의 잔액을 기반으로 하는 계산을 만들 때 중요하게 고려해야합니다. 다음 예시를 보겠습니다:

```sol
contract Vulnerable {
    function () payable {
        revert();
    }
    
    function somethingBad() {
        require(this.balance > 0);
        // 무언가 악의적인 행동을 함
    }
}
```

컨트랙트 로직은 컨트랙트로의 결제를 허용하지 않기 때문에 "무언가 악의적인 행동"이 일어나는 것을 허용하지 않는 것처럼 보입니다. 그러나 컨트랙트에 이더를 강제로 전송하고 그 잔액을 0보다 크게 만드는 몇 가지 방법이 있습니다.

`selfdestruct` 컨트랙트 메서드는 사용자가 초과하는 이더를 받을 수령인을 명시하는 것을 가능하게 합니다.  `selfdestruct`는 [컨트랙트의 폴백 함수를 작동시키지 않습니다](https://solidity.readthedocs.io/en/develop/security-considerations.html#sending-and-receiving-ether).

또한 컨트랙트를 배포하기 전에 컨트랙트의 주소와 이더를 그 주소로 보내는 것을 [미리 계산하는 것](https://github.com/Arachnid/uscc/tree/master/submissions-2017/ricmoo)도 가능합니다.

컨트랙트 개발자들은 이더가 컨트랙트에 강제로 전송될 수 있다는 것을 반드시 알고 있어야 하며 컨트랙트 로직을 상황에 맞게 설계해야 합니다. 일반적으로는 당신의 컨트랙트에 돈을 보내는 출처를 차단하는것이 불가능하다는 것을 가정합니다.

## 지금은 불가능한, 역사적으로 발생했던 공격들(Deprecated/historical attacks)

이와 같은 공격들은 프로토콜의 변경들과 솔리디티의 개선으로 인해 더 이상은 불가능합니다. 후배 개발자들과 이에 대한 관심을 위해 이곳에 기록되었습니다.

### 호출 깊이 공격(Call Depth Attack, 현재에는 불가능함)

[EIP 150](https://github.com/ethereum/EIPs/issues/150)의 하드포크로 Call Depth 공격은 더 이상 관련이 없습니다<sup><a href='http://ethereum.stackexchange.com/questions/9398/how-does-eip-150-change-the-call-depth-attack'>\*</a></sup> (모든 가스가 1024 호출 깊이 한도에 다다르기 이전에 소모될 것입니다).