- scheduler queue-based covert channel

## Sender-receiver image
- reliably, without use of the network
- breaks browser sandbox
- security risk
- virtually undetected

- how this exploit works
- structured - AMD's consumer CPUs

## Zen2 Arch
- simplified diagram of the way a CPU of AMD's Zen2 family operates
- SMT
- uOp
- scheduler queuing, passing results

- several smaller Schedulers instead

- makes CPUs more efficient
- affect each others execution time
- possibility for timing attacks

## port contention
- targets an execution unit
- receiver issues operations
- sender delays receivers operations

## SQUIP
- up one step in the chain
- measures scheduler queue overflows
- almost completely
- queue overflows
- stall execution
- big spike
- easier to measure - faster than port-contention attacks
- relying on split-scheduler layout with smaller scheduler queues
- applicability to AMD CPUs

## SQUIP in JavaScript
- browsers limit functionality of JavaScript for security reasons
- colocation
- timing on the order of 25ns
- synchronisation between sender and receiver
- fill up one scheduler queue

- colocation: bound to be on the same physical core
- accurate timing up to one millisecond - own timer
- not allow communication between separate instances
- targeting a scheduler queue

## JavaSQUIP
- first scheduler queue contention covert channel in a browser setting
- split schedulers - limited to AMD CPUs
- speed of 1000 bits per second
- five times faster than state of the art in port contention covert channels in a browser
- increase transmission speed: replacing datetime with self-synchronizing cipher

I hope i was able to give you a good overview of the JavaSQUIP covert channel, and I thank you all for your attention.
