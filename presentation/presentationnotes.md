Hello, my name is Christoph Royer and I'm going to talk to you about JavaSQUIP, 
which is a scheduler queue-based covert channel across two separate browser instances.

## Sender-receiver image
Here you can see a screenshot of the JavaSQUIP covert channel in action. 
The sender can reliably send data to the receiver without use of the network.
This breaks the browser sandbox, because normally different browser instances should not be able to communicate with each other directly.
This of course poses a significant security risk, as an attacker executing malicious code - for example in an embedded ad - 
can pass sensitive data from one browser instance to another virtually undetected.

But to understand how this exploit works we first need to look at the way modern CPUs are structured - especially AMD's consumer CPUs.

## Zen2 Arch
Here you can see a simplified diagram of the way a CPU of AMD's Zen2 family operates. 
Zen2 supports simultaneous multithreading, which means that a single CPU core takes in instructions from two threads at the same time.
After the instructions are split up into smaller Microinstructions, the retire control unit passes these Microinstructions on to the schedulers.
The scheduler then takes care of queuing microinstructions and passing the results back to the retire control unit.

And this is where AMD CPUs differ from other manufacturers:
they have several smaller Schedulers instead of one big one to balance these microinstructions - 
sometimes one scheduler only services one execution unit.

All of this makes CPUs more efficient.
But because two threads are executing on the same CPU core, that means that two threads can also affect each other's execution time.
As you can imagine, this opens up the possibility for timing attacks.

## Port contention
In port contention attacks, the attacker targets an execution unit. 
The receiver issues operations, for example on ALU1, and measures the time until they are finished.
The sender can send a 1 by also using ALU1, which delays the receiver's operations more than if the sender was not using ALU1 - which equals sending a 0.

## SQUIP: scheduler queue contention
SQUIP moves the attack up one step in the chain. 
Instead of measuring timing on an execution unit, the receiver measueres scheduler queue overflows.
For this, the receiver fills up a scheduler queue almost completely.
If the sender now issues microinstructions to this scheduler, its queue overflows.
Because the retire control unit has no place to put this microinstruction, it needs to stall execution as a whole.
This results in a comparatively big spike in execution time.
Since this makes contention easier to measure, it is possible to run SQUIP faster than port-contention attacks.
But SQUIP also has the drawback of relying on a split-scheduler layout with smaller scheduler queues ,
which limits its applicability mostly to AMD CPUs.

## SQUIP in JavaScript
So now we want to implement the SQUIP covert channel in Javascript.
As you may know, Browsers nowadays limit the functionality of Javascript for security reasons, so we faced some challenges.
- the first was colocation, as the receiver needs to be located on the same physical core as the sender for this covert channel to work
- accurate timing; we need timing on the order of 25ns to measure scheduler queue contention
- synchronisation: between sender and receiver, to synchronize the start and end of the transmission
- and lastly we need a way to fill up one specific scheduler queue.

Luckily, we were able to find a way to solve all of these problems:
- For colocation, we took a rather brute force approach: We created as many receiving threads as were needed to place one thread on each physical core of the CPU.
This means that one thread was bound to be on the same physical core as the sending thread.
- Browsers only give accurate timing up to one millisecond, reportedly to prevent the exact type of attack we are developing.
So we implemented our own timer with a counting thread, which just continually increments a shared variable; all the other threads can now use this variable as an accurate timer.
- Of course the browser does not allow communication between two separate instances, the whole point of this exploit is to break this limitation.
But there is one thing that is synchronized globally: the DateTime.
It is a millisecond-accurate measure of the time elapsed since the beginning of the year 1970.
With it, we can synchronize transmissions up to 1 thousand times per second, which is currently the limiting factor for transmission speed.
- Targeting a scheduler queue was comparatively easy: JavaScript provides the Math.imul() function, which translates exactly to an imul operation on the CPU, filling up the queue for ALU1 on AMD Zen2 CPUs.

## JavaSQUIP
Putting it all together, we get JavaSQUIP, the first scheduler queue contention covert channel in a browser setting.
Because it relies on split schedulers, its usage is limited mostly to AMD CPUs.
On the upside however, JavaSQUIP has a transmission speed of up to 1000 bits per second, which is five times faster than the state of the art in browser-based port contention covert channels.
Since the bottleneck is the cross-instance synchronisation,
we expect to be able to increase transmission speed even further by replacing the dateTime synchronisation with a self-synchronizing cipher,
such as the Manchester code.

I hope i was able to give you a good overview of the JavaSQUIP covert channel, and I thank you all for your attention.

