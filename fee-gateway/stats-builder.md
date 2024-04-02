## Детали реализации
Вместо TAsyncTableStatsBuilder
``` C++
class TAsyncTableStatsBuilder : public TActorBootstrapped<TAsyncTableStatsBuilder>
```
определяем корутинный актор TTableStatsCoroBuilder
``` C++
class TTableStatsCoroBuilder : public TActorCoroImpl, IPages 
``` 

Реализуемый интерфейс IPages, при вызове метода TryGetPage не будет генерировать page fault. Если страница отсутствует в локальной коллекции, происходит отправка события NSharedCache::TEvRequest и приостановка корутины до получения события NSharedCache::TEvResult.\
Все подгруженные страницы, как и прежде хранятся в локальной коллекции.\
После обработки каждого TPart, проверяем продолжительность работы корутины. В случае превышения некоторого значения, прерываем выполнение корутины путем отправки себе сообщения и ожидания его получения.\
Расширяем интерфейс метода построения статистики BuildStats для проверки времени работы корутины. Добавим функтор в аргументы метода, при вызове которого будем проверять продолжительность выполнения корутины: start - now. В случае превышения допустимого времени, отправляем себе событие TTableStatsCoroBuilder::EvResume и прерываем корутину до момента его получения.\
За начало времени работы корутины будем считать время возобновления исполнения корутины или начала работы актора.

### Схема работы актора
```mermaid
flowchart TD
    A[Start] --> B
    B -->|suspend coroutine| C1
    C1 -->|error| Z
    C1 -->|success| C

    C -->|next| D[Get page]
    C -->|done| Y
    D -->|success| E
    D -->|page fault| G[Send NSharedCache::TEvRequest]
    G -->|suspend coroutine| F[Wait NSharedCache::TEvResult]
    F -->|sucess| E
    F -->|error| Z
    C2 -->|exhausted| C3
    C2 -->|continue| C
    C3 -->|suspend coroutine| C4
    C4 -->|resume coroutine| C

    B[Send TEvResourceBroker::TEvSubmitTask]
    C1[Wait TEvResourceBroker::TEvResourceAllocated]
    C2[Check execution time]
    C3[Send TTableStatsCoroBuilder::EvResume]
    C4[Wait TTableStatsCoroBuilder::EvResume]
    C{Foreach DataShard parts}
    E[Build page stats] --> C2
    Y[Send TDataShard::TEvPrivate::TEvAsyncTableStats]
    Z[Terminate]
```
