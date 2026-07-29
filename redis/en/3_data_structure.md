# Redis Data Structures

This tutorial explains how Redis data types are represented internally and why their encodings affect performance and memory usage.

## Simple Dynamic String (SDS)

SDS stores the current length and allocated capacity alongside the byte buffer. This makes length checks constant-time and allows controlled growth without relying on C string termination.

## Lists

Redis has used several list representations, including linked lists, ziplists, quicklists, and listpacks. Compact encodings save memory for small collections; Redis can convert them as a collection grows.

## Hash Tables

Redis hash tables handle collisions with chaining. Rehashing expands or shrinks the table, and incremental rehashing spreads the work across normal commands instead of blocking on one large operation.

## Skip Lists

Sorted sets use skip lists together with a hash table. Multiple forward-pointer levels provide expected logarithmic search and insertion while keeping the implementation simpler than a balanced tree.

## Integer Sets

An integer set stores integers in the smallest suitable encoding. When a value does not fit, Redis upgrades the entire set to a wider encoding.
