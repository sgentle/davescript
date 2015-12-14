Davescript
==========

Davescript is a simple stack-based language. The syntax is as follows:

1. "Dave" increments the value on the top of the stack by 1
2. "!" adds a 0 on top of the stack
3. "\n" (or EOF) executes the opcode reprented by the value on top of the stack
4. As a special syntactic shorthand, "Daaaaaaave" with x "a"s increments by x
5. Any other characters and malformed statements are ignored


Davescript interpreter
----------------------

First we set up an empty stack. We don't need to initialise it to 0 because
the Dave operation automatically adds a 0 if the stack is empty.

    stack = []

Our operations are chosen by the opcode on the top of the stack, so we just
put them in a big array. The current set of operations are probably not
sufficient for anything useful.

    OPERATIONS = [
      -> # NOOP
      -> console.log (String.fromCharCode c while (c = stack.pop()) > 0).reverse().join '' # PRINT
      -> stack.push stack.pop() + stack.pop() #ADD
      -> stack.push stack.pop() - stack.pop() #SUBTRACT
      -> stack.push stack.pop() * stack.pop() #MULTIPLY
      -> stack.push stack.pop() / stack.pop() #DIVIDE
      -> count = stack.pop(); op = stack.pop(); OPERATIONS[op]() while count-- #LOOP
    ]


Interpret a "Dave" statement. Our interpreters take the whole string and
return how many extra characters they consumed. In the case of the Dave
statement, that's the same as the number to increment the stack by.

    interpretDave = (str, i) ->
      n = 0
      n++ while str[i+n] is 'a'
      return 0 unless str[i+n] is 'v' and str[i+n+1] is 'e'
      stack.push 0 if stack.length is 0
      stack[stack.length-1] += n
      return n

Interpret a "!"

    interpretBang = (str, i) ->
      stack.push 0
      0

Interpret a line. The newlines are stripped out by this point so we can just
assume that we execute at the end of each line.

    interpretLine = (line) ->
      i = 0
      while (c = line[i++])?
        switch c
          when 'D' then i += interpretDave(line, i)
          when '!' then i += interpretBang(line, i)
      OPERATIONS[stack.pop() or 0]()

Now we hook up the interpreter to the input stream. This buffers each line in
memory, which may not be okay if you have Davescript programs that are many
megabytes per line.

    stream = if process.argv[2] then require('fs').createReadStream(process.argv[2]) else process.stdin

    rl = require('readline').createInterface input: stream
    rl.on 'line', interpretLine
