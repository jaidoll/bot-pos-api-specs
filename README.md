# bot-pos-api-specs

봇 - POS 간 원격 주문 시 예상되는 시나리오와 그에 따른 기능, API를 스펙 정리

## POS -> BOT

api prefix : `/api/kakao/v1`

**주문 접수 완료**
---
  
  Returns json data about a items.

* **URL**

  /order/cook

* **Method:**

  `POST`
  
* **URL Params**

   **Required:**

   `storeId=[string]`

* **Data Params**

  None

* **Success Response:**

  * **Code:** 200 <br />
    **Content:** `{ id : 12, name : "Michael Bloom" }`

* **Error Response:**

  * **Code:** 404 NOT FOUND <br />
    **Content:** `{ error : "User doesn't exist" }`

* **Sample Call:**

  ```python
  ```



**완료 알림**
---
  POS -> BOT

* **URL**

  /order/complete

* **Method:**

  `POST`
  
* **URL Params**

   **Required:**

   `storeId=[string]`

* **Data Params**

  None

* **Success Response:**

  * **Code:** 200 <br />
    **Content:** `{ id : 12, name : "Michael Bloom" }`

* **Error Response:**

  * **Code:** 404 NOT FOUND <br />
    **Content:** `{ error : "User doesn't exist" }`

* **Sample Call:**

  ```python
  ```

## BOT -> POS

api prefix : `/api/v1`

**주문**
---
  Returns json data about a items.

* **URL**

  /stores/:storeId/items

* **Method:**

  `GET`
  
* **URL Params**

   **Required:**

   `storeId=[string]`

* **Data Params**

  None

* **Success Response:**

  * **Code:** 200 <br />
    **Content:** `{ id : 12, name : "Michael Bloom" }`

* **Error Response:**

  * **Code:** 404 NOT FOUND <br />
    **Content:** `{ error : "User doesn't exist" }`

* **Sample Call:**

  ```python
  ```

**주문 상태 조회**
---
  Returns json data about a items.

* **URL**

  /stores/:storeId/orders/:orderId/numbs/:numb

* **Method:**

  `GET`
  
* **URL Params**

   **Required:**

   * `storeId=[string]`
   * `ORDERID=[string]`
   * `NUMB=[string]`
   

* **Data Params**

  None

* **Success Response:**

  * **Code:** 200 <br />
    **Content:** `{ id : 12, name : "Michael Bloom" }`

* **Error Response:**

  * **Code:** 404 NOT FOUND <br />
    **Content:** `{ error : "User doesn't exist" }`

* **Sample Call:**

  ```python
  ```

**주문 취소**
---
  BOT -> POS
  주문 취소 요청. 조리 중일 때는 주문 취소 불가.

* **URL**

  /cancel

* **Method:**

  `DELETE`
  
* **URL Params**

   **Required:**

   `storeId=[integer]`

* **Data Params**

  None

* **Success Response:**

  * **Code:** 200 <br />
    **Content:** `{ id : 12, name : "Michael Bloom" }`

* **Error Response:**

  * **Code:** 404 NOT FOUND <br />
    **Content:** `{ error : "User doesn't exist" }`

* **Sample Call:**

  ```python
  ```

**상품 정보 수신**
----
  Returns json data about a items.

* **URL**

  /stores/:storeId/items

* **Method:**

  `GET`
  
* **URL Params**

   **Required:**

   `storeId=[string]`

* **Data Params**

  None

* **Success Response:**

  * **Code:** 200 <br />
    **Content:** `{ id : 12, name : "Michael Bloom" }`

* **Error Response:**

  * **Code:** 404 NOT FOUND <br />
    **Content:** `{ error : "User doesn't exist" }`

* **Sample Call:**

  ```python
  ```
