---
layout:     post
title:      SysV msgq vs Posix msgq
date:       2018-4-1
summary:    Learnt differences the hard way.
categories: sysv msgq message queue protocols posix msgq
---

In the [Whitefield](https://github.com/whitefield-framework/whitefield)
project, I needed to select an IPC mechanism which could,
Req#1. provide a way for a process to receive messages with specific type i.e. a message intended for the process based on some id.
Req#2. a event based mechanism for receiving messages

## Use of 'mtype' in sysv msgq
[Sysv message queue](https://linux.die.net/man/7/svipc) support req#1 in a very elegant manner. A message could be posted using a specific 'mtype' such that a receiver process could call the interface [msgrcv](https://linux.die.net/man/2/msgrcv) with that particular mtype to receive only those marked messages, all on the same queue.
![Alt text](images/sysv_msgq.png "SysV Message Queue")

Long story short, currently 
