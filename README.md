# bot-pos-api-specs

봇 - POS 간 원격 주문 시 예상되는 시나리오와 그에 따른 기능, API를 예상해봅니다.

## 시나리오들

각 시나리오의 흐름과 그에 따른 API 예상

### 주문

1. 자연어로 주문한다. (아메리카노 얼음 많이 1개)

1. 메뉴를 조회한다
1. 주문 항목을 선택한다
    1. 어떤걸
    1. 몇개
    1. 옵션 (얼음 적게)(옵션을 자유롭게 적을 수 있을지 혹은 지정된 형식이 있을지)
1. 항목을 추가할 수 있다. (1로 돌아가 반복)
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

### 주문 취소 (option, 낙장불입 정책으로?)

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

봇 → POS → 봇

봇 → POS: `GET` 요청
> 예: <POS_API>/api/v1/stores/:storeId/menu

그에 따른 응답 예상

```json
{
    "version": "v1",
    "store": {
        "storeId": "pan-n8",
        "storeName": "카카오 8층 카페",
        "storeStatus": "open",
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
            "itemOptions": [
                {
                    "optionId": "",
                    "optionName": "얼음 적게",
                    "optionPrice": 0
                },
                {
                    "optionId": "",
                    "optionName": "샷 추가",
                    "optionPrice": 200
                }
            ],
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
            "itemOptions": [
                {
                    "optionId": "",
                    "optionName": "얼음 적게",
                    "optionPrice": 0
                },
                {
                    "optionId": "",
                    "optionName": "샷 추가",
                    "optionPrice": 200
                }
            ],
            "itemStatus": "sale",
            "itemProduceTime": 300
        }
    ],
    "extra": {}
}
```

제품은 이름, id, 이미지, 가격, 할인여부와 할인액, 이 아이템에서 추가할 수 있는 옵션, 판매 여부, 예상 제작 시간 정보를 갖고 있습니다.

### 메뉴 확인

봇 -> POS -> 봇

Entity validation에서 쓰일 예정. 자세한 스펙 추가 필요.

### 옵션 확인

봇 -> POS -> 봇

Entity validation에서 쓰일 예정. 자세한 스펙 추가 필요.

### 주문 요청

봇 → POS → 봇

봇 → POS: `POST` 요청
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
            "itemOptions": [
                "얼음 적게"
            ],
            "itemAmount": 1,
            "itemPrice": 1500
        },
        {
            "itemId": "2",
            "itemName": "아이스 아메리카노 라지",
            "itemOptions": [
                "샷 추가"
            ],
            "itemAmount": 2,
            "itemPrice": 5400
        }
    ],
    "totalPrice": 6900,
    "extra": {}
}
```

Option이 문자열만 넘어가는 건 제조 시 참고만 하면 될 것이라 예상해서입니다. 위의 `option` 을 그대로 넘겨도 무방할 것 같습니다.

POS -> 봇 응답

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

`orderId`는 POS에서 무작위로 생성한 unique id 문자열입니다. waitingCount, waitings 는 전체 대기열을 주시는게 밑의 `주문 상태 조회`쪽과 동일한 로직으로 돌릴 수 있을테니 좋지 않을까요?

### 주문 상태 조회

봇 → POS → 봇

봇 -> POS: `GET` 요청
> 예: <POS_API>/api/v1/order/stores/:storeId/order/:orderId

POS -> 봇 응답

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
                "itemOptions": [
                    "얼음 적게"
                ],
                "itemAmount": 1,
                "itemPrice": 1500
            },
            {
                "itemId": "2",
                "itemName": "아이스 아메리카노 라지",
                "itemOptions": [
                    "샷 추가"
                ],
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

POS → 봇 - ? → POS

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
                "itemOptions": [
                    "얼음 적게"
                ],
                "itemAmount": 1,
                "itemPrice": 1500
            },
            {
                "itemId": "2",
                "itemName": "아이스 아메리카노 라지",
                "itemOptions": [
                    "샷 추가"
                ],
                "itemAmount": 2,
                "itemPrice": 5400
            }
        ]
    }
}
```

### 취소 요청

봇 → POS → 봇

*미정* 취소가 가능할지 여부에 따라 취소 요청 API 존재 여부가 갈리게 됩니다.

가능하다면 HTTP `DELETE` 로 orderId만 넘겨서 취소 요청을 보내고 응답을 받으면 될 듯 합니다.

### 내역 조회

봇 → IBS(?) → 봇

봇 -> IBS `POST` or `GET` 요청

```json
{
    "version": "",
    "user": {
        "krewId": "?",
        "company": ""
    }
}
```

IBS -> 봇 으로 응답