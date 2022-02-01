# SQLite Clone from stratch in C

Author: Anvesh G. Jhuboo
Date: 31 Jan 2022

The goal of this exercise is to build a simple database.

- [x] Set up a REPL
- [x] Add a simple SQL compiler & Virtual Machine
- [x] Add In-Memory, Append-Only, Single-Table Database
- [x] Add Tests (for Bugs)
- [x] Persistence to Disk
- [x] The Cursor Abstraction
- [x] B-Trees
- [x] B-Tree Leaf Node Format
- [ ] Binary Search and Duplicate Keys

### Usage so far

```
~ ./db database.db
db > insert 1 math name@email.com
Executed.
db > insert 2 physics foo@bar.org
Executed.
db > .exit

~ ./db database.db
db > select
(1, math, name@email.com)
(2, physics, foo@bar.com)
Executed.

~ ./db database.db
db > .constants
db > .btree

db > insert foo bar 3
Syntax Error. Could not parse statement.
db > .exit
~
```

### Note

Use vim as a hex editor to view the binary database file in hex format.
To do so:
```
~ vim my_database.db

Inside the binary file opened by vim, do the following command:
:%!xxd
```
