# Implementation details
* Mongodb is used as a database. The database name is "govfee".
* RabbitMQ is used as a message queue. The processing queue name is "govfee.payment".


## HTTP request processing

```plantuml

(*) -right-> "Received a new \nresuest"
--> "Save the payment \ninto the database"
If "check existence" then
--> [not exist] "Put the task in the \nprocessing queue"
--> "Return 201"
-->[Ending process] (*)
else
--> If "check payment state" then
--> [state=Created]"Put the task in the \nprocessing queue"
else
--> "Return 200"
-->[Ending process] (*)
Endif

Endif

```

## Payment processing
### Processing flow

```plantuml
start

:Try to "capture" payment;
note right
<b>Conditions</b>
state=Suspend or state=Created
state=Processing && task-id="CurrentTaskId"
state=Delayed && postponed-to <= "CurrentTime"

<b>Set ExecutionContext</b>
state=Processing
task-id="CurrentTaskId"
end note

if (Is success) then
-> True;
:Processing the payment by the agent;
-> Switch the context by ExecutionResult;
split
   :Finish;
split again
   :Suspend;
split again
   :Delayed;
   :Schedule a new processing;
end split

:Ack the task;
else
-> False;
:Ack the task;
endif

stop
```

### Execution context switching
```plantuml

[*] --> created
created: New Payment\n
created: ExecutionContext
created: state: Created

created --> processing
processing: Payment processing\n
processing: ExecutionContext
processing: state: Processing
processing: taskId: "string"

processing --> delayed
delayed: Processing is delayed \nuntil the moment of time\n
delayed: ExecutionContext
delayed: state: Delayed
delayed: postponedTo: "timestamp"
delayed --> processing : Changed by a new task

processing --> suspend
suspend: Processing is suspend \nuntil the cause is resolved \n
suspend: ExecutionContext
suspend: state: Suspend
suspend: cause: "string"
suspend --> processing : Changed by a new task

processing --> finished
finished: Processing completed successfully \n
finished: ExecutionContext
finished: state: Finished
finished: payedAt: "timestamp"
finished --> [*]

```

## Payment processing


# TODO
* threshold for resuming payment (state=suspend|delayed)