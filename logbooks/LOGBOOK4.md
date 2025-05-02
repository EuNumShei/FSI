# LOGBOOK 4

## Task 1
Ao experimentar os comandos dados nesta tarefa chegamos a estas conclusões:

- O comando printenv e env mostram as variáveis de ambiente atuais.
- O comando "printenv PWD" imprime o caminho para o repositorio atual (/home/seed/Labsetup)
- O comando export permite criar ou modificar as variáveis de ambiente 
- O comando unset permite remover as varáveis de ambiente


## Task 2
Nesta tarefa, separamos o código dado em 2 ficheiros, um para o pai (myprintenv_parent.c) e um para o filho (myprintenv_child.c).

Ao salvarmos as variáveis de ambiente do processo pai em um arquivo (file_parent) e as do processo filho em outro (file_child) e ao comparar os dois arquivos, não encontramos diferenças logo podemos concluir que o processo filho herda todas as variáveis de ambiente do processo pai após o fork().

<img src="img/image_logbook4.png" width="500" height="300" alt="Task2_image">

Isso indica que ambos os processos compartilham o mesmo ambiente de execução, sem qualquer alteração nas variáveis de ambiente entre o pai e o filho.


## Task 3
Ao mudarmos o código de acordo com enunciado desta tarefa chegamos a esta conclusão:

Se nós executarmos o código do arquivo myenv.c com o terceiro parâmetro da função execve definido a NULL, as variáveis de ambiente não são passadas, resultando em uma saída vazia, pois o ambiente do processo não é inicializado.
 
No entanto, se substituirmos o NULL pela variável environ, as variáveis de ambiente serão transferidas para o novo processo. 


## Task 4
Nesta tarefa, vamos usar o comando system() e o comando execve() no código e ver as diferenças.

Se nós utilizarmos o comando system(), um novo processo é criado e nele são automaticamente passadas todas as variáveis de ambiente do processo original. 

Por outro lado, a chamada de execve() substitui o processo atual pelo novo comando especificado, mantendo o mesmo espaço de processo. 


## Task 5
Nós usamos o programa dado que exibe todas as variáveis de ambiente do processo atual, dando-lhe o nome de 'envshower', e ajustamos as permissões do programa para que seja executado com previlégios de root de acordo com o enunciado.

``$ sudo chown root envshower``

``$ sudo chmod 4755 envshower``

Após fazer alterações em variáveis de ambiente, como PATH, LD_LIBRARY_PATH e COURSE_NAME usando o comando "export",

``` bash
export PATH=$PATH:/home/seed/Desktop/script5`
export LD_LIBRARY_PATH=/home/seed/script25`
export VARIABLE_T5=/home/seed`
```

nós observamos que ao executar o programa, todas essas variáveis estavam visíveis, exceto LD_LIBRARY_PATH.

Isto ocorre porque a variável LD_LIBRARY_PATH permite especificar um caminho onde o programa buscará bibliotecas dinâmicas compartilhadas e essa configuração pode possibilitar a execução de um programa malicioso, caso uma das bibliotecas usadas for substituída por uma versão comprometida.

## Task 6
Dado o código apresentado no passo 2.6 deste Lab, criámos um ficheiro **system.c** com esse mesmo código, assim como um ficheiro **ls.c** que contém o software malicioso (no nosso caso um simples print da string "ola"). Estes dois ficheiros foram colocados no folder com o path **'/home/seed/Desktop/cena'**.

Após isso, compilámos os 2 ficheiros, colocámos o 'root' como dono do ficheiro system e tornámos o system num programa Set-UID.

![permissionslogbook4.jpeg](img/permissionslogbook4.jpeg)

Por fim, alterámos a variável de ambiente PATH para o path dos nossos ficheiros, de modo a que quando os comandos sejam executados, eles sejam procurados primeiro no folder com os nossos ficheiros, e apenas caso não os encontre que os procure no folder bin.

![changeenvvariableslogbook4.jpeg](img/changeenvvariableslogbook4.jpeg)

A função system(cmd) executa o programa ``/bin/sh`` antes de pedir à shell para executar o comando cmd. Ora, na nossa versão do Ubuntu, ``/bin/sh`` é um link que aponta para ``/bin/dash``, sendo que este programa não permite processos Set-UID como o que utilizamos. Para contornar este problema, executamos o comando ``sudo ln -sf /bin/zsh /bin/sh`` que liga ``/bin/sh`` a ``/bin/zsh``, uma shell instalada neste SO que permite a execução de processos Set-UID, permitindo assim a continuação deste exploit.

![zshbashlogbook4.jpeg](img/zshbashlogbook4.jpeg)

Depois de todos estes passos, ao correr o executável system, obtivemos o output ``ola``, indicando que o exploit funcionou. Após este passo, ao executar o comando ``ls`` o output passou a ser ``ola`` em vez dos conteúdos dentro do path em questão.

![resultlogbook4.jpeg](img/resultlogbook4.jpeg)

Com este passo, demonstrámos que uma mudança de variáveis de ambiente pode afetar o sistema de diversas maneiras, sendo neste caso específico possível alterar a variável de ambiente PATH de maneira a que código malicioso possa ser executado aquando da execução de um comando simples e inofensivo.

