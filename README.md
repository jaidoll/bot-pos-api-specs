# bot-pos-api-specs

봇 - POS 간 원격 주문 시 예상되는 시나리오와 그에 따른 기능, API를 예상해봅니다.

## 시나리오들

각 시나리오의 흐름과 그에 따른 API 예상

### 주문

1. 메뉴를 조회한다
1. 주문 항목을 선택한다
    1. 어떤걸
    1. 몇개
    1. 옵션 (얼음 적게)
1. 항목을 추가할 수 있다. (1로 돌아가 반복)
1. 주문 완료

> 메뉴 조회 ( 봇 → POS → 봇 )\n
> 주문 요청 ( 봇 → POS → 봇 )

### 진행 (상태 관리)

1. POS로 주문이 들어옴
1. 주문 진행 상태로 변경
1. 완료될 경우 완료 상태로 변경
1. POS → 봇서버로 push

> 주문 상태 조회 ( 봇 → POS → 봇 )\n
> 완료 알림 ( POS → 봇 - ? → POS )

### 주문 취소 (option, 낙장불입 정책으로?)

1. 주문 내역 조회
1. (가능할 경우) 취소 요청
1. 취소 확인

> 취소 요청 ( 봇 → POS → 봇)

## 예상되는 API

`version` 필드는 API나 JSON의 포매팅 버전을 의미합니다. `extra` 필드는 후에 JSON포맷을 유지하며 추가 정보가 필요할 때 넘기는 key-value 형태로 전달할 수 있도록 확보한 예비 영역입니다.
> Draft로 언제든 변경될 수 있습니다.

### 메뉴 조회

봇 → POS → 봇

봇 → POS: `GET` 요청
> 예: <POS_API>/api/v1/stores/:storeId/menu

그에 따른 응답 예상

```json
{
    "version": "",
    "storeId": "pan-n8",
    "storeStatus": "open",
    "itemCount": 5,
    "items": [
        {
            "name": "",
            "id": "",
            "price": 2000,
            "isDiscount": false,
            "discountAmount": 500,
            "options": [
                "", ""
            ],
            "status": "notSale"
        },
        {
            "name": "아메리카노",
            "id": "",
            "price": 2000,
            "isDiscount": false,
            "discountAmount": 500,
            "options": [
                "", ""
            ],
            "status": "sale"
        }
    ],
    "extra": {}
}
```

### 주문 요청

봇 → POS → 봇

봇 → POS: `POST` 요청
> 예: <POS_API>/api/v1/stores/:storeId/order

```json
{
    "version": "",
    "orderId": "",
    "orderer": {
        "id": "",
        "name": "",
        "company": ""
    },
    "orderCount": 1,
    "orders": [
        {
            "id": "",
            "name": "",
            "options": [],
            "amount": 0
        }
    ],
    "extra": {}
}
```

POS -> 봇 응답

```json
{
    "version": "",
    "status": "",
    "reason": "",
    "orderInfo": {
        "orderId": "",
        "orderer": {
            "id": "",
            "name": "",
            "company": ""
        }
    },
    "storeInfo": {
        "status": "",
        "waitingCount": 0,
        "waitings": [
        ]
    }
}
```

### 주문 상태 조회

봇 → POS → 봇

봇 -> POS: `GET` 요청
> 예: <POS_API>/api/v1/order/stores/:storeId/order/:orderId

POS -> 봇 응답
> 주문 시와 같습니다.

```json
{
    "version": "",
    "status": "",
    "reason": "",
    "orderInfo": {
        "orderId": "",
        "orderer": {
            "id": "",
            "name": "",
            "company": ""
        }
    },
    "storeInfo": {
        "status": "",
        "waitingCount": 0,
        "waitings": [
        ]
    }
}
```

### 완료 알림

POS → 봇 - ? → POS

POS -> 봇 : `POST` 요청
> 예: <BOT_API>/api/v1/order/complete

```json
{
    "version": "",
    "orderId": "",
    "orderInfo": {
        "orderer": {
            "id": "",
            "name": "",
            "company": ""
        },
        "orderCount": 1,
        "orders": [
            {
                "id": "",
                "name": "",
                "options": [],
                "amount": 0
            }
        ],
        "extra": {}
    }
}
```

### 취소 요청

봇 → POS → 봇

*미정*

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

*미정*