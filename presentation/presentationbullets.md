- scheduler queue-based covert channel

## Sender-receiver image
- showvase of the JavaSQUIP covert channel
- reliably, without use of the network
- breaks browser sandbox
- security risk
- virtually undetected

- how this exploit works
- structured - AMD's consumer CPUs

## Zen2 Arch
- simplified diagram of the way a CPU of AMD's Zen2 family operates
- SMT
- two threads/instruction streams
- uOp
- scheduler queuing, passing results

- smt widely used
- what is not as common, is the layout of the schedulers
- most one big scheduler queue
- Zen 2,3,4, Apple M1
- several smaller Schedulers instead
- reduces complexity, more efficient power usage

- make CPUs more efficient
- affect each others execution time
- possibility for timing attacks

## port contention
- smt
- side channel
- timing diff in an execution unit
- cooperating sender-receiver pair
- receiver issues operations
- sender delays receivers operations

## SQUIP
- SMT in comb with split scheduler
- up one step in the chain
- measures scheduler queue overflows
- mov instruction
- queue overflows
- stall execution
- big difference in execution time

- drawback: split-scheduler
- what advantage does squip have?
- transmission speed
- easier to measure
- signal to noise ratio
- faster rate

## SQUIP in JavaScript
- covert channel as a showcase
- real-world scenario
- communicate only over SQUIP
- sender: simple

## implementing JavaQUIP
- low-level features in browser
- browsers limit functionality of JavaScript for security reasons

- timing on the order of 25ns
- synchronisation between sender and receiver
- fill up one scheduler queue
- colocation

- accurate timing up to one millisecond - own timer
- not allow communication between separate instances
- targeting a scheduler queue
- colocation: bound to be on the same physical core

## Co-Location
- possible distribution of threads
- two logical cores per physical core
- sender in orange
- counting thread
- fills rest with receiving threads
- 1 in 15
- co-location 14 out of 15 tries

## JavaSQUIP - evaluation
- first scheduler queue contention covert channel in a browser setting
- speed and reliability
- brevity - Zen 4
- 1000 bits/s
- co-location 96%
- bit error rate 0.7%
- cumulative error rate 7.45%
- Shannon's noisy coding theorem
- true capacity 613.63 bits/s

## comparison
- most closely resembles JavaSQUIP
- Rocicki and Botvinnik
- raw transmission rate 200 bits/s
- native code - eliminate uncertainty
- packet loss 5.5%
- error rate 1%
- cumulative error rate 6.5%
- true capacity 131.03 bits/s

## Summary
- advantage of the JavaSQUIP CC
- limited to CPUs with SMT and split-scheduler
- more than 3 times the rate
- real-time data: monitoring all user inputs
- short audio files within lifetime of website
- few features
- unlikely: software update can completely mitigate the underlying vuln

I hope i was able to give you a good overview of the JavaSQUIP covert channel, 
and I thank you all for your attention.
