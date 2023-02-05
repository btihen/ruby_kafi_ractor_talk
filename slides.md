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

# Ractors

Ruby Parallel Computing (with Ruby Actors)

Bill Tihen

---
layout: image-right
image: /images/british-library-Gw_UOoFk4Wk-unsplash.jpg
---

# Outline

* Defined
* Comparison
* Components
* Life-cycle
* Communications
* Design Examples
* Simple Ractor Webserver

Photo by <a href="https://unsplash.com/@britishlibrary?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">British Library</a> on <a href="https://unsplash.com/s/photos/map?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/kyle-head-p6rNTdAPbuk-unsplash.jpg
---

# Defined: **Ruby Actors** (Ractors)

## Ruby’s Actor-like concurrent abstraction

Parallel computing with object safety using an actor like model using message passing instead of object sharing.

This is very helpful with heavy CPU tasks that can be split into sensible independent tasks.

**NOTE**:
* _Not all tasks are faster using Ractors._
* Ractors are still considered experimental, but seem very stable.

<small>Photo by <a href="https://unsplash.com/@kyleunderscorehead?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Kyle Head</a> on <a href="https://unsplash.com/photos/p6rNTdAPbuk?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

---
layout: image-right
image: /images/joe-neric-AkEMVbdEmUQ-unsplash.jpg
---

# Comparison

Using Fibonacci recursion:

| **Code Method** | **Time (Sec)** | **Notes**      |
| ----------------| --------------:|:---------------|
| **Ractors Fork-Join** | **2.3**  | _All CPUs (7)_ |
| **Ractor Pool**       | **7.9**  | _4 CPUs_       |
| Multi-Threaded        | 17.3     | 4 Threads      |
| Single Thread         | 17.4     | Main Thread    |

**Note**:
* not all algorithms are faster with Ractors.
* only the single thread returns sequential results

Photo by <a href="https://unsplash.com/@jneric?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Joe Neric</a> on <a href="https://unsplash.com/collections/2337461/comparison-meeting?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/01_ractor_components.png
---

# Components

* **in-port** - accepts messages when open
* **queue** (Mailbox) - Infinite (limited by RAM)
* **data** - isolated data to be used in execution
* **code** - code that executes
* **out-port** - open when the Ractor has results to share (queue length of 1)

---
layout: image-right
image: /images/evie-s-zn4Pl32WgWM-unsplash.jpg
---

# Lifecycle

* A Ractor is basically alive as long as a port (in or out) is open.
* While a port is open the Ractor is **blocking** (waiting) - the incoming port needs to be in an infinite loop - otherwise it closes after the first message.
* The Queue is by default first in first out.  The queue length is infinite (limited by memory).
* The outgoing port is open as soon as the Ractor as finished processing a message (the outgoing )
* A Ractor processes a message when the outgoing port is closed (nothing queued) and two there is a message to computing in the incoming port.
* Ractors 'die' if the experience an exception (and kill all downstream Ractors too)


Photo by <a href="https://unsplash.com/@evieshaffer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Evie S.</a> on <a href="https://unsplash.com/images/people/life?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/ishan-seefromthesky-4xmgrNUbyNA-unsplash.jpg
---

## Short lived (single use) Ractor

```ruby
integer = 1

r1 = Ractor.new(integer, name: 'r1') do |i|
  puts "Executing #{i} + 2"
  i + 2
end

r1.inspect # => "#<Ractor:#1 r1 (irb):8 terminated>"

r1.send(1) # `send': The incoming-port is already closed (Ractor::ClosedError)

r1.take # outport is open, waiting for the result to be taken
# => 3

r1.take # `take': The outgoing-port is already closed (Ractor::ClosedError)
```

Photo by <a href="https://unsplash.com/@seefromthesky?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Ishan @seefromthesky</a> on <a href="https://unsplash.com/s/photos/single-use?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>


---
layout: image-right
image: /images/matt-seymour-8X2siC3gSj4-unsplash.jpg
---

## Long lived (multi-use) Ractor

```ruby
r1 = Ractor.new(name: 'r1') do
  loop do # internal Ractor infinite loop allows multi-use
    input = Ractor.receive # receive pulls from the inbox
    result = input + 2
    puts "Executed - result will be: #{result}"
    Ractor.yield(result)
  end
end
r1.inspect # waiting for input => #<Ractor :#8 r1 blocking>
r1.send(1) # puts is seen immediately (since outport is open)

# we can add messages to the incoming queue
r1.send(2) # doesn't execute since output has a result
r1.take # get first result & process next message
# => 3

