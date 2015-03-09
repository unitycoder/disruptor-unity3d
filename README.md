# disruptor-unity3d
Basic implementation of Disruptor for Unity3d. Only supports a single producer/single consumer.

## Usage

Copy [RingBuffer.cs](https://github.com/dave-hillier/disruptor-unity3d/blob/master/DisruptorUnity3d/Assets/RingBuffer.cs)  into your Unity project's assets folder (or sub-folder) and use the generic `RingBuffer` class.

## Motivation

[Unity3d](http://unity3d.com/) is a game engine with Mono embedded in it. The version of Mono inside Unity is very old; it doesnt have the SGen collector and uses the [Boehm GC](http://www.hboehm.info/gc/). The Boehm GC does not have great performance. 

When working on a Unity project I've found myself in need of a [Queue](http://en.wikipedia.org/wiki/Queue_%28abstract_data_type%29) for sending messages between threads. The current version of Mono contains an implementation of [ConcurrentQueue](https://github.com/mono/mono/blob/effa4c07ba850bedbe1ff54b2a5df281c058ebcb/mcs/class/corlib/System.Collections.Concurrent/ConcurrentQueue.cs). 

My game uses a queue pretty intensively. When I want to send a message, it is enqueued to a queue and then subsequently dequeued by another thread, which serializes and sends it. When profiling, I've seen that the queue seems to be the source of the ocasional slow frame because it allocates with every message that is queued. The allocations can cause GC stalls at any time, but sometimes it doesnt doesnt explicitly appear on the profiler (for example, the profiler will show a long time in allocation itself and a reduction in total memory usage). 

I wanted to replace the `ConcurrentQueue` with something that did not have any extra allocation overhead. A [circular buffer](http://en.wikipedia.org/wiki/Circular_buffer) is a fixed size data structure that could be used in this case. When searching for an existing implementation I remembered the [Disruptor](https://lmax-exchange.github.io/disruptor/); a high performance lockless queue. 

There is a [.Net port](https://github.com/disruptor-net/Disruptor-net) of the Disruptor, but it is for .Net 4 which is not supported by Unity. I did not want to spend the time porting it and I have a very much simpler use case. I've implemented a very simple, self-contained version that uses the volatile long that is key to the implementation. 


## Benchmark

I've created a simple benchmark in Test.cs for my simple single producer/single consumer RingBuffer.Each frame a batch of random integers are queued. The values are dequeued and discarded by the other thread. 

### Concurrent Queue

![ConcurrentQueue profile](https://raw.githubusercontent.com/dave-hillier/disruptor-unity3d/master/readme-img/ConcurrentQueueProfile.png)

### RingBuffer

![RingBuffer profile](https://raw.githubusercontent.com/dave-hillier/disruptor-unity3d/master/readme-img/RingBufferProfile.png)