---
layout: post
title: Understanding SQL execution cycle
description: The lifecycle of a SQL execution
date: 2024-08-29 20:30:44 -0800
tags: SQL
---

A normal SQL statement go through below phase before producing results.

<img src="/img/sql/sql-execution-lifecycle.png">

### 1. PARSE PHASE

Parsing stage will tell what does the statement mean and how best to run it.
There are two type of parses:
<br>
**Sort Parse**

<ul>
    <li>Check syntax, semantics and permissions.</li>
    <li>Search the library cache for matching statement.</li>
</ul>

**note:** If it finds the statement in library cache then it will not do the hard parse.

**Hard Parse**

<ul>
    <li>Also called optimization step.</li>
    <li>Neccessary on first execution.</li>
    <li>Take parse locks on referenced object.</li>
    <li>Develop a decision tree of possible plans.</li>
    <li>Select the lowest cost plan for execution.</li>
    <li>Store the plan in the library cache of the shared pool.</li>
</ul>

Important: Hard parse takes time.
<br>
<strong>General goal: PARSE ONCE AND EXECUTE MANY TIMES.</strong>

### 2. BIND PHARSE

This phase explain any variables to be used by SQL

<ul>
    <li>Not necessary if statement uses no variables.</li>
    <li>Retrieves values for variables from the user process.</li>
    <li>Usually occurs after parsing.</li>
    <li>Make possible re-use of existing optimized code</li>
    <li>Exception - first execution will "peek" the binds.</li>
    <li>Different binds may warrant different execution plans - can be tackled by adaptive cursor sharing.</li>
</ul>

### 3. EXECUTE PHASE

This phase works in buffer cache:
<ul>
    <li>Most of the work for DML occurs here.</li>
    <li>Shared locks on tables and locks on rows are taken.</li>
    <li>Generate redo - write change vectors to the log buffer.</li>
    <li>Write changes to the buffer cache by generating Undu OR Writing new values to the table and index blocks.</li>
</ul>

Data I/O during Execution phase can be:
<br>
**Logical I/O:**
<ul>
    <li>Search the buffer cache for necessary data blocks and get it.</li>
    <li>Latch is light weight lock which is required to access Logical I/O. So logical I/O is not free. REMEMBER that.</li>
    <li>Latches and Locks reduce scalability.</li>
</ul>

**Physical I/O:**
<ul>
    <li>Copy blocks of table, index and undo segments into cache cause Physical I/O and it is costly operation.</li>
</ul>

### 4. FETCH PHASE

This phase prepare and return the result set.
<ul>
    <li>Most of the work for SELECT occurs here.</li>
    <li>I/O from disk into memory if required.</li>
    <li>Search the buffer cache for required data blocks or do direct/indirect reads.
    ( <b>Direct Reads:</b>  Copy Blocks into PGA. It is private area to user so no latch/lock contention - <b>Indirect Reads:</b> Copy Blocks into Buffer Cache. It is a shared area so latches will come into play).</li>
    <li>Performs join, aggregation (group by), projections, sorts (order by)</li>
    <li>Return the result set to the user process.</li>
</ul>