r1.take # get result, nothing in the queue, nothing executes
# => 4
r1.take # closed outport error since nothing is in its queue
```

Photo by <a href="https://unsplash.com/@mattseymour?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Matt Seymour</a> on <a href="https://unsplash.com/s/photos/circular?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/yogendra-singh-BxHnbYyNfTg-unsplash.jpg
---

## Sudden Death (Exceptions)

```ruby
integer = 'a'

# automatically queues & processes the message
r1 = Ractor.new(integer, name: 'doomed') { |i| i + 2 }
#<Thread:0x0000000104373ae8 run> terminated with exception (report_on_exception is true):
# (irb):16:in `+': no implicit conversion of Integer into String (TypeError) from (irb):16:in `block in <top (required)>'
# => #<Ractor:#4 doomed (irb):16 running>

# because the Ractor was build and immediately tried to process
# the return messages are in an odd order.
# actually checking the status confirms it died
r1.inspect
# => "#<Ractor:#9 r1 (irb):58 terminated>"
```

**Note**: if you are using a pipeline - all downstream Ractors crash too.  If you are using supervision - you will need to remove and recreate all downstream Ractors.

Photo by <a href="https://unsplash.com/@yogendras31?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Yogendra Singh</a> on <a href="https://unsplash.com/s/photos/problem?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-right
image: /images/guilherme-stecanella-SZ80v2lmhSY-unsplash.jpg
---

# Communications

```
Ports/Commands:   +---------------------------------------+
    r.send(obj) ->*->[incoming queue]  Ractor.yield(obj)->*-> r.take
                  |         v                 ^           |
Ractor r          |   Ractor.receive -> Code Execution    |
                  +---------------------------------------+

Push Pipeline:    +----------+    r2: +-------------------+
              r1: | r2.send->|------->*-> Ractor.receive  *
                  +----------+        +-------------------+

Pull Pipeline:    +-------------------+     r2: +---------+
              r1: * Ractor.yield(obj) *-------->- r1.take |
                  +-------------------+         +---------+

Pull from Pool:   +--------------------+
              r1: * Ractor.yield(obj)->*--+
                  +--------------------+  |
                                          +-> Ractor.select(r1, r2)
                  +--------------------+  |   (Waiting on Ractors)
              r2: * Ractor.yield(obj)->*-=+
                  +--------------------+
```

Photo by <a href="https://unsplash.com/@guilhermestecanella?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Guilherme Stecanella</a> on <a href="https://unsplash.com/s/photos/message?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/mohammadreza-alidoost-0rUp9vgyEYo-unsplash.jpg
---

## Design Examples

* Ring
* Fork-Join
* Pipeline
* Worker-Pool
* Supervision

Photo by <a href="https://unsplash.com/@mralidoost?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Mohammadreza alidoost</a> on <a href="https://unsplash.com/s/photos/display?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>


---
layout: image-right
image: /images/aaron-jones-IJbfutoo7_U-unsplash.jpg
---

# Pipeline

```ruby
def increment(n) = n + 1
def doubler(n) = n * 2

r1 = Ractor.new do
  loop do
    input_r1 = Ractor.receive
    incremented = increment(input_r1)
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
r1.send(1); r2.take
r1.send(4); r2.take
```

Photo by <a href="https://unsplash.com/@ajonesyyyyy?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Aaron Jones</a> on <a href="https://unsplash.com/s/photos/pipeline?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/matheo-jbt-HLhvZ9HRAwo-unsplash.jpg
---

# Ring (Recursion)

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
layout: image-right
image: /images/mae-mu-Pvclb-iHHYY-unsplash.jpg
---

# Fork-Join (pool consumption)

**Very FAST** - Uses as many resources as allowed and then gone

```ruby
MAX_FIB_NUM = 39.freeze
CALC_LIST = MAX_FIB_NUM.downto(1).map { |i| i }.freeze
def fib(n) = n < 2 ? 1 : fib(n-2) + fib(n-1)

pool = CALC_LIST.map do |val| # Ractor per value wanted
  Ractor.new(val) { |n| [n, fib(n)] }
end

results = []
t1 = Time.now
until pool.empty? # Collect results until work completed
  dead_worker, value = Ractor.select(*pool) # get completed work
  p answer: value # so we see something while working
  pool.delete dead_worker # remove terminated Ractors
  results << value # collect results
end
puts "Fork Join - full cpu usage - duration #{Time.now - t1}"
p results: results
```

Photo by <a href="https://unsplash.com/de/@picoftasty?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Mae Mu</a> on <a href="https://unsplash.com/s/photos/fork?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/redd-f-yinfkjyiptY-unsplash.jpg
---

## Worker Pool (Load Balancer)


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
pp results.map { |r| r.last }.sort
```

