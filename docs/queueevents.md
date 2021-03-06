# Queue AMQP Exchanges

##

The queue, typically available at `queue.taskcluster.net`, is responsible
for accepting tasks and track their state as they are executed by
workers. In order ensure they are eventually resolved.

This document describes AMQP exchanges offered by the queue, which allows
third-party listeners to monitor tasks as they progress to resolution.
These exchanges targets the following audience:
 * Schedulers, who takes action after tasks are completed,
 * Workers, who wants to listen for new or canceled tasks (optional),
 * Tools, that wants to update their view as task progress.

You'll notice that all the exchanges in the document shares the same
routing key pattern. This makes it very easy to bind to all messages
about a certain kind tasks.

**Task specific routes**, a task can define a task specific route using
the `task.routes` property. See task creation documentation for details
on permissions required to provide task specific routes. If a task has
the entry `'notify.by-email'` in as task specific route defined in
`task.routes` all messages about this task will be CC'ed with the
routing-key `'route.notify.by-email'`.

These routes will always be prefixed `route.`, so that cannot interfere
with the _primary_ routing key as documented here. Notice that the
_primary_ routing key is always prefixed `primary.`. This is ensured
in the routing key reference, so API clients will do this automatically.

Please, note that the way RabbitMQ works, the message will only arrive
in your queue once, even though you may have bound to the exchange with
multiple routing key patterns that matches more of the CC'ed routing
routing keys.

**Delivery guarantees**, most operations on the queue are idempotent,
which means that if repeated with the same arguments then the requests
will ensure completion of the operation and return the same response.
This is useful if the server crashes or the TCP connection breaks, but
when re-executing an idempotent operation, the queue will also resend
any related AMQP messages. Hence, messages may be repeated.

This shouldn't be much of a problem, as the best you can achieve using
confirm messages with AMQP is at-least-once delivery semantics. Hence,
this only prevents you from obtaining at-most-once delivery semantics.

**Remark**, some message generated by timeouts maybe dropped if the
server crashes at wrong time. Ideally, we'll address this in the
future. For now we suggest you ignore this corner case, and notify us
if this corner case is of concern to you.



## QueueEvents Client

```js
// Create QueueEvents client instance with default exchangePrefix:
// exchange/taskcluster-queue/v1/

const queueEvents = new taskcluster.QueueEvents(options);
```

## Exchanges in QueueEvents Client

```js
// queueEvents.taskDefined :: routingKeyPattern -> Promise BindingInfo
queueEvents.taskDefined(routingKeyPattern)
```

```js
// queueEvents.taskPending :: routingKeyPattern -> Promise BindingInfo
queueEvents.taskPending(routingKeyPattern)
```

```js
// queueEvents.taskRunning :: routingKeyPattern -> Promise BindingInfo
queueEvents.taskRunning(routingKeyPattern)
```

```js
// queueEvents.artifactCreated :: routingKeyPattern -> Promise BindingInfo
queueEvents.artifactCreated(routingKeyPattern)
```

```js
// queueEvents.taskCompleted :: routingKeyPattern -> Promise BindingInfo
queueEvents.taskCompleted(routingKeyPattern)
```

```js
// queueEvents.taskFailed :: routingKeyPattern -> Promise BindingInfo
queueEvents.taskFailed(routingKeyPattern)
```

```js
// queueEvents.taskException :: routingKeyPattern -> Promise BindingInfo
queueEvents.taskException(routingKeyPattern)
```

```js
// queueEvents.taskGroupResolved :: routingKeyPattern -> Promise BindingInfo
queueEvents.taskGroupResolved(routingKeyPattern)
```