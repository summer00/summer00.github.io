---
title: "《发布！》读书笔记"
date: 2020-11-10 12:00:00 +0800
categories: [book-note]
---

# Living in production

Software design as taught today is terribly incomplete. It only talks about what systems should do. It doesn't address the converse - what systems should not do.

Most software is designed for the development lab or the testers in the QA
department. Testing - even agile, pragmatic, automated testing - is not enough to prove that software is ready for the real world.

## The scope of the challenge

1. large active user counts
2. Uptime demands have increased
3. to build software fast that's cheap to build, good for users, and cheap to operate

# Stabilize your system

A robust system keeps processing transactions, even when transient impulses, persistent stresses, or component failures disrupt normal processing.

## Extending your life span

Testing makes problems visible so you can fix them. Following Murphy's Law, whatever you do not test against will happen.

The only way you catch bugs before they bite you in production is to run your own longevity tests.

If all else fails, production becomes your longevity testing environment by default. You'll definitely find the bugs there, but it's not a recipe for a happy life style.

## Failure modes

The original trigger and the way the crack spreads to the rest of the system, together with the result of the damage, are collectively called a failure mode. No matter what, your system will have a variety of failure modes. Denying
the inevitability of failures robs you of your power to control and contain
them.

## Stopping crack propagation

1. not blocking forever when all connections are checked out
2. using timeout in remote calls
3. multiple server groups
4. the more tightly coupled the architecture, the greater the chance this coding error can propagate

# Stability anti-patterns

1. integration points: Every single one of those feeds presents a stability risk. Every socket, process, pipe, or remote procedure call can and will hang.
   1. socket-based protocols: Network failures can hit you in two ways: fast(connection refused) or slow(dropped ACK).

# Words

1. failover
2. presence
3. deduce
4. portion
5. wary
6. peculiar
7. ounce
8. leverage
9. impulse
10. stubbing
11. jettison
12. terminology
13. propagating
14. continuously
15. boundaries
16. chaotic
17. scenario
18. hammered
19. ramp
20. paranoid
21. probe
22. torn

# Sentence

1. A postmortem is like a murder mystery. You have a set of clues. Some are reliable, such as server log copied from the time of the outage. Some are unreliable, such as statements from people about what they saw. The postmortem can actually be harder to solve than a murder, because the body goes away.
2. In other words, once you know where to look, it's simple to make a test that finds it.
3.
