# Persistent layer
## Общие детали имплементации
* Имя базы данных - govfee

## Payment
Информации о платеже. Название коллекции - payments. Ключ - payment-id.

Name | Type | Restrictions | Description
---|---|---|---
*payment-id* | **uuid** | **required**: *true* <br> **format**: *UUIDv1* | Идентификатор платежа
*uin* | **string** | **required**: *true* | Уникальный идентификатор начисления
*sum* | **integer** | **required**: *true* | Сумма начисления (в копейках)
*correlation-id* | **uuid** | **required**: *false* | Идентификатор внешнего контекста 
*version* | **integer** | **required**: *true* | Версия документа
*attempt* | **integer** | **required**: *true* | Номер попытки отправки платежа
*context* | **object** | **required**: *true* <br> **format**: *[ExecutionContext](#ExecutionContextBD)* | Контекст обработки платежа
*created-at* | **datetime** | **required**: *true* | Дата создания платежа
*receipt* | **object** | **required**: *false* <br> **format**: *[PaymentReceipt](#PaymentReceiptBD)* | Квитнция об оплате

### <a name="ExecutionContextBD"></a>ExecutionContext
Name | Type | Restrictions | Description
---|---|---|---
*state* | **enum** | **required**: *true* | Состояние контекста обработки платежа
*task-id* | **uuid** | **required**: *false* | Идентификатор задачи, в рамках которой обрабатывается платеж
*reason* | **string** | **required**: *false* | Описание причины перехода платежа в текущее состояние
*payed-at* | **datetime** | **required**: *false* | Метка времени, когда платеж был завершен
*postponed-to* | **datetime** | **required**: *false* | До какого момента отложен платеж

### <a name="PaymentReceiptBD"></a>PaymentReceipt
Name | Type | Restrictions | Description
---|---|---|---


## Newsfeed
Лента новостей. Название коллекции - payments.newsfeed. Ключ - ObjectId
##### Индексы:
* task-id

Name | Type | Restrictions | Description
---|---|---|---
*payment-id* | **uuid** | **required**: *true* <br> **format**: *UUIDv1* | Идентификатор платежа
*correlation-id* | **uuid** | **required**: *false* | Идентификатор контекста, в рамках которого проводится платеж
*task-id* | **uuid** | **required**: *true* | Идентификатор задачи, в рамках которой обрабатывается платеж
*context* | **object** | **required**: *true* <br> **format**: *[ExecutionContext](#ExecutionContextBD)* | Контекст обработки платежа


## Teko log
События от агента "Teko", при проведении транзакции. Название коллекции - teko.log. Ключ - ObjectId.
##### Индексы:
* paymentId + task-id

Name | Type | Restrictions | Description
---|---|---|---
*payment-id* | **uuid** | **required**: *true* | Идентификатор платежа
*task-id* | **uuid** | **required**: *true* | Идентификатор задачи, в рамках которого проводится платеж
*created-at* | **datetime** | **required**: *true* | Дата создания события
*method* | **enum** | **required**: *true* <br> **items:** <br> &nbsp;&nbsp; *isPaymentPossible* <br> &nbsp;&nbsp; *resumePayment* <br> &nbsp;&nbsp; *cancelPayment* | Метод, повлекший возникновения события
*error* | **object** | **required**: *false* <br> **format**: *[TekoError](#TekoErrorBD)* | Описание ошибки при вызове метода
*transaction* | **object** | **required**: *false* <br> **format**: *[TekoTransaction](#TekoTransactionBD)* | Данные транзакции

### <a name="TekoErrorBD"></a>TekoError
Name | Type | Restrictions | Description
---|---|---|---
*code* | **string** | **required**: *true* | Код ошибки
*description* | **string** | **required**: *true* | Описание ошибки

### <a name="TekoTransactionBD"></a>TekoTransaction
Name | Type | Restrictions | Description
---|---|---|---
*id* | **string** | **required**: *true* | Идентификатор транзакции
*init* | **datetime** | **required**: *true* | Время инициализации транзакции в миллисекундах
*start* | **datetime** | **required**: *true* | Время начала транзакции в миллисекундах
*finish* | **datetime** | **required**: *true* | Время окончания транзакции в миллисекундах

