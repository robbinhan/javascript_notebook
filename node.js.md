> ## Node.js timers
>
> [Node.js timers](https://nodejs.org/api/timers.html), which have the same API as[window timers](https://developer.mozilla.org/es/docs/Web/API/WindowTimers/setTimeout)in Web API, are very useful, easy to schedule / deschedule and are used extensively across the entire ecosystem.
>
> As such, it’s likely that there may be a large amount of timeouts scheduled at any given time in an application.
>
> Similar to other[hashed wheel timers](http://cseweb.ucsd.edu/users/varghese/PAPERS/twheel.ps.Z), Node.js[uses a hash table and a linked list](https://github.com/nodejs/node/blob/master/lib/timers.js)to maintain the timers instances. But unlike other wheel timers, instead of having a fixed-length hash table, it keys each list of timers by duration.
>
> When the key exists \(a timer with the same duration exist\), it is appended to the bucket as a O\(1\) operation.
>
> When a key does not exist, a bucket is created and the timer is appended to it.
>
> With that in mind, you have to make sure you reuse the existing buckets, trying to avoid removing a whole bucket and creating a new one. For example, if you are using sliding delays you should create the new timeout \(setTimeout\(\)\) before removing \(clearTimeout\(\)\) the previous one.
>
> In our case, by[scheduling the idle timeout \(heartbeat\)](https://github.com/datastax/nodejs-driver/blob/v3.1.6/lib/connection.js#L434-L443)before removing the previous one, we make sure the scheduling and descheduling of idle timeouts are O\(1\) operations.
>
> ## System calls
>
> Libuv exposes a platform-independent API that is used by Node.js to perform non-blocking IO and your application IO \(sockets, file system, ...\) ultimately translates into system calls.
>
> There is a significant cost in scheduling those system calls. You should try to minimize the amount of syscalls by can grouping / batching writes.
>
> When using a socket or a file stream, instead of issuing a write every time, you can buffer and flush the data from time to time.
>
> You can use a write queue to process and group your writes. The logic for a write queue implementation should be something like:
>
> * While there are items to write and we are within the window size
>   * Push the buffer to the “to-write list”
> * Concatenate all the buffers in the list and write it to the wire.
>
> You can define a window size based either on the total buffer length or the amount of time that passed since the first item was queued. Defining a window size is tradeoff between the latency of a single write and the average latency of all writes. You should also consider the maximum amount of write requests to be grouped and the overhead of generating each write request.
>
> You would generally want to flush writes of buffers in the order of kilobytes. We found a sweet spot around 8 kilobytes, but your mileage may vary. You can check out the[implementation in the client driver](https://github.com/datastax/nodejs-driver/blob/v3.1.6/lib/writers.js#L159)for a complete implementation of a write queue.
>
> Grouping or batching writes will translate into higher throughput thanks to less system calls.



