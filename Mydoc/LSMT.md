Log Structured Merge Tree
===========

## Idea
Designed to provide better performance than B+ tree on write operations. It does
this via convert random updates to sequential updates.

other:
log update-in-place is slow (evolved in random IO)