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


### Testing (Step 4)

Since we can now insert rows into the database, and print all the rows, let's start testing. We'll use `rspec`.


### Persistence to Disk (Step 5)

Like SQLite, we're going to persist records by saving the entire database to a file. We have already set up to do that by serializing rows into page-size memory blocks. We can now write those blocks of memory to a file to add persistence. We read them back into memory the next time the program starts up.

We build an abstraction called the pager to make things easier. When we ask the pager for page no x, it gives us back a block of memory. It first look in its cache. On a cache miss, it copies data from disk into memory (by reading the database file).

The Pager access the page cache and the file. The Table object makes request for pages through the pager.


### The Cursor Abstraction (Step 6)

Goals here are to refactor a bit to start implementing the B-Tree, and to add a Cursor object which represents a location in the table. The cursor can:
- Create a cursor at the beginning of the file
- Create a cursor at the end of the file
- Access the row the cursor is pointing to
- Advance the cursor to the next row

Behaviours to implement for the cursor:
- Delete the row pointed to by the cursor
- Modify the row pointed by the cursor
- Search a table for a given ID, and create a cursor pointing to that row with that ID


### B-Tree (Step 7)

SQLite uses a B-Tree data structure to represent both tables and indexes. 

Why is a tree a good data structure for a database?
- Searching for a particular value is fast in Log(n)
- Inserting / deleting a value that you've already foudn is fast (constant-ish time to rebalance)
- Traversing a range of values is fast (unlike a hash map)

A B-Tree is different from a binary tree. Unlike a binary tree, each node in a B-Tree can have more than 2 children. Each node can have up to `m` children, where `m` is called the tree's order. To keep the tree mostly balanced, the nodes have to be at least m/2 children (rounded up).

Exceptions:
- Leaf nodes have 0 children
- The root node can have fewer than m children but must have at least 2
- If the root node is a leaf node (the only node, it still has 0 children


There are B-tree (Bee Trees) and B+trees (Bee Plus Trees).

| 				| B-Tree 	| B+ Trees 		|
| ------------- 		| ------ 	| -------- 		|
| Used to Store 		| Indexes 	| Tables   		|
| Internal nodes store keys 	| Yes 		| Yes			|
| Internal nodes store values 	| Yes 		| No			|
| No of children per node 	| Less 		| More			|
| Internal nodes vs leaf nodes 	| Same structure | Different structure	|

We'll be using B+Trees (btree)

Nodes with children are called "internal" nodes. Internal nodes and leaf nodes are structured differently.

| For an order-m tree		| Internal Node			| Leaf Node	|
| ------------- 		| ------ 			| -------- 	|
| Stores	 		| Keys & ptrs to children 	| Keys & values |
| Number of ptrs		| no of keys + 1 		| none		|
| Number of values		| none		 		| no of keys	|
| key purpose			| used for routing 		| paired with value |
| Stores values? 		| No 				| Yes 		|

A B+Tree grows when elements are inserted into it. To keep things simple, the tree will be order 3, which means:
- Up to 3 children per internal node
- up to 2 keys per internal node
- at least 2 children per internal node
- at least 1 key per internal node

Wikiepdia Link to [B+Trees](https://en.wikipedia.org/wiki/B%2B_tree)


### B-Tree Leaf Node Format (Step 8)

We're changing the format of our table from an unsorted array of rows to a B-Tree. The reasons for switching to a tree structure are:
- With the current format, each page stores only rows (no metadata) so it's pretty space efficient. Insertion is also fast because we just append to the end. However, finding a particular row can only be done by scanning the entire table. If we want to delete a row, we have to fill in the hole by moving every row that comes after it.
- If we stored the table as an array, but kept rows sorted by id, we could use binary search to find a particular id. However, insertion would be slow because we would have to move a lot of rows to make space.
- Instead with a tree structure, each row in the table can contain a varialbe no of rows, so we have to stome information in each node to keep track of how many rows it contains. Plus there is the storage overhead of all the internal nodes which don't store any rows. In exchange for a larger database, we get fast insertion, deletion and lookup.


### Binary Search and Duplicate Keys (Step 9)

We will now store keys in a sorted order, and detect and reject duplicate keys.


### Splitting a Leaf Node (Step 10)

We need to modify our B+Tree since it only has one node right now. To fix that, we need some code to split a leaf node in two. After that, we need to create an internal node to serve as a parent for the two leaf nodes.

***Splitting Algorithm***
If there is no space on the leaf node, we would split the existing entries residing there an the new one (being inserted) into two equal halves: lower, and upper halves. (Keys on the upper half are strictly greater than those on the lower half.) We allocate a new leaf node, and move the upper half into the new node.
