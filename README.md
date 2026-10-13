# things i figured out

and a few things i very much have not.

---

I like the part before you know what you're doing.

New codebase.  
New machine.  
New protocol.  
Something is broken and nobody knows why.

There is a short period where the system is basically a black box.

Then you poke it enough times, break the right things, read enough logs,
draw enough terrible diagrams, and suddenly it has a shape.

This is a collection of a few times that happened.

---

## 01 — "it doesn't work" is not a bug report

One thing I've learned from debugging real machines:

**the symptom is usually nowhere near the cause.**

A system doesn't boot.

Okay.

That could mean firmware.
Memory.
A bad image.
Configuration.
Hardware.
A peripheral.
Something in the boot chain.
Something that worked yesterday and absolutely refuses to explain
why it changed its mind today.

At first, debugging felt like searching.

Now I think it's closer to repeatedly asking:

> what would have to be true for this exact behavior to happen?

Then removing possibilities one by one.

Check the logs.

Change one variable.

Compare against a known-good system.

Swap a component.

Reflash.

Boot again.

Be wrong.

Update the model.

Repeat.

I used to think good debugging meant knowing a lot of answers.

I increasingly think it means getting unusually good at asking
**the next useful question.**

That has probably been my favorite thing to figure out.

---

## 02 — I wanted to know what happens when the software itself stops behaving

So I bought an RP2040 and started building a fault monitor.

No framework doing the interesting parts for me.

Just C, timers, interrupts, registers, UART, and a tiny computer
that was extremely willing to let me make bad decisions.

I ended up building:

- interrupt-driven UART with an RX ring buffer
- execution/heartbeat monitoring
- fault injection
- a circular diagnostic log
- watchdog/reset diagnostics
- recovery and safe-state behavior
- ADC monitoring
- a CLI over serial
- Python host-side tests that intentionally try to make it fail

The obvious goal was:

> detect the fault.

The more interesting questions became:

> How do you know something has stopped progressing?

> How much state should survive a failure?

> When should a system recover automatically?

> When is recovery actually more dangerous than stopping?

> How do you test failure paths without waiting for reality to produce them?

At some point I realized I wasn't really building a "fault monitor."

I was building increasingly elaborate ways to ask a tiny computer:

**are you actually okay?**

[code →](https://github.com/Vaishnavi-Patil08/rp2040-fault-monitor)

---

## 03 — 169 tests happened by accident

I built a backend called Sentinel.

The original idea was straightforward:
multiple users, private documents, semantic retrieval.

Then I started asking annoying questions.

What if user A knows user B's document ID?

What if the cache has an answer the user can no longer access?

What if Redis disappears?

What if the vector database disappears?

What if the LLM fails after retrieval succeeds?

What if two tenants upload very similar content?

What exactly does "authorization before retrieval" mean in every path?

The project turned into a multi-tenant backend with:

- JWT authentication
- tenant/document-level authorization
- Redis caching and identity state
- Qdrant vector retrieval
- payload filtering
- rate limiting
- explicit dependency-failure behavior
- 169 unit and integration tests

The 169 tests were not the plan.

I just kept finding new ways the system could technically "work"
while still being wrong.

That distinction now lives permanently in my head:

> **working is not the same thing as being correct.**

[code →](https://github.com/Vaishnavi-Patil08/sentinel)

---

## 04 — copy-on-write made me suspicious of "simple" operations

`fork()` looks very innocent from user space.

Then I implemented Copy-on-Write in xv6.

Suddenly one apparently simple operation involved:

page tables,
PTE flags,
trap handling,
physical-page reference counts,
synchronization,
page faults,
memory ownership,
and deciding exactly when a page should stop being shared.

That project changed how I look at abstractions.

The nice API is real.

But underneath it, somebody had to make hundreds of tiny rules agree
with each other.

And when one of those rules is wrong, the abstraction leaks.

Badly.

I also extended xv6 with process-memory system calls and doubly indirect
file-system blocks.

Mostly, though, I came away with a new reflex:

when something looks magically simple from the top,
I want to know what ugly machinery is hiding underneath.

[code →](https://github.com/Vaishnavi-Patil08/COW)

---

## 05 — I learned that automation gets interesting when it has to survive itself

I volunteered with an organization that had a very normal problem:

they needed information about a large shared-drive hierarchy.

The first version of the solution was basically:

> walk the folders and write things to a sheet.

Then reality showed up.

APIs paginate.

Execution time is limited.

Calls fail.

Directories nest.

Long jobs get killed halfway through.

Starting over every time is technically a solution,
just not a very good one.

So I turned it into a resumable traversal:

- persistent queue state
- pagination
- retries
- runtime guards
- checkpointing
- continuation across executions

That project taught me something surprisingly general:

> if a process can take a long time, failure is not an edge case.

Failure is part of the control flow.

---

# things I haven't figured out

This is probably the more interesting section.

### When does knowledge turn into intuition?

Two people can look at the same logs.

One starts randomly checking things.

The other says:

> "look there."

And they're right.

We call that experience.

But what exactly got compressed?

Which failed attempts mattered?

Which patterns became so familiar that the person stopped consciously
recognizing them?

How much expert knowledge exists in the sequence of things someone
checks rather than the final answer they produce?

I don't know.

I think about it a lot.

---

### How much of engineering is knowing what *not* to investigate?

Beginners often explore everything.

Experts seem to delete entire branches of the search tree almost instantly.

That feels important.

Maybe expertise isn't just accumulating knowledge.

Maybe it's also accumulating extremely good reasons to ignore things.

---

### When should software recover by itself?

Automatic recovery sounds obviously good.

Until the system is confidently recovering into the wrong state.

I still don't have a satisfying general rule for the boundary between:

> "fix yourself"

and

> "stop and make a human look at this."

---

### What makes a good abstraction?

Not "what is abstraction?"

I mean:

how much complexity can you hide before hiding it becomes dangerous?

The best abstractions make difficult systems usable.

The worst ones make failures incomprehensible.

I haven't figured out where that line is.

---

### Why is figuring something out so addictive?

No technical theory here.

I just really like the moment when:

> "I have absolutely no idea what's happening"

turns into:

> "wait."

then:

> "WAIT."

then three hours disappear.

Still investigating.

---

# the current hypothesis

I used to think I liked computer science because I liked building things.

That's true.

But I think the deeper reason is that computers let you ask
ridiculously precise questions about cause and effect.

Change this bit.

Send this packet.

Move this pointer.

Kill this dependency.

Delay this response.

Break this assumption.

See what happens.

And if your mental model is wrong, the machine is wonderfully
uninterested in protecting your feelings.

It just gives you the wrong answer.

So you update the model and try again.

I love that.

---

If you found this because I sent you a job application:

hi.

I am probably applying for something I am slightly underqualified for.

That is somewhat intentional.

New stack, new domain, new problem — **I'll figure it out.**
