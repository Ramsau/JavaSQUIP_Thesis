## Intro
Hello, my name is Christoph Royer and I'm going to talk to you about JavaSQUIP, 
which is a scheduler-queue covert channel across two separate browser instances.

## Sender-receiver image
Here you can see a screenshot of our showcase of the JavaSQUIP covert channel. 
The sender can reliably send data to the receiver without use of the network.
This breaks the browser sandbox, because normally different browser instances should not be able to
communicate with each other directly.
This of course poses a significant security risk,
as an attacker executing malicious code - for example in an embedded ad - 
can pass sensitive data from one browser instance to another virtually undetected.
[This could be used to exfiltrate data from a website ...]

To understand how this exploit works we first need to look at the way modern CPUs are structured - 
especially AMD's consumer CPUs.

## Zen2 Arch
Here you can see a simplified diagram of the way a CPU of AMD's Zen2 family operates. 
Zen2 supports simultaneous multithreading,
which means that a single CPU core takes in instructions from two threads at the same time.
After the instructions are split up into smaller Microinstructions,
the retire control unit passes these Microinstructions on to the schedulers.
The scheduler then takes care of queuing microinstructions 
and passing the results back to the retire control unit,
and back to the respective thread.

Simultaneous multithreading is a widely used feature among modern CPUs.
What is not as common, however, is the layout of the schedulers in the Zen2 architecture.
Whereas some manufacturers rely on a single, larger scheduler queue, 
architectures like AMD's Zen 2, Zen3 and Zen 4, or the Apple M1 use several smaller scheduler queues.
Sometimes one scheduler only services one execution unit.
This reduces the complexity of the CPU and enables more efficient power usage.

These two features - simultaneous multithreading and split scheduler queues - make CPUs more efficient.
But because two threads are executing on the same CPU core,
that means that two threads can also affect each other's execution time.
As you can imagine, this opens up the possibility for timing attacks.

## Port contention
In port contention attacks, the attacker exploits simultaneous multithreading 
to build a side channel based on timing differences in an execution unit.
In the special case of a covert channel, we assume a cooperating sender-receiver pair.
The receiver issues operations, for example on ALU1, and measures the time until they are finished.
The sender can send a 1 by also using ALU1, or a 0 by staying idle.
The receiver can detect a 1 by measuring a higher delay in the execution time of its instructions.

## SQUIP: scheduler queue contention
The SQUIP covert channel exploits simultaneous multithreading in combination with split scheduler queues.
It moves the attack up one step in the chain: 
instead of measuring timing on an execution unit, the receiver measueres scheduler queue overflows.
For this, the receiver fills up a scheduler queue almost completely.
If the sender now issues microinstructions to this scheduler, its queue overflows.
Because the retire control unit has no place to put this microinstruction, it needs to stall execution as a whole.

SQUIP has the drawback of relying on a split-scheduler layout with smaller scheduler queues ,
which limits its applicability compared to port contention covert channels.
This begs the question: What advantage does SQUIP have over port contention side channels?
The answer is transmission capacity: 
a scheduler queue stall affects the whole CPU, making it easier to detect than port contention.
This increases the signal-to-noise ratio of the covert channel,
which in turn lets us transmit data at a faster rate than port-contention attacks.

Since port contention has been successfully exploited in a browser environment, 
our aim was to do the same with SQUIP.

## SQUIP in JavaScript
So now we want to implement the SQUIP exploit in Javascript.
The goal is to construct a covert channel as a showcase that reflects 
a real-world scenario as closely as possible.
We created two webpages; one for the sender, and one for the receiver.
The two should communicate only over the SQUIP covert channel
Since the sender corresponds to the victim in a non-cooperative side-channel attack,
we aimed to keep it as simple as possible, while putting most of the logic in the receiver

