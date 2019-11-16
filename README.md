# cretask
A distributed task queue where you can disconnect while the cluster runs the task and persistently stores the result. Built on CouchDB, RabbitMQ and Erlang.

1. Create a CouchDB view which lists tasks to be done. The documents corresponding to the tasks are user-specific and are never changed by other users. Tasks must contain
        1. task document key
        2. a status
        3. a generic task
        4. a result key
2. cretask will see tasks with status "incomplete"
    1. Subscribe to task-specific status messages (RabbitMQ) 
    2. Queue a single query-message (RabbitMQ)
    3. A cretask query-consumer will get this query-message (RabbitMQ)
        1. Query the entire cluster for the task status
        2. After a while, if there are no responses, start the task
        4. Only now, ack the query-message (because this consumer might fail before starting the task)
    4. Wait for task-specific status messages
        1. Update the status in CouchDB
        2. If complete, pull (i.e. replicate) the result (CouchDB) and unsubscribe
        3. If timeout, this means the consumer running the task has failed and the query-message must be retried
