# LOGBOOK 6

## Setup

In order to set up our computers for this lab's execution, we had to go through 3 steps:

- Execute this line of code, which disables address space randomization, which means that our address in the heap and stack will not change between commands, allowing for the attack to be much more simplified.

```
$ sudo sysctl -w kernel.randomize_va_space=0
```

- Compile and install the format.c and server.c files, using the commands given in the Makefile

```
make
make install
```

- Setup the containers defined in `docker-compose.yaml`, using the commands below

```
dcbuild  # Build the container images
dcup # Start the containers
```

## Task 1

In this task, our only goal is to provide an input to the server, such that when it tries to print said input, the server crashes. For this task we used `%s` as an input, since this format specifier prints something pointed to by an address in the stack. Therefore, if no address is given it will fetch the address immediatly above its, which in this case leads to the crashing of the server. This happens because that address fetched is probably out of accessible memory for the process.

![Imagem #1](img/Imagem_1.png)

## Task 2

The objective of this task is to get the server to print out some data from its memory. This task is divided into 2 subtasks.

### Task 2.A: Stack Data

In this task, we will try and make the server side print the data inside the stack. In other for this to happen, we have to input enough `%x` specifiers to reach the address of the stack containing data. As a result, we chose **00** as our input, which in hexadecimal is equivalent to **3030**. 

Afterwards, we kept adding `%x` to our input until we reached that elusive number. In the end, we reached that value after 64 specifiers, which tells us there are 63 bytes or 504 characters in between our input and the data. So, we concluded that in other to reach the data buffer from now on we should use 64 `%x` specifiers in our input.

![Imagem #2](img/Imagem_2.png)

![Imagem #3](img/Imagem_3.png)

 ### Task 2.B: Heap Data

 In this subtask, we are required the contents of a secret message in the heap area, where its address is given as a print message from `myprintf`. Our plan was to add the address of the secret message in the beggining of the input and then add `%s`, so that the contents of the address given would be printed. However, as shown before, a single `%s` crashes the server. As a result, we realised we had to reach the data buffer before using `%s`. So, as that specifier already counts as moving 4 bytes forward, we only had to add 63 `%x` in between the address and `%s`. After that, the end of the server's print included "A secret message", which was our goal.

P.S: Form this point on, as strings started to increase in size, we started running build_string.py instead of writing them directly in the terminal

![Imagem #4](img/Imagem_4.png)

![Imagem #5](img/Imagem_5.png)

## Task 3

### Task 3.A: Change the value to a different value

For now our goal is to change the value of the **target** variable, whose address is also given as a print message from `myprintf`. 

In order to do so, we can `%n`. This specifier allows to substitute the value of an atribute on a given address by the number of characters printed in the command before said specifier. 

Since at this point no specific value is required, we simply replaced the address from the last input to the variable's address and `%s` from the last input by `%n`. This change the variable's value to **0xEE**, which means that 254 characters were written before reaching the last format specifier.

![Imagem #6](img/Imagem__6.png)

![Imagem #7](img/Imagem__7.png)

### Task 3.B: Change the value to 0x5000

For the last task, we had to repeat the last task, but changing the value of target to **0x5000**. Doing some math, this means that 20480 characters or 2560 bytes must be written before reaching `%n`.

Firstly, we tried filling the difference between the last value and this one with blankspaces. However, this lead to server overflowing, not being to produce the desired results. This means we had to find a way to increase the number of chars written without overflowing.

Finally, we discovered that the amount of bytes occupied by `%x` can be changed by simply changing it to `%.{number}x`, without risking any overflow. As a result, since the address occupies 4 bytes, we add 20476 to fill, which after some math meant that the specifier would be `%.325x`, and that we would have to add a blankspace, which would result in 4 + 325*63 + 1 = 20480 bytes written before `%n`.

After all these steps, the variable finally took the intended value of **0x5000**, concluding the final step of this logbook.

![Imagem #8](img/Imagem__8.png)

![Imagem #9](img/Imagem__9.png)

## Question 2

By allocating the format string to the heap instead of the stack, we removing direct access to stack-based data and addresses, which are essential to all tasks mentioned here. Except for task 2.B (this task is performed on the heap, meaning that storing the format string there would make this task as difficult as before in the worst case), all other tasks would become much more difficult, or even impossible in some cases.