## implementing JavaSQUIP
To implement a side-channel exploit in JavaScript is a challenge compared to native code,
since many of the low-level features are not available in the browser sandbox.
Additionally, Browsers nowadays limit the functionality of Javascript for security reasons, 
so we faced some challenges.
- the first was accurate timing; we need timing on the order of 25ns to measure scheduler queue contention
- synchronisation: between sender and receiver, to synchronize the start and end of the transmission
- a way to fill up one specific scheduler queue.
- and lastly we need a way to get colocation of the receiver and the sender
  - as the receiver needs to be located on the same physical core as the sender for this covert channel to work

Luckily, we were able to find a way to solve all of these problems:

- Browsers only give accurate timing up to one millisecond,
reportedly to prevent the exact type of attack we are developing. [maybe write this better]
So we implemented our own timer with a counting thread,
which just continually increments a shared variable;
all the other threads can now use this variable as an accurate timer.
- Of course the browser does not allow communication between two separate instances, the whole point of this exploit is to break this limitation.
But there is one thing that is synchronized globally: the DateTime.
It is a millisecond-accurate measure of the time elapsed since the beginning of the year 1970.
With it, we can synchronize transmissions up to 1 thousand times per second, which is currently the limiting factor for transmission speed.
- Targeting a scheduler queue was comparatively easy: JavaScript provides the Math.imul() function,
which translates exactly to an imul operation on the CPU, when it is compiled by JavaScripts JIT compiler.
With this, we can fill up the queue for ALU1 on AMD Zen2 CPUs.
- JavaScript does not provide the functionality to pick the core that a thread should be executed on.
However, the WebWorkers API gives us the ability to create multiple threads.
To ensure that one thread of the receiver would be co-located with the sender,
we took a rather brute force approach:
We created as many receiving threads as were needed to place one thread on each logical core of the CPU.
Since all of them do continuous work, the scheduler will distribute them across the logical cores evenly.
This means that one thread was bound to be on the same physical core as the sending thread.

## Co-Location
You can see here one possible distribution of threads on the CPU - 
with two logical cores for each physical core.
Since the sender should be kept simple, it runs on only one thread, here in orange.
The receiver on the other hand creates one counting thread, to provide accurate timing,
and fills the rest with receiver threads.
Now the only way that the sender is not co-located with one of the receivers,
is that it is co-located with the counting thread, which is only a 1 in 15 chance.
So we get co-location 14 out of 15 times.

Putting it all together, we get JavaSQUIP, the first scheduler queue contention covert channel in a browser setting.
## JavaSQUIP - evaluation
To evaluate the speed and reliability of our solution, we conducted tests where we sent random data between 
the sender and the receiver, and compared the sent and received data.
For the sake of brevity, we will look at the results for the newest AMD architecture, Zen 4.
Because of the limitations of synchronized clocks, the raw transmission speed was fixed at 1000 bits/s.
We were able to detect successful co-location of the sender and one of the receivers 96% of the time.
Given co-location, the transmitted data had a bit error rate of 0.7%.
Factoring these rates together, we calulate a cumulative error rate of 7.45%.
According to Shannon's noisy coding theorem, our covert channel has a true capacity of 
617.63 bits/s after error correction.

## comparison to port contention (portable)
I now want to compare these results to the covert channel that most closely resembles JavaSQUIP:
Rokicki's and Botvinnik's adaptation of port contention side channels to a browser setting.
Their solution runs at a raw transmission rate of 200 bits/s.
They implemented the sender in native code, where they were able to bind their thread to a core.
Thus, they eliminated the uncertainty of co-location.
However, they reported a packet loss rate of 5.5% and an error rate of 1%, 
resulting in a cumulative error rate of 6.45%.
This gives port contention a true capacity of 131.03 bit/s.

## Summary
This shows the advantage of the JavaSQUIP covert channel, 
even though its attack surface is limited to CPUs with SMT and split schedulers.
It can transmit data at more than 3 times the rate of the next best alternative, 
with speeds that can be used to transmit real-time data like all user inputs, 
or short audio files within the lifetime of a website.
JavaSQUIP relies only on few features of the browser environment, 
and we have shown that it is unlikely that a software update can completely mitigate the underlying vulnerability.


I hope i was able to give you a good overview of the JavaSQUIP covert channel, and I thank you all for your attention.
