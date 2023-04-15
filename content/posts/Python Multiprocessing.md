---
title: "Python Multiprocessing"
date: 2022-02-13T16:52:25+03:00
draft: true
---

## Multiprocessing

Due to the Global Interpreter Lock in the implementation of CPython, real concurrent multi-threading in a single process can not be achieved. An alternative way of leveraging the full power of multi-core CPUs is to use Multiprocessing, which supports spawning processes using an API similar to the threading module.

## Pool

A Pool is a set of worker processes that can be utilized with load balancing. It has methods that allow tasks to be offloaded to the worker processes in a few different ways.

## map & imap

The map/imap function sends elements from iterable objects to a specified function and returns the results as an iterable object.

### Differences between map and imap

How does the Pool.imap() function compare to the Pool.map() for issuing tasks to the process pool?

Both the imap() and map() may be used to issue tasks that call a function to all items in an iterable via the process pool.

The following summarizes the key differences between these two functions:

The imap() function issues one task at a time to the process pool, the map() function issues all tasks at once to the pool.
The imap() function blocks until each task is complete when iterating the return values, the map() function blocks until all tasks are complete when iterating return values.
The imap() function should be used for issuing tasks one-by-one and processing the results for tasks in order once they are available.

The map() function should be used for issuing all tasks at once and processing results in order only once all issued tasks have been completed.
