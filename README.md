# bot-pos-api-specs

봇 - POS 간 원격 주문 시 예상되는 시나리오와 그에 따른 기능, API를 예상해봅니다.

## 시나리오들

각 시나리오의 흐름과 그에 따른 API 예상

### 주문

#### 메뉴 조회 & 주문

1. 자연어로 주문한다. (아메리카노 얼음 많이 1개)

혹은

1. 메뉴를 조회한다
1. 주문 항목을 선택한다
    1. 어떤걸
    2. 몇개

그리고

1. 항목을 추가할 수 있다. (1로 돌아가 반복)

#### 주문 완료

1. 주문 완료

* 메뉴 조회 ( 봇 → POS → 봇 )
* 주문 요청 ( 봇 → POS → 봇 )

### 진행 (상태 관리)

1. POS로 주문이 들어옴
1. 주문 진행 상태로 변경
1. 완료될 경우 완료 상태로 변경
1. POS → 봇서버로 push

* 주문 상태 조회 ( 봇 → POS → 봇 )
* 완료 알림 ( POS → 봇 - ? → POS )

### 주문 취소 (optional)

주문 취소는 조금 복잡할 수 있습니다. 초기에는 빼고 가도 될 것 같아요.

1. 주문 내역 조회
1. (가능할 경우) 취소 요청
1. 취소 확인

* 취소 요청 ( 봇 → POS → 봇)

## 예상되는 API

* `version` 필드는 API나 JSON의 포매팅 버전을 의미합니다.
* `extra` 필드는 후에 JSON포맷을 유지하며 추가 정보가 필요할 때 넘기는 key-value 형태로 전달할 수 있도록 확보한 예비 영역입니다.
* `store`나 `item`, `option` 처럼 공통으로 쓰이는 필드와 구조가 있습니다. 다를 경우 오타이니 지적 부탁드립니다.

> Draft로 언제든 변경될 수 있습니다.

### 메뉴 조회

`봇 → POS(POS API) → 봇`

봇 → POS : `GET` 요청
> 예: <POS_API>/api/v1/stores/:storeId/menu

그에 따른 응답 예상

```json
{
    "version": "v1",
    "store": {
        "storeId": "pan-n8",
        "storeName": "카카오 8층 카페",
        "storeStatus": "open"
    },
    "itemCount": 5,
    "items": [
        {
            "itemName": "아이스 아메리카노",
            "itemId": "1",
            "itemImageUrl": "http://path.to",
            "itemPrice": 2000,
            "isDiscount": true,
            "discountAmount": 500,
            "itemStatus": "sale",
            "itemProduceTime": 300
        },
        {
            "itemName": "아이스 아메리카노 라지",
            "itemId": "2",
            "itemImageUrl": "http://path.to",
            "itemPrice": 2500,
            "isDiscount": false,
            "discountAmount": 0,
            "itemStatus": "sale",
            "itemProduceTime": 300
        }
    ],
    "extra": {}
}
```

제품은 이름, id, 이미지, 가격, 할인여부와 할인액, 판매 여부, 예상 제작 시간 정보를 갖고 있습니다.
(할인 같은건 향 후 있지 않을 데이터라면 삭제해도 무방합니다.)

### 메뉴 확인

특정 문자열이 판매 중인 메뉴인지 확인하는 API입니다. 예를 들어 "아이스아메리카노" 가 실제 판매 중인 메뉴 이름인지 질의하고 그에 따른 결과를 받습니다.

`봇 -> POS(POS API) -> 봇`

Entity validation에서 쓰일 예정. 자세한 스펙 추가 필요. 급한게 아니니 일단 스킵.

* 1안: POS API 서버로 항상 물어본다. 파라미터 검증 API를 도입.
* 2안: 하루에 한번 정도 entity를 갱신.
    * 2-1: 수동으로 엔티티를 업데이트 (...)
    * 2-2: 파라미터 검증 API를 쓰긴 하는데 타겟이 우리 내부 서버.

### 주문 요청

최종 흐름은 `"봇 → POS(POS API -> POS 기계 -> POS API) → 봇"` 이지만, [POS API - POS 기계] 데이터 전달 특성 상 2단계를 거쳐서 이루어 집니다.

#### 주문 요청 및 대기

`봇 → POS(POS API) -> 봇`

봇 → POS : `POST` 요청
> 예: <POS_API>/api/v1/stores/:storeId/order

```json
{
    "version": "v1",
    "orderer": {
        "ordererId": "bart.lee",
        "ordererName": "이철민",
        "ordererCompany": "kakao"
    },
    "orderAmount": 3,
    "orders": [
        {
            "itemId": "1",
            "itemName": "아이스 아메리카노",
            "itemAmount": 1,
            "itemPrice": 1500
        },
        {
            "itemId": "2",
            "itemName": "아이스 아메리카노 라지",
            "itemAmount": 2,
            "itemPrice": 5400
        }
    ],
    "totalPrice": 6900,
    "extra": {}
}
```

