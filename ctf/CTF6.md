# CTF 6 - Sexta Semana

Este CTF é baseado em format string manipulation, sendo essa a estratégia usada para obter a flag neste contexto. A flag encontra-se num ficheiro flag.txt, que será aberto utilizando um ficheiro **exploit-template.py** para manipular um outro ficheiro **program**. De seguida, usaremos a estratégia delineada no **exploit-template.py** para obter a flag escondida no servidor CTF-FSI.

## Primeiro Passo - Analisar as vulnerabilidades do program

De maneira a podermos delinear uma estratégia de ataque, executamos o comando `checksec --file=program`, de modo a obter as informações necessárias ao nosso ataque. Ao executar o comando, verificamos que não existe RELRO (não significante para este CTF) mas que contudo existe canário, o que dificulta o nosso processo.

Além disso, sabemos que a stack tem permissões de execução, que tem zonas com tripla permissão (leitura, escrita e execução), e que as posições do endereços na stack não são randomizadas (PIE). Quase todas estas informações favorecem o lado do atacante, facilitando bastante a nossa tarefa.

![Imagem 1](img/Imagem6_1.png)

## Segundo Passo - Descrição do executável program

O ataque será executado no programa **program**, pelo que analisaremos o ficheiro main.c no qual ele foi construído.

Este ficheiro de código inclui 2 funções auxiliares (readtxt e echo), para além da função principal. A função readtxt recebe um argumento chamado name, que corresponde ao nome do ficheiro que é aberto nesta função, pois este argumento é adicionado ao comando `cat <name>.txt`, que depois é executado usando a função system. Este nome contudo é cortado no 7º caractere, tendo portanto apenas 6 caracteres disponíveis para o nome do ficheiro. Já a função echo simplesmente imprime uma linha de código `Echo <str>`, sendo str o argumento de echo.

Por fim, a função main cria as 2 variáveis locais fun (pointer para o início de uma string) e buffer (string), e utiliza fun para imprimir os conteúdos de rules.txt, fazendo-a apontar para readtxt e preenchendo-a com "rules". A seguir, faz fun apontar para echo e, antes de receber o input do user, fornece o endereço da variável fun como pista, colocando depois o input na string buffer. Após isso, fun é preenchido com buffer, efetivamente fazendo com que o user input seja imprimido pela função echo.

![Imagem 2](img/Imagem6_2.png)

## Terceiro Passo - Executar o ataque localmente

O nosso plano é fornecer como input a string "flag" junto com uma série de format strings que nos permitam mudar o endereço para o qual fun aponta (echo) para o endereço de readtxt. Como tal, o nosso payload é composto de 2 partes.

Primeiro, contém a string "flag", que pretendemos que seja o argumento tomado pela função readtxt. Contudo, a função toma os primeiros 6 caracteres dados como o nome do ficheiro e, graças a tal, esta parte terá de ocupar 8 caracteres, de modo a não influenciar o que vem a seguir. Para contornar isto, colocamos "./" atrás de "flag", para que quando se faça `cat ./flag.txt`, o programa pegue o ficheiro flag.txt do repositório atual, permitindo assim continuar a abrir o ficheiro certo. No fim desaa parte, adicionamos 2 caracteres quaisquer ("AA" neste caso), pois não serão adicionados ao comando de qualquer maneira, de modo a atingir os 8 caracteres, ficando no fim **"./flagAA"**.

A segunda parte consiste na mudança de endereços de fun, do echo para o do readtxt. Graças à permissão de usarmos a biblioteca **pwntools**, podemos usar a função `fmtstr_payload(payload)`, que nos automatiza o processo de formação do payload com as format strings. Contudo, para usarmos esta função, precisamos de saber o **offset**, que nada mais é do que a posição do input em relação à stack.

Para isso, usamos a função `exec_fet`, dada na documentação sobre as format strings do pwntools, que com mais dois comandos nos fornece o offset, que neste caso acabou por ser 1. Como a parte anterior do payload ocupa 2 bytes, a função tomará o valor 3 como offset.

![Imagem 3](img/Imagem6_3.png)

![Imagem 4](img/Imagem6_4.png)

Posto isto, ao formarmos o payload e o enviarmos, recebemos como input **flag{asd}**, verificando-se o sucesso do exploit.

![Imagem 5](img/Imagem6_5.png)

![Imagem 6](img/Imagem6_6.png)

## Quarto Passo - Executar o ataque no servidor

Posto isto, foi simplesmente uma questão de ativar a linha de código `r = remote('ctf-fsi.fe.up.pt', 0000)`, substituindo o último valor, pelo valor 4005 da porta do servidor deste desafio e comentar a linha `r = process('./program')`. Fazendo isto e ligando a VPN da FEUP, obtivemos a flag **flag{L34k1ng_Fl4g_0ff_Th3_St4ck_44789C2D}**, concluindo assim este CTF.

![Imagem 7](img/Imagem6_7.png)