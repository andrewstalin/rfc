# Implementation details
* Mongodb is used as a database. The database name is "govfee".
* RabbitMQ is used as a message queue. The processing queue name is "govfee.payment".


## HTTP request processing

![](diagrams/images/http-request.svg)

## Payment processing
### Processing flow
![](diagrams/images/processing-flow.svg)

### Execution context switching
![](diagrams/images/context-switching.svg)

## Payment processing


# TODO
* threshold for resuming payment (state=suspend|delayed)