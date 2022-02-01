# SQLite Clone from stratch in C

Author: Anvesh G. Jhuboo
Date: 31 Jan 2022

The goal of this exercise is to build a simple database.

- [x] Set up a REPL
- [x] Add a simple SQL compiler & Virtual Machine
- [x] Add In-Memory, Append-Only, Single-Table Database
- [ ] Add Tests (for Bugs)

### Usage so far

```
~ ./db
db > insert 1 math name@email.com
Executed.
db > insert 2 physics foo@bar.org
Executed.
db > select
(1, math, name@email.com)
(2, physics, foo@bar.com)
Executed.
db > insert foo bar 3
Syntax Error. Could not parse statement.
db > .exit
~
```
