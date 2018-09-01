# 토큰 구현을 위한 모범 사례

토큰을 구현하는것은 다른 모범 사례를 지켜야 하지만 또한 몇 가지 고유한 고려사항도 가져야만 합니다.

## 최신 표준 준수

일반적으로 토큰의 스마트 컨트랙트는 승인되고 안정한 표준을 따라야 합니다.

현재 승인된 표준들의 예시입니다:

* [EIP20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)
* [EIP721](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md) (대체할 수 없는 토큰)

## EIP-20에 대한 선행매매 공격을 알아야 합니다.

EIP-20 토큰의 `approve()` 함수는 인가된 소비자가 의도한 양보다 더 많이 소비할 수 있는 가능성을 만듭니다. [선행매매 공격](./known_attacks/#transaction-ordering-dependence-tod-front-running)이 사용될 수 있으며, 이것은 `approve()` 호출이 처리되기 전후에 인가된 소비자가 `transferFrom()`을 호출할 수 있도록 합니다. 더 자세한 사항은 [EIP](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md#approve)와 [이 문서](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit)에서 확인할 수 있습니다.


## 0x0 주소로의 토큰 전송 방지

이 글을 작성중인 현재, "영(0)" 주소([0x0000000000000000000000000000000000000000](https://etherscan.io/address/0x0000000000000000000000000000000000000000))는 총 8,000만 달러 이상의 가치를 가진 토큰들을 가지고 있습니다.


## 컨트랙트 주소로의 토큰 전송 방지

또한 스마트 컨트랙트의 주소와 같은 주소로의 토큰 전송을 방지하는 것을 고려해야 합니다.

이것을 열려있는 상태로 남겨둠으로써 발생하는 손실의 가능성에 대한 예시는 [EOS 토큰 스마트 컨트랙트](https://etherscan.io/address/0x86fa049857e0209aa7d9e616f7eb3b3b78ecfdb0)로 90,000개 이상의 토큰들이 컨트랙트 주소에 갇혀있습니다.

### 예시

위의 권장 사항들을 모두 구현한 예시는 다음의 수정자(modifier)를 만들어야 합니다; "받는 사람(to)" 주소가 0x0과 스마트 컨트랙트 자신의 주소가 아닌지를 검증합니다.

```sol
    modifier validDestination( address to ) {
        require(to != address(0x0));
        require(to != address(this) );
        _;
    }
```

그리고 수정자를 "transfer"과 "transferFrom" 메서드에 적용해야 합니다.

```sol 
    function transfer(address _to, uint _value)
        validDestination(_to)
        returns (bool) 
    {
        (... 당신의 로직 ...)
    }

    function transferFrom(address _from, address _to, uint _value)
        validDestination(_to)
        returns (bool) 
    {
        (... 당신의 로직 ...)
    }
```