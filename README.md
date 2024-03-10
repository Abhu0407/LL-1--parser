# LL-1--parser

An LL(1) parser made in Python.

It print out the production rules befor and after remove left recursion and factorial, first, follow, parsing table and chack input string 

# ussage
Compile the code with g++ and run with arguments

Grammar file contains all the productions for the grammar.

  Each line contains a production for the grammar in the format LHS->RHS
  All upper case alphabets and with {'} are non-terminals or variables.
  Hash sign is used as end character.
  Any other character is taken as a terminal for the grammar

Need pakages are 
  tabulate libraries in python