Photo by <a href="https://unsplash.com/@raddfilms?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Redd F</a> on <a href="https://unsplash.com/s/photos/workers?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>


---
layout: image-right
image: /images/urban-gyllstrom-vxYVvoeuFQw-unsplash.jpg
---

## Supervisor

```ruby
MAX_CPUS = 4.freeze; MAX_FIB_NUM = 39.freeze
def fib(n) = n < 2 ? 1 : fib(n-2) + fib(n-1)
def make_pool = Ractor.new { loop { Ractor.yield(Ractor.receive) } }
def make_worker(pool)
  Ractor.new(pool) do |p|
    loop { input = p.take; fib_num = fib(input)
      Ractor.yield( [input, fib_num] )
    }
  end
end
def make_workers(pool, count) = (1..count).map { |i| make_worker(pool) }
def send_work(input, pool) = Array(input).each { |i| pool.send(i) }
def collect_results(input, pool, workers)
  results = []
  input.count.times do
    begin
      worker, answer = Ractor.select(*workers)
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

Photo by <a href="https://unsplash.com/@gyllstrom_photo?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Urban Gyllström</a> on <a href="https://unsplash.com/s/photos/supervisor?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/trend-SmY0VRc1lDU-unsplash.jpg
---

## Usage

```ruby
def run_fib(input, pool, workers)
  send_work(input, pool)
  collect_results(input, pool, workers)
end

pool = make_pool
workers =  make_workers(pool, 4)

input_list = [29, 28, 27, 26, 25]
answers = run_fib(input_list, pool, workers)
pp answers.count
pp answers
pp workers

input_list = [24, 23, '21', '22', 20]
answers = run_fib(input_list, pool, workers)
pp answers.count
pp answers
pp workers
```

Photo by <a href="https://unsplash.com/@trend_io?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Trend</a> on <a href="https://unsplash.com/s/photos/pool?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-left
image: /images/hannes-richter-qp8Prc0Dy9I-unsplash.jpg
---

## Simple Web Responder

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

Photo by <a href="https://unsplash.com/@weristhari?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Hannes Richter</a> on <a href="https://unsplash.com/s/photos/answer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---

## Web Ractors

```ruby
Ractor.make_shareable(WEBrick::Config::HTTP) # freeze unsafe object
Ractor.make_shareable(WEBrick::LF); Ractor.make_shareable(WEBrick::CRLF)
Ractor.make_shareable(WEBrick::HTTPRequest::BODY_CONTAINABLE_METHODS)
Ractor.make_shareable(WEBrick::HTTPStatus::StatusMessage)
def make_pool = Ractor.new { loop { Ractor.yield(Ractor.receive, move: true) } }
def make_worker(pool)
  Ractor.new(pool) do |p|
    loop do
      s = p.take  # capture input
      req = WEBrick::HTTPRequest.new( WEBrick::Config::HTTP.merge(RequestTimeout: nil) )
      req.parse(s) # process input
      respond(s, req) # build/send response webpage
      s.close # drop the connection (so we can take a new request)
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
layout: image-right
image: images/yasamine-june-wh9Cbrl9yGY-unsplash.jpg
---

## Web Server (with Supervision)

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


Photo by <a href="https://unsplash.com/@yasamine?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Yasamine June</a> on <a href="https://unsplash.com/s/photos/service?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---
layout: image-right
image: /images/brooke-cagle--uHVRvDr7pg-unsplash.jpg
---

# Discussion / Questions

## Resources

* [Ractor Author Docs - very helpful](https://docs.ruby-lang.org/en/3.1/ractor_md.html) - https://docs.ruby-lang.org/en/3.1/ractor_md.html
* [Ractor Author Demo - very helpful](https://www.youtube.com/watch?v=0kM7yFM6Dao) - https://www.youtube.com/watch?v=0kM7yFM6Dao
* [Ruby Ractor Docs - has all technical aspects](https://ruby-doc.org/core-3.1.1/Ractor.html) - https://ruby-doc.org/core-3.1.1/Ractor.html
* [Web Server Article - starter code](https://kirshatrov.com/posts/ractor-web-server-part-two/) - https://kirshatrov.com/posts/ractor-web-server-part-two/

Photo by <a href="https://unsplash.com/fr/@brookecagle?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Brooke Cagle</a> on <a href="https://unsplash.com/s/photos/discussion?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
