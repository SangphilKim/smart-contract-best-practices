[일반적인 철학](/general_philosophy/)에서 논했던 것처럼, 당신 스스로를 알려진 공격들로부터 방어하는 것만으로는 충분하지 않습니다. 블록체인 상에서의 실패 비용은 매우 높을 수 있기 때문에, 이러한 위험을 고려하여 소프트웨어를 작성하는 방법을 바꿔야 합니다.

우리가 선호하는 접근은 "실패에 대비하라"입니다. 당신의 코드가 안전한지 사전에 아는 것은 불가능 합니다. 그러나 당신은 컨트랙트가 실패하는 것을 최소한의 피해로 너그럽게 받아들일 수 있도록 설계할 수 있습니다. 이 부분은 당신이 실패에 대비하는 것에 도움을 줄 수 있는 다양한 기법들을 보여줍니다.

새로운 구성요소를 당신의 시스템에 추가하는 것은 항상 위험이 있습니다. 형편없이 설계된 고장-안전(fail-safe)은 그 자체로 취약점이 될 수 있습니다 - 다수의 잘 설계된 고장-안전 사이의 상호작용 또한 마찬가지입니다.당신이 컨트랙트에 사용하는 각 기법들에 대해 깊이 생각할 수 있어야 하며, 어떻게 그것들이 함께 동작하여 탄탄한 시스템을 구축할 수 있을지에 대해 신중히 고려해야 합니다.

### 고장난 컨트랙트 업그레이드(Upgrading Broken Contracts)

코드는 오류가 발견되었거나 개선이 필요할 때 변경이 필요하게 됩니다. 버그를 찾는 것은 좋지는 않지만, 버그를 다룰 수 있는 방법도 없습니다.

스마트 컨트랙트를 위해 효율적인 업그레이드 시스템을 설계하는 것은 활발히 연구중인 부문이며 이 문서 내에의 모든 문제들을 해결할 수는 없습니다. 그러나 가장 일반적으로 사용되는 두 가지 기본적인 접근법이 있습니다. 두 가지 중 더 간단한 것은 가장 최신 버전의 컨트랙트 주소를 가지고 있는 등록 컨트랙트를 만드는 것입니다. 컨트랙트 사용자를 위한 좀 더 매끄러운 방법은 최신 버전의 컨트랙트에 호출과 데이터를 전달하는 컨트랙트를 만드는 것입니다.

기법이 무엇이든, 모듈화와 구성요소 사이를 적절히 분리하는 것은 중요하며, 이와 같은 과정을 거친 코드는 변경이 되더라도 기능성을 잃거나, 데이터를 유실하거나 또는 복사에 큰 비용이 발생하지 않게 됩니다. 특히 당신의 데이터 저장공간으로부터 복잡한 로직을 분리하는 것은 대개 효과적이며, 그럼으로써 기능성을 변경하기 위해 모든 데이터를 변경하지 않아도 됩니다.

