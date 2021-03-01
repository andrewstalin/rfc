
# Публичный контракт
## Модель данных
### Схема данных

#### Payment
```json
{
  "traceId": "string",
  "paymentId": "uuid",
  "uin": "string",
  "sum": "integer",
  "correlationId": "uuid",
  "receipt": {
  },
  "createdAt": "datetime",
  "context": {
    "state": "enum",
    "taskId": "string",
    "payedAt": "datetime",
    "postponedTo": "datetime"
  }
}
```

#### Newsfeed
```json
{
  "traceId": "string",
  "news": [{
    "paymentId": "uuid",
    "correlationId": "uuid",
    "state": "enum"
  }],
  "offset": "string"
}
```

#### TekoLog
```json
{
  "traceId": "string",
  "paymentId": "uuid",
  "log": [{
    "taskId": "uuid",
    "responses": [{
        "createdAt": "datetime",
        "method": "string",
        "error": {
          "code": "string",
          "description": "string"
        },
        "transaction": {
          "id": "string",
          "init": "datetime",
          "start": "datetime",
          "finish": "datetime"
        }
    }]
  }]
}
```

### Описание типов

#### Payment
Name | Type | Restrictions | Description
---|---|---|---
*traceId* | **string** | **required**: *false* | Идентификатор контекста трассировки
*paymentId* | **uuid** | **required**: *true* <br> **format**: *UUIDv1* | Идентификатор платежа
*uin* | **string** | **required**: *true* | Уникальный идентификатор начисления
*sum* | **integer** | **required**: *true* | Сумма начисления (в копейках)
*correlationId* | **uuid** | **required**: *false* | Идентификатор внешнего контекста
*receipt* | **object** | **required**: *false* <br> **format**: *[PaymentReceipt](#PaymentReceiptAPI)* | Квитнция об оплате
*createdAt* | **datetime** | **required**: *false* | Дата создания платежа
*context* | **object** | **required**: *false* <br> **format**: *[ExecutionContext](#ExecutionContextAPI)* |

##### <a name="PaymentReceiptAPI"></a>PaymentReceipt
Name | Type | Restrictions | Description
---|---|---|---

##### <a name="ExecutionContextAPI"></a>ExecutionContext
Name | Type | Restrictions | Description
---|---|---|---
*state* | **enum** | **required**: *true* | Состояние контекста обработки платежа
*taskId* | **uuid** | **required**: *false* | Идентификатор задачи, в рамках которой обрабатывается платеж
*attempt* | **integer** | **required**: *true* | Номер попытки отправки платежа
*payedAt* | **datetime** | **required**: *false* | Метка времени, когда платеж был завершен
*postponedTo* | **datetime** | **required**: *false* | До какого момента отложен платеж

#### Newsfeed
Name | Type | Restrictions | Description
---|---|---|---
*traceId* | **string** | **required**: *false* | Идентификатор контекста трассировки
*news* | **array** | **required**: *true* <br> **format**: *[NewsItem[]](#NewsItem)* |
*offset* | **string** | **required**: *true* | Смещение в ленте новостей

##### <a name="NewsItem"></a>NewsItem
Name | Type | Restrictions | Description
---|---|---|---
*paymentId* | **uuid** | **required**: *true* <br> **format**: *UUIDv1* | Идентификатор платежа
*correlationId* | **uuid** | **required**: *false* | Идентификатор внешнего контекста
*context* | **object** | **required**: *true* <br> **format**: *[ExecutionContext](#ExecutionContextAPI)* |

#### TekoLog
Name | Type | Restrictions | Description
---|---|---|---
*paymentId* | **uuid** | **required**: *true* | Идентификатор платежа
*log* | **array** | **required**: *false* <br> **format**: *[TekoLogItem[]](#TekoLogItem)* | Лог обработки, в рамках взаимодействия с Теко

##### <a name="TekoLogItem"></a>TekoLogItem
Name | Type | Restrictions | Description
---|---|---|---
*taskId* | **uuid** | **required**: *true* | Идентификатор задачи, в рамках которого проводится платеж
*responses* | **array** | **required**: *false* <br> **format**: *[TekoResponse[]](#TekoResponse)* | Массив ответов от Теко 

##### <a name="TekoResponse"></a>TekoResponse
Name | Type | Restrictions | Description
---|---|---|---
*createdAt* | **datetime** | **required**: *true* | Дата создания события
*method* | **enum** | **required**: *true* <br> **items:** <br> &nbsp;&nbsp; *isPaymentPossible* <br> &nbsp;&nbsp; *resumePayment* <br> &nbsp;&nbsp; *cancelPayment* | Метод, повлекший возникновения события
*error* | **object** | **required**: *false* <br> **format**: *[TekoError](#TekoErrorAPI)* | Описание ошибки при вызове метода
*transaction* | **object** | **required**: *false* <br> **format**: *[TekoTransaction](#TekoTransactionAPI)* | Данные транзакции

##### <a name="TekoErrorAPI"></a>TekoError
Name | Type | Restrictions | Description
---|---|---|---
*code* | **string** | **required**: *true* | Код ошибки
*description* | **string** | **required**: *true* | Описание ошибки

##### <a name="TekoTransactionAPI"></a>TekoTransaction
Name | Type | Restrictions | Description
---|---|---|---
*id* | **string** | **required**: *true* | Идентификатор транзакции
*init* | **datetime** | **required**: *true* | Время инициализации транзакции в миллисекундах
*start* | **datetime** | **required**: *true* | Время начала транзакции в миллисекундах
*finish* | **datetime** | **required**: *true* | Время окончания транзакции в миллисекундах

## API
### Создание платежа
#### Request
```http request
POST govfee/v1/payments HTTP/1.1
Content-Type: application/json; charset=utf-8
Trace-Id: value

{
  "paymentId": "uuid",
  "uin": "string",
  "sum": "integer",
  "correlationId": "uuid"
}
```
#### Response
```http request
HTTP/1.1 201 OK
ETag: value
Trace-Id: value

{ Payment }
```

[comment]: <> (При существовании платежа с указаным paymentId)
```http request
HTTP/1.1 200 OK
ETag: value
Trace-Id: value

{ Payment }
```

```http request
HTTP/1.1 403 Forbidden
Trace-Id: value
```

```http request
HTTP/1.1 400 Bad Request
Trace-Id: value

{
  "traceId": "",
  "code": "",
  "message": ""
}
```

### Получение платежа
#### Request
```http request
GET govfee/v1/payments/{paymentId} HTTP/1.1
If-None-Match: etag
Trace-Id: value
```
#### Response
```http request
HTTP/1.1 200 OK
ETag: value
Trace-Id: value

{ Payment }
```

```http request
HTTP/1.1 304 Not Modified
ETag: value
Trace-Id: value
```

```http request
HTTP/1.1 404 Not Found
Trace-Id: value
```

```http request
HTTP/1.1 403 Forbidden
Trace-Id: value
```

```http request
HTTP/1.1 400 Bad Request
Trace-Id: value

{
  "traceId": "",
  "code": "",
  "message": ""
}
```

### Лента новостей
#### Request
```http request
GET govfee/v1/payments/newsfeed?offset=value HTTP/1.1
Trace-Id: value
```
#### Response
```http request
HTTP/1.1 200 OK
ETag: value
Trace-Id: value

{ Newsfeed }
```

```http request
HTTP/1.1 403 Forbidden
Trace-Id: value
```

```http request
HTTP/1.1 400 Bad Request
Trace-Id: value

{
  "traceId": "",
  "code": "",
  "message": ""
}
```

### Взаимодействие с Теко
#### Журнал взаимодействия в рамках платежа
##### Request

```http request
GET govfee/v1/teko/log/{paymentId} HTTP/1.1
Trace-Id: value
```
##### Response
```http request
HTTP/1.1 200 OK
Trace-Id: value

{ TekoLog }
```

```http request
HTTP/1.1 403 Forbidden
Trace-Id: value
```

```http request
HTTP/1.1 404 Not Found
Trace-Id: value
```

```http request
HTTP/1.1 400 Bad Request
Trace-Id: value

{
  "traceId": "",
  "code": "",
  "message": ""
}
```

#### Проброс вызовов API
##### Requests

```http request
POST govfee/v1/teko/is-payment-possible HTTP/1.1

{
  "uin": "string",
  "sum": "integer",
  "partnerTransactionInit": "integer"
}
```

```http request
POST govfee/v1/teko/resume-payment HTTP/1.1

{
  "uin": "string",
  "sum": "integer",
  "partnerTransactionInit": "integer",
  "partnerTransactionId": "string",
  "partnerTransactionStart": "integer"
}
```

```http request
POST govfee/v1/teko/cancel-payment HTTP/1.1

{
  "uin": "string",
  "sum": "integer",
  "partnerTransactionInit": "integer",
  "partnerTransactionId": "string",
  "partnerTransactionStart": "integer"
}
```

# TODO
* отмена платежа
* принудительное возобновление платежа