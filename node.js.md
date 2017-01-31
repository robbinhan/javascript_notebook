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