또한 단체들이 코드를 업그레이드를 결정하는 것을 위한 안전한 방법을 찾는 것도 중요합니다. 당신의 컨트랙트에 따라, 코드의 변경은 신뢰할 수 있는 단일 단체 또는 집단의 구성원들의 승인을 필요로 하거나 전체 주주들의 투표를 필요로 할 수도 있습니다. 만약 이 절차가 어느 정도의 시간을 필요로 한다면, [비상 정지(emergency stop) 또는 서킷 브레이커(circuit-breaker)](#circuit-breakers-pause-contract-functionality) 디자인 패턴과 같이 공격시 보다 더 빠르게 반응할 수 있는 다른 방법들이 있는지를 고려해야 합니다.

**예시 1: 컨트랙트의 최신 버전의 컨트랙트 주소를 저장하기 위한 등록 컨트랙트를 사용**

이 예시에서 호출은 전달되지 않기 때문에, 사용자들은 컨트랙트와 상호작용하기 전에 매번 현재 주소를 받아와야만 합니다.

```sol
contract SomeRegister {
    address backendContract;
    address[] previousBackends;
    address owner;

    function SomeRegister() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner)
        _;
    }

    function changeBackend(address newBackend) public
    onlyOwner()
    returns (bool)
    {
        if(newBackend != backendContract) {
            previousBackends.push(backendContract);
            backendContract = newBackend;
            return true;
        }

        return false;
    }
}
```

이러한 접근 방식에는 두 가지 큰 단점이 있습니다.

1. 사용자가 항상 반드시 현재 주소를 조회해야 하며, 이렇게 하지 않은 사용자들은 오래된 버전의 컨트랙트를 사용하는 위험을 감수해야 합니다.
2. 당신이 컨트랙트를 바꿀 때 컨트랙트의 데이터를 어떻게 다뤄야 할지 신중히 생각해야 합니다.

대안은 호출과 데이터를 최신 버전의 컨트랙트로 전달하는 컨트랙트를 사용하는 것입니다.

**예시 2: [`델리게이트 콜(DELEGATECALL)`을 사용](http://ethereum.stackexchange.com/questions/2404/upgradeable-contracts)해 데이터와 호출을 전달합니다.**

```sol
contract Relay {
    address public currentVersion;
    address public owner;

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    function Relay(address initAddr) {
        currentVersion = initAddr;
        owner = msg.sender; // 이 소유주는 한 명의 컨트랙트 소유주가 아닌 다중 서명된 또 다른 컨트랙트일 수 있습니다.
    }

    function changeContract(address newVersion) public
    onlyOwner()
    {
        currentVersion = newVersion;
    }

    function() {
        require(currentVersion.delegatecall(msg.data));
    }
}
```

이 접근 방식은 이전의 문제를 피할 수는 있지만 그 자체의 문제를 가지고 있습니다. 당신은 컨트랙트 안에 데이터를 저장하는 방법에 대해 극도로 신경써야만 합니다. 만약 당신의 새로운 컨트랙트가 첫 번째와 다른 형태로 저장소를 만들었다면, 결국 당신의 데이터에는 오류가 발생할 것입니다. 추가로 이 패턴의 간단한 버전은 함수로부터 값을 반환할 수 없으며, 오직 응용에 한계를 가진 값들을 전달만 할 수 있습니다. ([더 복잡한 구현들](https://github.com/ownage-ltd/ether-router)은 이 문제를 인라인(in-line) 어셈블리 코드와 반환 값들의 저장소로 해결하는 것을 시도합니다.)

당신의 접근 방식에도 불구하고, 당신의 컨트랙트를 업그레이드하는 몇 가지 방법을 가지는 것은 중요합니다. 그렇지 않으면 컨트랙트들은 불가피한 버그들이 컨트랙트들에서 발견된다면 사용할 수 없게 될 것입니다.

### 서킷 브레이커(Circuit Breakers, 함수의 기능을 정지시킴)

서킷 브레이커는 특정 조건이 만족되면 실행을 중지시킵니다. 그리고 새로운 버그들이 발견되었을 때 유용합니다. 예를 들어, 대부분의 동작들이 컨트랙트 내에서 버그가 별견되었다면 중단될 것입니다. 그리고 가능한 유일한 동작은 인출 뿐입니다. 당신은 특정한 신뢰할 수 있는 단체에게 서킷 브레이커를 작동시킬 수 있는 능력을 주거나, 아니면 특정 조건이 만족되었을 때 자동으로 서킷 브레이커가 작동되게 하는 프로그램 규칙을 넣을 수 있습니다.

예시:

```sol
bool private stopped = false;
address private owner;

modifier isAdmin() {
    require(msg.sender == owner);
    _;
}

function toggleContractActive() isAdmin public {
    // 당신은 사용자들의 투표와 같이 다른 동작에 기반하여 컨트랙트를 멈추는 것을 제한하는 수정자를 추가할 수 있습니다.
    stopped = !stopped;
}

modifier stopInEmergency { if (!stopped) _; }
modifier onlyInEmergency { if (stopped) _; }

function deposit() stopInEmergency public {
    // some code
}

function withdraw() onlyInEmergency public {
    // some code
}
```

### 속도 제한(Speed Bumps, 컨트랙트의 동작을 지연시킴)

속도 제한은 동작을 느리게 함으로써, 악의적인 행동이 발생하더라도 복구할 시간을 확보합니다. 예를 들어, [The DAO](https://github.com/slockit/DAO/)는 DAO를 나누는 성공적인 요청과 이를 수행하는데 27일을 필요로 합니다. 이것은 자금이 컨트랙트 내에 보관되는 것을 보장하고, 복구 가능성을 높입니다. DAO의 경우 속도 제한에 의해 주어지는 시간 동안 할 수 있는 효율적인 행동이 없습니다. 그러나 우리의 다른 기법들과의 조합한다면, 매우 효과적일 것입니다.

예시:

```sol
struct RequestedWithdrawal {
    uint amount;
    uint time;
}

mapping (address => uint) private balances;
mapping (address => RequestedWithdrawal) private requestedWithdrawals;
uint constant withdrawalWaitPeriod = 28 days; // 4주

function requestWithdrawal() public {
    if (balances[msg.sender] > 0) {
        uint amountToWithdraw = balances[msg.sender];
        balances[msg.sender] = 0; // 간결함을 위해, 우리는 모든 것을 인출할 것입니다;
        // 아마 예금 함수는 인출이 진행중이라면 새로운 예금을 막습니다.

        requestedWithdrawals[msg.sender] = RequestedWithdrawal({
            amount: amountToWithdraw,
            time: now
        });
    }
}

function withdraw() public {
    if(requestedWithdrawals[msg.sender].amount > 0 && now > requestedWithdrawals[msg.sender].time + withdrawalWaitPeriod) {
        uint amountToWithdraw = requestedWithdrawals[msg.sender].amount;
        requestedWithdrawals[msg.sender].amount = 0;

        require(msg.sender.send(amountToWithdraw));
    }
}
```

### 빈도수 제한(Rate Limiting)

빈도수 제한은 큰 변화에 대해 중단 또는 승인을 요청합니다. 예를 들어, 예금자는 일정 기간이 경과한 후에 특정 금액 또는 전체 예금의 일정 비율을 인출할 수 있습니다(예를 들면 하루 이후에 최대 100 이더) - 그 기간 중의 추가 인출은 실패하거나 특별한 승인 등이 필요합니다. 또한 빈도수 제한은 일정 기간 동안 컨트랙트에 의해 발행되는 특정 양의 토큰들과 같이 컨트랙트 수준에서도 가능합니다.

[예시](https://gist.github.com/PeterBorah/110c331dca7d23236f80e69c83a9d58c#file-circuitbreaker-sol)

### 컨트랙트 공개(Contract Rollout)

컨트랙트는 상당한 그리고 장기간의 테스트 기간을 가져야만 합니다 - 상당한 돈이 위험에 놓이기 전에.

최소한 당신은 다음과 같이 해야합니다:

- 100% 테스트 범위(또는 이에 근접한)를 가진 테스트 묶음을 가지고 있어야 합니다.
- 당신만의 테스트넷에 배포해야 합니다.
- 많은 테스트와 버그 바운티를 위해 공개 테스트넷에 배포해야 합니다.
- 철저한 테스트를 통해 더 큰 규모에서 다양한 참가자들이 컨트랙트를 통해 상호작용할 수 있게 해야 합니다.
- 메인넷에 위험도를 제한한 베타 버전으로 배포해야 합니다.

##### 자동 기능 중지(Automatic Deprecation)

테스트 중 당신은 특정 기간 이후에 동작들을 막음으로써 자동 기능 중지를 만들어낼 수 있습니다. 예를 들어, 알파 컨트랙트는 몇 주간 동작을 했고, 마지막 인출 기능을 제외한 모든 기능을 자동으로 중지했습니다.

```sol
modifier isActive() {
    require(block.number <= SOME_BLOCK_NUMBER);
    _;
}

function deposit() public isActive {
    // some code
}

function withdraw() public {
    // some code
}

```
##### 사용자/컨트랙트의 이더의 양을 제한(Restrict amount of Ether per user/contract)

초기 단계에 당신은 사용자의 이더의 양을 제한할 수 있었습니다 (또는 전체 컨트랙트에 대해) - 위험을 감소시킵니다.

### 버그 바운티 프로그램(Bug Bounty Programs)

바운티 프로그램을 운영하기 위한 몇 가지 팁:

- 바운티의 보상으로 어떤 화폐가 분배될 것인지를 결정해야 합니다 (BTC 그리고/또는 이더).
- 바운티 보상을 위해 예상되는 전체 예산을 결정해야 합니다.
- 예산에서 보상의 3가지 단계를 결정합니다.
  - 최소 보상 (흔쾌히 제공 합니다)
  - 최대 보상 (보통 보상받을만 합니다)
  - 추가 범위의 보상 (매우 심각한 취약점들인 경우 보상을 제공합니다)
- 누가 바운티 심사위원을 할지 결정합니다 (3명이 이상적인 숫자로 일반적입니다).
- 리드 개발자는 바운티 심사위원 중 한 명이어야 합니다.
- 버그 보고서를 받으면, 리드 개발자는 심사위원들의 조언을 참고해 버그의 심각도을 평가합니다.
- 이 단계에서의 작업은 개인(private) 레파지토리에 있어야 합니다. 그리고 문제(issue)는 깃허브에 보관됩니다.
- 개인 레파지토리에서 버그가 수정되었다면, 개발자는 실패해야 하며 버그를 확인할 수 있는 테스트 케이스를 작성해야만 합니다.
- 개발자는 수정 사항을 구현하고 테스트가 이제 통과한다는 것을 확인해야 합니다; 필요시 추가 테스트 케이스를 작성합니다.
- 바운티 헌터에게 수정 사항을 공개합니다; 공개 레파지토리에 수정사항을 병합(merge)시키는 것도 한 가지 방법입니다.
- 바운티 헌터가 수정 사항에 대한 또 다른 피드백이 있는지 결정합니다.
- 바운티 심사위원들은 버그의 *발생가능성*과 *영향력* 두 가지의 평가를 바탕으로 보상의 규모를 결정합니다.
- 과정 중에 발생하는 정보를 바운티 참가자들에게 계속해서 제공하고, 그들에게 보상을 보내는데 지연되는 것을 막기 위해 노력합니다.

보상의 3가지 단계에 대한 예시는, [이더리움의 바운티 프로그램](https://bounty.ethereum.org)을 참고하세요:

> 지불되는 보상의 가치는 영향의 심각도에 기반해 달라집니다. 중요하지 않은 '무해한' 버그들의 보상은 0.05 BTC부터 시작합니다. 합의 문제까지 이어질 수 있는 중요한 버그들은 5BTC까지 보상이 올라가게 됩니다. 매우 심각한 취약점들의 경우 더 높은 보상이 가능합니다 (최대 25BTC 까지).