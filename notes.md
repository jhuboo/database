# SQLITE

### Architecture

Tokenizer -> Parser -> Code Generator -> VM -> B-Tree -> Pager -> OS Interface

A query goes through a chanin of components in order to retrieve or modify data. The ***front-end*** consists of
- Tokenizer
- Parser
- Code Generator

The input to the front-end is an SQL query. The output is sqlite VM bytecode that operates on the database.

The ***back-end*** consists of
- VM
- B-Tree
- Pager
- OS Interface


### REPL (Step 1)

Sqlite starts with a read-execute-print loop (REPL) when started in terminal. Therefore, our main function will have an infinte loop that prints the prompt, gets a line of input, which is processes.


### Virtual Machine (Step 2)

The front-end of Sqlite is an SQL compiler that parses a string and outputs an internal representation called bytecode. The bytecode is passed to the VM, which executes it. Breaking things into these 2 steps brings some advantages:
- Reduces complexity
- Allows comiling common queries once & caching the bytecode for improved performance


### In-Memory, Append-Only, Single-Table Database (Step 3)

We'll put some limitations on our database to make it as simple as possible. So it will:
- support 2 operations: insert a row & print all rows
- reside only in memory (no persistence to disk)
- support a single, hard-coded table

SQLite uses a B-Tree for fast lookups, inserts & deletes. Our data structure will group rows into pages, but will arrange them like a tree instead of an array for simplicity.
- Store rows in blocks of memory called pages
- Each page stores as many rows as it can fit
- Rows are serialized into a compact representation with each page
- Pages are only allocated as needed
- Keep a fixed-sized array of pointers to pages

Our pages size is made to be 4 KB so that it's the same size as a page used in the virtual memory systems of most computer architectures. This means that one page in our database corresponds to one page used by the OS. The OS will move pages in & out of memory as whole units instead of breaking them up.
