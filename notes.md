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
