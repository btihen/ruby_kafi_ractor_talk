---
# try also 'default' to start simple
theme: light-icons
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS (experimental)
css: unocss
---

# Ruby Ractors

**Ruby Actors** - _Escaping Ruby's Global-Lock_

**Parallel & Concurrent Computing with Ruby 3.x**


**by Bill Tihen**
<br><br>
<small>
**Slides @ https://github.com/btihen/ruby_kafi_ractor_talk**<br>
_Adapted from my [Ractor Post](https://btihen.dev/posts/ruby/ruby_3_x_ractor/)_ @ https://btihen.dev
</small>

---
layout: image-right
image: /images/british-library-Gw_UOoFk4Wk-unsplash.jpg
---

# Topics Outline

* **What?, Why?, When?**
* **Components**
* **Life-cycle**
* **Messaging Options**
* **Design Options & Code**

Photo by <a href="https://unsplash.com/@britishlibrary?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">British Library</a> on <a href="https://unsplash.com/s/photos/map?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/kyle-head-p6rNTdAPbuk-unsplash.jpg
---

# **Ruby Actors**

Ruby Actor like models that can avoid the Global Lock

## Features

* only use message passing
* mandatory object safety (frozen or scope transfer)
* parallel & concurrent computing

<small>Photo by <a href="https://unsplash.com/@kyleunderscorehead?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Kyle Head</a> on <a href="https://unsplash.com/photos/p6rNTdAPbuk?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

---
layout: image-right
image: /images/brett-jordan-_Xwnk1DgTb8-unsplash.jpg
---

# **Ruby Actors**

When to use?

**Scenarios**

* Independent Tasks
* CPU intensive
* Supervision

**NOTE**
* Experimental
* Not always faster (test)
  - object creation & optimized algorithms
  - Short-lived output & async data fetches

Photo by <a href="https://unsplash.com/de/@brett_jordan?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Brett Jordan</a> on <a href="https://unsplash.com/s/photos/when?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/joe-neric-AkEMVbdEmUQ-unsplash.jpg
---

# Comparison

Using Fibonacci recursion (not efficient, but cpu heavy)<br>def fib(n) = n < 2 ? 1 : fib(n-2) + fib(n-1)

| **Code Method**   | **Time (Sec)** | **Notes**         |
| ------------------| --------------:|:------------------|
| Fibonacci(39)     | **6.5**    | **Min Possible Time** |
| Ractors unlimited | **6.6**        | _All CPUs (7)_    |
| Ractor Pool (4)   | **7.9**        | _4 CPUs_          |
| Thread Pool (4)   | 17.3           | 4 Threads (1 CPU) |
| Single Thread     | 17.4           | 1 Thread (main)   |

Photo by <a href="https://unsplash.com/@jneric?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Joe Neric</a> on <a href="https://unsplash.com/collections/2337461/comparison-meeting?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-right
image: /images/01_ractor_components.png
---

# Components

* **in-port** - accepts messages when open / listening
* **in-box** (queue) - Infinite (limited by RAM)
* **object data** - immutable (frozen) or isolated to ractor
* **code** - executable block
* **out-port** (out-box) - result storage (queue length = 1)

---
layout: image-left
image: /images/evie-s-zn4Pl32WgWM-unsplash.jpg
---

# Lifecycle

* **running** - ractor is processing
* **blocking** - in-port is open & to _waiting_ to process
* **terminated** - in-port is closed & nothing to process <br> _(may still have results to share)_
* **exceptions** terminates a ractor <br> _(and all downstream Ractors in a pipeline - except Main)_


Photo by <a href="https://unsplash.com/@evieshaffer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Evie S.</a> on <a href="https://unsplash.com/images/people/life?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-right
image: /images/ishan-seefromthesky-4xmgrNUbyNA-unsplash.jpg
---

## Single-use Life-Cycle Example

```ruby
r1 = Ractor.new(1, name: 'r1') do |i|
  puts "Executing '#{i} + 2'"
  i + 2
end
# Executing '1 + 2' # runs immediately with a queued message and empty outport
# => #<Ractor:#2 r1 (irb):3 running>
# inspect already shows 'terminated' - in-port is closed and nothing to run
r1.inspect # however, we out-port still has a result to collect ()
# => "#<Ractor:#1 r1 (irb):8 terminated>"
# we can't send another request to a terminated ractor (in-port is closed)
r1.send(1) # `send': The incoming-port is already closed (Ractor::ClosedError)
# out-port is open, get the ractor the result
r1.take # note - 'take' blocks our current thread to wait for the result
# => 3
# now the out-port is empty and is closed
r1.take # `take': The outgoing-port is already closed (Ractor::ClosedError)
```

Photo by <a href="https://unsplash.com/@seefromthesky?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Ishan @seefromthesky</a> on <a href="https://unsplash.com/s/photos/single-use?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>


---
layout: image-left
image: /images/matt-seymour-8X2siC3gSj4-unsplash.jpg
---

## Multi-use Life-Cycle Example

```ruby
r1 = Ractor.new(name: 'r1') do
  loop do # internal Ractor infinite loop allows multi-use
    input = Ractor.receive # receive pulls from the inbox
    result = input + 2
    puts "Running - inbox had: #{input} - outbox has: #{result}"
    Ractor.yield(result)
  end
end
# => #<Ractor:#3 r1 (irb):14 blocking>
r1.inspect # still blocking waiting for input
r1.send(1) # puts is seen immediately (since outport is open)
# Running - inbox had: 1 - outbox has: 3
# => #<Ractor:#3 r1 (irb):14 blocking>
r1.send(2) # add another message - doesn't execute yet (out-box full)
# => #<Ractor:#3 r1 (irb):14 blocking>
r1.take # gets 1st result (3) & executes next message (fill out-box)
# Running - inbox had: 2 - outbox has: 4
# => 3
r1.take # get 2nd result, nothing in the in-box (nothing executes)
# => 4
r1.take # CAREFUL: nothing in out-box so we are blocked - oops!
```

Photo by <a href="https://unsplash.com/@mattseymour?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Matt Seymour</a> on <a href="https://unsplash.com/s/photos/circular?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-right
image: /images/yogendra-singh-BxHnbYyNfTg-unsplash.jpg
---

## Sudden Death (Exceptions)

```ruby
# queue & process a message immediately - reports crash and running?
r1 = Ractor.new('a') { |i| i + 2 }
#<Thread:0x000000010782d1d0 run> terminated with exception (report_on_exception is true):
# (irb):30:in `+': no implicit conversion of Integer into String (TypeError)
# 	from (irb):30:in `block in <top (required)>'
# => #<Ractor:#5 (irb):30 running>

# ractor inspect confirms ractor died - from runtime exception
r1.inspect
# => "#<Ractor:#9 r1 (irb):58 terminated>"
```

**Note**: Trapping exceptions and creating replacements is the basis of Supervision. _When using pipelines all downstream ractors need to be recreated too!_

Photo by <a href="https://unsplash.com/@yogendras31?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Yogendra Singh</a> on <a href="https://unsplash.com/s/photos/problem?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/guilherme-stecanella-SZ80v2lmhSY-unsplash.jpg
---

# Messaging/Design Options

```
Message Summary
                  +------------------------------------------+
   r1.send(obj)-->*-->[incoming queue]   Ractor.yield(obj)-->*-->r1.take
              r1: |         v                    ^           |
                  |   Ractor.receive ----> Block-Execution   |
                  +------------------------------------------+

Push Pipeline:    +----------------+   r2: +-------------------+
              r1: | r2.send(obj)---:------>*--> Ractor.receive *
                  +----------=-----+       +-------------------+

Pull Pipeline:    +---------------------+      r2: +-----------+
              r1: * Ractor.yield(obj)-->*----------:-->r1.take |
                  +---------------------+          +-----------+

Ractor Pool:      +---------------------+
              r1: * Ractor.yield(obj)-->*---+
                  +---------------------+   |
                                            +--> Ractor.select(r1, r2)
                  +---------------------+   |    (take ready outboxes)
              r2: * Ractor.yield(obj)-->*---+
                  +---------------------+
```

Photo by <a href="https://unsplash.com/@guilhermestecanella?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Guilherme Stecanella</a> on <a href="https://unsplash.com/s/photos/message?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-right
image: /images/mohammadreza-alidoost-0rUp9vgyEYo-unsplash.jpg
---

## Ractor Messaging Designs

* **Ring**
* **Pipeline**
* **Fork-Join**
  - short-lived tasks
* **Worker-Pool**
  - long running tasks
* **Supervision**

Photo by <a href="https://unsplash.com/@mralidoost?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Mohammadreza alidoost</a> on <a href="https://unsplash.com/s/photos/display?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>


---
layout: image-left
image: /images/aaron-jones-IJbfutoo7_U-unsplash.jpg
---

# Pipeline Code

```ruby
def incrementor(n) = n + 1
def doubler(n) = n * 2
r1 = Ractor.new do
  loop do
    input_r1 = Ractor.receive
    incremented = incrementor(input_r1)
    p incremented: [input_r1, incremented]
    Ractor.yield(incremented)
  end
end
r2 = Ractor.new(r1) do |r1|
  loop do
    input_r2 = r1.take
    doubled = doubler(input_r2)
    p doubled: [input_r2, doubled]
    Ractor.yield(doubled)
  end
end
r1.send(1); r1.send(4)
r2.take; r2.take
```

Photo by <a href="https://unsplash.com/@ajonesyyyyy?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Aaron Jones</a> on <a href="https://unsplash.com/s/photos/pipeline?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-right
image: /images/matheo-jbt-HLhvZ9HRAwo-unsplash.jpg
---

# Ring Code (circular pipeline)

```ruby
MAX_FIB_NUM = 39.freeze
def fib(n) = n < 2 ? 1 : fib(n-2) + fib(n-1)
CR = Ractor.current

r = Ractor.new do
  p Ractor.receive
  CR << :done # alias of CR.send(:done)
end

MAX_FIB_NUM.times{
  r = Ractor.new(r) do |next_r|
    input = Ractor.receive
    p result: [input, fib(input)]
    next_r << (input - 1) # alias of next_r.send(input - 1)
  end
}

t1 = Time.now
r << MAX_FIB_NUM # alias of r.send(MAX_FIB_NUM)
p Ractor.receive
puts "Ring - duration #{Time.now - t1}"
```

Photo by <a href="https://unsplash.com/@matheo_jbt?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Matheo JBT</a> on <a href="https://unsplash.com/s/photos/circle?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/mae-mu-Pvclb-iHHYY-unsplash.jpg
---

# Fork-Join (pool consumption)

**Very FAST** - Uses as many resources as allowed and then gone

```ruby
MAX_FIB_NUM = 39.freeze
CALC_LIST = MAX_FIB_NUM.downto(1).map { |i| i }.freeze
def fib(n) = n < 2 ? 1 : fib(n-2) + fib(n-1)
results = []

t1 = Time.now
pool = CALC_LIST.map do |val| # Ractor per value wanted
  Ractor.new(val) { |n| [n, fib(n)] }
end

until pool.empty? # Collect results until work completed
  dead_worker, value = Ractor.select(*pool) # get completed work
  p answer: value # so we see something while working
  pool.delete dead_worker # remove terminated Ractors
  results << value # collect results
end
puts "Fork Join - full cpu usage - duration #{Time.now - t1}"
```

Photo by <a href="https://unsplash.com/de/@picoftasty?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Mae Mu</a> on <a href="https://unsplash.com/s/photos/fork?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-right
image: /images/redd-f-yinfkjyiptY-unsplash.jpg
---

## Worker Pool (Load Balancer) Code

```ruby
MAX_CPUS = 4.freeze; MAX_FIB_NUM = 39.freeze
CALC_LIST = MAX_FIB_NUM.downto(1).map { |i| i }.freeze
def fib(n) = n < 2 ? 1 : fib(n-2) + fib(n-1)

pool = Ractor.new do
  loop { Ractor.yield( Ractor.receive ) }
end
workers = (1..MAX_CPUS).map do |i|
  Ractor.new(pool) do |p|
    loop do
      input = p.take
      fib_num = fib(input)
      p result: [input, fib_num]
      Ractor.yield([input, fib_num])
    end
  end
end

results = []; t1 = Time.now
CALC_LIST.each { |i| pool.send(i) } # send/start calculations
CALC_LIST.each { results << Ractor.select(*workers) } # collect results
puts "Worker Pool - with #{MAX_CPUS} CPUs - duration #{Time.now - t1}"
```

Photo by <a href="https://unsplash.com/@raddfilms?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Redd F</a> on <a href="https://unsplash.com/s/photos/workers?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>


---
layout: image-left
image: /images/megan-bucknall-V9-i2ORDQLA-unsplash.jpg
---

## Pool Supervisor Code

```ruby
def fib(n) = n < 2 ? 1 : fib(n-2) + fib(n-1)
def make_pool = Ractor.new { loop { Ractor.yield(Ractor.receive) } }
def make_worker(pool)
  Ractor.new(pool) do |p|
    loop { val = p.take; result = fib(val); Ractor.yield([val, result]) }
  end
end
def make_workers(pool, count) = (1..count).map { |i| make_worker(pool) }
def send_work(input, pool) = Array(input).each { |i| pool.send(i) }
def collect_results(input, pool, workers, results = [])
  input.count.times do # important to only run this for number of inputs
    begin
      worker, answer = Ractor.select(*workers) # returns worker & result
      results << answer # we only collect the answers
    rescue Ractor::RemoteError => e
      worker = e.ractor # capture dead worker
      workers.delete(worker) # remove dead worker
      workers << make_worker(pool) # create new worker
    end
  end
  results
end
```

Photo by <a href="https://unsplash.com/@meganmarkham?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Megan Bucknall</a> on <a href="https://unsplash.com/photos/V9-i2ORDQLA?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>


---
layout: image-right
image: /images/valeriia-kogan-lXJbz0KjQI8-unsplash.jpg
---

## Crash Cleanup (Supervision) Demo

```ruby
CPU_COUNT = 4.freeze
def run_fib(input, pool, workers)
  send_work(input, pool)
  collect_results(input, pool, workers)
end
pool = make_pool
workers =  make_workers(pool, CPU_COUNT)

# without errors
input_list = [29, 28, 27, 26, 25]
answers = run_fib(input_list, pool, workers)
pp answers.count
pp answers
pp workers

# with errors
input_list = [24, 23, '21', '22', 20]
answers = run_fib(input_list, pool, workers)
pp answers.count
pp answers
pp workers
```

Photo by <a href="https://unsplash.com/@newyork_creator?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Valeriia Kogan</a> on <a href="https://unsplash.com/collections/gCqv31Rmk0s/ambulance?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>


---
layout: image-left
image: images/yasamine-june-wh9Cbrl9yGY-unsplash.jpg
---

# Supervised Web Server

## - Simple Web Server (using Webrick)

## - Load-Balancing Ractors (Worker Pool)

## - Web Supervision Code


Photo by <a href="https://unsplash.com/@yasamine?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Yasamine June</a> on <a href="https://unsplash.com/s/photos/service?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---

## Simple Web Server

```ruby
require 'webrick'

def respond(c, req)
  # process request
  path = req.path
  query = req.query
  body = req.body

  # log request
  puts req.inspect
  puts "=" * 150

  # respond to Connection
  c.print "HTTP/1.1 200\r\n"
  c.print "Content-Type: text/html\r\n"
  c.print "\r\n"
  c.print "<h1>Hello #{query['name'] || 'world'}</h1>"
  c.print "<h2>Your path: #{path}</h2>"
  c.print "<br>"
  c.print "<hr>"
  c.print "<br>"
  c.print req.inspect
end
```

---

## Load-Balancing Ractors (worker pool)

```ruby
Ractor.make_shareable(WEBrick::Config::HTTP) # freeze unsafe object
Ractor.make_shareable(WEBrick::LF); Ractor.make_shareable(WEBrick::CRLF)
Ractor.make_shareable(WEBrick::HTTPRequest::BODY_CONTAINABLE_METHODS)
Ractor.make_shareable(WEBrick::HTTPStatus::StatusMessage)
def make_pool = Ractor.new { loop { Ractor.yield(Ractor.receive, move: true) } }
def make_worker(pool)
  Ractor.new(pool) do |p|
    loop do
      web_socket = p.take  # capture input
      web_request = WEBrick::HTTPRequest.new( WEBrick::Config::HTTP.merge(RequestTimeout: nil) )
      web_request.parse(web_socket) # process input
      respond(web_socket, web_request) # build/send response webpage
      web_socket.close # drop the connection (so we can take a new request)
    end
  end
end
def make_workers(pool, cpu_count) = cpu_count.times.map { make_worker(pool) }
def make_listener(pool)
  Ractor.new(pool) do |p|
    server = TCPServer.new(8080)
    loop do
      conn, _ = server.accept
      p.send(conn, move: true)
    end
  end
end
```

---

## Web Supervision Code

```ruby
CPU_COUNT = 4.freeze

pool = make_pool
listener = make_listener(pool)
worker_list = make_workers(pool, CPU_COUNT)

loop do
  begin
    Ractor.select(listener, *worker_list)
  # Worker Supervision
  rescue Ractor::RemoteError => e
    worker = e.ractor  # capture crashed worker
    worker_list.delete(worker) # remove crashed worker
    worker_list << make_worker(pool) # restore a new worker
  end
end
```

Now check:
* `http://localhost:8080/`
* `http://localhost:8080/works?name=Bill`


---
layout: image-left
image: /images/brooke-cagle--uHVRvDr7pg-unsplash.jpg
---

# Questions 1 - Road Map

 (Dec 2023 - Ruby 3.3?) Ideally

The following are needed:

1. Stabilize API - especially for the select command (he’s looking for feedback) a new API Ractor::Selector. If you have a comment, it is great. https://github.com/ruby/ruby/pull/7371 - I'm especially thinking about whether "auto removing" is good or not (or should be options). https://github.com/ruby/ruby/pull/7371/files#diff-2be07f7941fed81f90e2947cdd9a91a5775d0c94335e8332b4805d264380b255R446
2. MaNy Project (using the M:N algorithm - he has a talk in Japanese) - ractors should scale to multiple thousands and still run (be scheduled) efficiently (currently effective scheduling is limited to about 300)
3. Efficient memory usage - at scale - especially for long running Ractors

He isn’t promising Dec 2023 release - but its his goal.

---

# Data Copying

Data Copying - I didn’t fully understand - but here is his answer (hopefully I got the question correct) - it can be slow, but no martialing is used

Some objects (object tree structures for example) takes long time to make them shareable. If objects are immutable (sharable), you don't need to make marshal data.
Copying complex objects is slow:
For example, `Node.new(Node.new(Node.new(....))))` it makes tree structure objects and even if they are frozen, to check they are all frozen, it needs to traverse the tree.
For example, `Node.new(.... in deep ..., Node.new("str") ...)` creates almost frozen, but 1 mutable (unshareable) string object. The tree is not a shareable object(s).
Unfortunately, ‘moving’ objects is also slow when complex - especially when deeply nested.

---

# Garbage collection

Garbage collection of Ractors - there is no way to explore unreferenced Ractors, but the will be garbage collected when they finish running.

  (1) no references to the ractor
AND
  (2) the ractor is already terminated


---
# Back-pressure

controlling message flows at high volume - forcing senders to slow down. (I don’t have Rafael’s email - can someone forward this to him)

He is interested, but not familiar with it an will consider it if I can send him enough information to consider a plan)

