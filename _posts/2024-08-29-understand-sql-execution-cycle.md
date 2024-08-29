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
</br>
<strong>General goal: PARSE ONCE AND EXECUTE MANY TIMES.</strong>

### 2. BIND PHARSE


### 3. EXECUTE PHASE


### 4. FETCH PHASE