POS API -> 봇 : 응답

```json
{
    "status": "STANDBY",
    "order": {
        "orderId": "uuid",
        "orderer": {
            "ordererId": "bart.lee",
            "ordererName": "이철민",
            "ordererCompany": "kakao"
        }
    }
}
```

#### 주문 접수 및 확인

POS API 서버와 POS 기계 간 데이터 전달이 완료된 상태입니다. 즉 POS 기계에서 주문 접수 혹은 실패가 실제로 이루어진 상태를 봇쪽으로 PUSH 해주는 상황입니다.

`POS API -> 봇`

```json
{
    "version": "v1",
    "status": "OK",
    "reason": "",
    "order": {
        "orderId": "uuid",
        "orderer": {
            "ordererId": "bart.lee",
            "ordererName": "이철민",
            "ordererCompany": "kakao"
        }
    },
    "store": {
        "storeId": "",
        "storeName": "",
        "storeStatus": "Open",
    },
    "waitingCount": 4,
    "waitings": [
        {
            "orderId": "uuid",
            "orderProduceTime": 300
        },
        {
            "orderId": "uuid",
            "orderProduceTime": 300
        },
        {
            "orderId": "uuid",
            "orderProduceTime": 300
        },
        {
            "orderId": "uuid",
            "orderProduceTime": 300
        }
    ]
}
```

`orderId`는 POS API에서 무작위로 생성한 unique id 문자열입니다. waitingCount, waitings 는 전체 대기열을 주시는게 밑의 `주문 상태 조회`쪽과 동일한 로직으로 돌릴 수 있을테니 좋지 않을까요?

### 주문 상태 조회

스토어 아이디와 주문 아이디를 갖고 현재 어느 상태인지를 질의하는 API 입니다.

`봇 → POS API → 봇`

봇 -> POS : `GET` 요청
> 예: <POS_API>/api/v1/order/stores/:storeId/order/:orderId

POS -> 봇 : 응답

```json
{
    "version": "v1",
    "status": "OK",
    "reason": "",
    "store": {
        "storeId": "",
        "storeName": "",
        "storeStatus": "Open",
    },
    "order": {
        "orderId": "uuid",
        "orderer": {
            "ordererId": "bart.lee",
            "ordererName": "이철민",
            "ordererCompany": "kakao"
        },
        "orderAmount": 3,
        "orders": [
            {
                "itemId": "1",
                "itemName": "아이스 아메리카노",
                "itemAmount": 1,
                "itemPrice": 1500
            },
            {
                "itemId": "2",
                "itemName": "아이스 아메리카노 라지",
                "itemAmount": 2,
                "itemPrice": 5400
            }
        ]
    },
    "waitingOrderCount": 4,
    "waitingOrders": [
        {
            "orderId": "uuid",
            "orderProduceTime": 300
        },
        {
            "orderId": "uuid",
            "orderProduceTime": 300
        },
        {
            "orderId": "uuid",
            "orderProduceTime": 300
        },
        {
            "orderId": "uuid",
            "orderProduceTime": 300
        }
    ]
}
```

대기열이 길어지면 주고받는 데이터가 너무 길어질 것 같은데 Count와 예상 대기 시간을 주는 것도 괜찮아 보입니다.

### 완료 알림

주문한 아이템이 완료되었을 경우 알림을 주는 API 입니다. 전통적인 시스템의 진동벨의 역할입니다.

`(POS기계 -> POS API서버) → 봇`

POS -> 봇 : `POST` 요청
> 예: <BOT_API>/api/v1/order/complete

```json
{
    "version": "",
    "store": {
        "storeId": "",
        "storeName": "",
        "storeStatus": "Open",
    },
    "order": {
        "orderId": "uuid",
        "orderer": {
            "ordererId": "bart.lee",
            "ordererName": "이철민",
            "ordererCompany": "kakao"
        },
        "orderAmount": 3,
        "orders": [
            {
                "itemId": "1",
                "itemName": "아이스 아메리카노",
                "itemAmount": 1,
                "itemPrice": 1500
            },
            {
                "itemId": "2",
                "itemName": "아이스 아메리카노 라지",
                "itemAmount": 2,
                "itemPrice": 5400
            }
        ]
    }
}
```

### 취소 요청

`봇 → POS → 봇`

*미정* 취소가 가능할지 여부에 따라 취소 요청 API 존재 여부가 갈리게 됩니다. 주문 대기 상태일 때 취소가 가능 하지않을까 싶긴 합니다만.

가능하다면 HTTP `DELETE` 로 orderId만 넘겨서 취소 요청을 보내고 응답을 받으면 될 듯 합니다.

### 내역 조회

(KAKAO INTERNAL)

봇 → IBS(?) → 봇