I think it will be difficult since OTP has 2 ways to send messages - send with reply and without reply - and if you require reply and slowly answer the reply that signals to the client they need to slowdown,  Currently, factors only have send w/o reply.

If someone knows / has an design link for OTP backpressuree Ill send it along.

I’ll add these answers to the talk and the blog.

If you have more questions - I’ll gladly forward the otherwise here is his email if you want to ask directly - sasada@gmail.com



## Resources

* [Exploring Ractors, Bill Tihen](https://btihen.dev/posts/ruby/ruby_3_x_ractor/) - https://btihen.dev
* [Ractor Author Docs - very helpful](https://docs.ruby-lang.org/en/3.1/ractor_md.html) - https://docs.ruby-lang.org/en/3.1/ractor_md.html
* [Ractor Author Demo - very helpful](https://www.youtube.com/watch?v=0kM7yFM6Dao) - https://www.youtube.com/watch?v=0kM7yFM6Dao
* [Ruby Ractor Docs - has all technical aspects](https://ruby-doc.org/core-3.1.1/Ractor.html) - https://ruby-doc.org/core-3.1.1/Ractor.html
* [Web Server Article - starter code](https://kirshatrov.com/posts/ractor-web-server-part-two/) - https://kirshatrov.com/posts/ractor-web-server-part-two/
* [Introduction to Ractors in Ruby 3](https://blog.kiprosh.com/ruby-3-introduction-to-ractors/)
* [Making *MaNy* threads on Ruby](https://rubykaigi.org/2022/presentations/ko1.html#day1) - [Video Introduction](https://youtu.be/G0LX53QJdBE)
* [Parallel testing with Ractors: putting CPUs to work](https://www.youtube.com/watch?v=bvFj6_dulSo)
* [Ruby Ractors...Ruh Roh Raggy! | Rubber Duck Dev Show June 15, 2022](https://www.youtube.com/watch?v=B9vRsMGec-I) - benchmark example - threads vs ractors (21min)

Photo by <a href="https://unsplash.com/fr/@brookecagle?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Brooke Cagle</a> on <a href="https://unsplash.com/s/photos/discussion?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>


---

# Appendix B: Faster Fibonacci Algorithms

## Efficient One-Line Fibonacci

https://stackoverflow.com/questions/6418524/fibonacci-one-liner

**Fibonacci using Enumeration**

```ruby
def fibonacci(n) = (0..n).inject([1,0]) { |(a,b), _| [b, a+b] }[0]

fibonacci(256)
```

**Fibonacci with Hash Memoization**

```ruby
fibonacci = Hash.new { |h,k| h[k] = k < 2 ? k : h[k-1] + h[k-2] }

fibonacci[256]

# memoization with a lambda
fibonacci = ->(num) { Hash.new { |h,k| h[k] = k < 2 ? k : h[k-1] + h[k-2] }[num] }

fibonacci.call(256)
```

---

# Furter Interest Efficient Fibonacci Algorithms

https://stackoverflow.com/questions/12178642/fibonacci-sequence-in-ruby-recursion


## Fibonacci Recursion with caching

```ruby
module Fib
  @@mem = {}
  def self.compute(index)
    return index if index <= 1

    @@mem[index] ||= compute(index-1) + compute(index-2)
  end
end

Fib.compute(256)
```

---

## Fibonacci Using Closures

```ruby
module Fib
  def self.compute(index)
    f = fibonacci
    index.times { f.call }
    f.call
  end

  def self.fibonacci
    first, second = 1, 0
    Proc.new {
      first, second = second, first + second
      first
    }
  end
end

Fib.compute(256)
```

---

# Fibonacci using Linearity

```ruby
module Fib
  def self.compute(index)
    first, second = 0, 1
    index.times do
      first, second = second, first + second
    end
    first
  end
end

Fib.compute(256)
```
