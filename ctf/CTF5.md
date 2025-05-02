# CTF5 - Quinta Semana

Este CTF é baseado na matéria de buffer overflow, sendo esta técnica a pretendida a utilizar para obter a flag desejada. Esta flag encontra-se no ficheiro **flag.txt**, sendo que é suposto executarmos o ficheiro **program** sem o alterarmos usando apenas o ficheiro **exploit-template.py** numa primeira instância, usando depois esse ficheiro para obter a flag escondida no servidor do CTF-FSI.

## Primeiro Passo - Verificar as características do executável

De maneira a podermos delinear uma estratégia de ataque, executamos o comando `checksec --file=program`, de modo a obter as informações necessárias ao nosso ataque. Ao executar o comando, verificamos que não existe RELRO (não significante para este CTF) nem canário (embora apareça na imagem, a descrição do CTF no moodle diz que não existe, e que para além disso não influencia o ataque).

Além disso, sabemos que a stack tem permissões de execução, que tem zonas com tripla permissão (leitura, escrita e execução), e que as posições do endereços na stack não são randomizadas (PIE). Todas estas informações favorecem o lado do atacante, facilitando bastante a nossa tarefa.

![Imagem 1](img/Imagem5_1.png)

## Segundo Passo - Descrição do executável programa

O ataque será executado no programa **program**, pelo que analisaremos o ficheiro main.c no qual ele foi construído.

Este ficheiro de código inclui 2 funções auxiliares (readtxt e echo), para além da função principal. A função readtxt recebe um argumento chamado name, que corresponde ao nome do ficheiro que é aberto nesta função, pois este argumento é adicionado ao comando `cat <name>.txt`, que depois é executado usando a função system. Já a função echo simplesmente imprime uma linha de código `Echo <str>`, sendo str o argumento de echo.

Por fim, a função main cria as 2 variáveis locais fun (pointer para o início de uma string) e buffer (string), e utiliza fun para imprimir os conteúdos de rules.txt, fazendo-a apontar para readtxt e preenchendo-a com "rules". A seguir, faz fun apontar para echo e recebe o input do user, colocando-o na string buffer. Após isso, fun é preenchido com buffer, efetivamente fazendo com que o user input seja imprimido pela função echo.

![Imagem 2](img/Imagem5_2.png)

## Terceiro Passo - Executar o ataque localmente

Dados as informações que temos, o nosso plano é efetuar um overflow na variável buffer com o nosso input, fazendo com a variável fun passa a apontar para readtxt, e que o seu conteúdo contenha a palavra "flag", sendo tomada como argumento desta função, abrindo o ficheiro flag.txt e revelando a flag escondida.

Ao construirmos o payload, definimos que a primeira parte do payload será "flag\0", de modo a que quando a função readtxt receba o nosso payload como argumento, leia flag e pare graças ao caractere de final de string (esta parte foi descoberta após verificar que a função readtxt estava a ser corretamente chamada, mas o input não estava a parar em flag).

No fim do payload, estaria o endereço da função readtxt (mostramos na imagem como foi obtido). Este endereço ultrapassará o tamanho do buffer, dando overflow nele. Este overflow passará até à variável, cujo apontador será reescrito. Desta maneira, esta variável passa a apontar para a função readtxt. 

Por fim, a parte do meio são caracteres quaisquer ('A' no nosso caso), em número suficiente para que, em conjunto com o componente flag anterior, ocupem todo o espaço do buffer, permitindo ao endereço de readtxt dar overflow ao buffer. No nosso caso, foram 32 (tamanho do buffer) - 5 (tamanho do componente flag do payload) = 27 caracteres.

Ao enviarmos este payload como input, obtemos como output `flag{test}`, verificando-se o sucesso do ataque.

![Imagem 3](img/Imagem5_3.png)

![Imagem 4](img/Imagem5_4.png)

## Quarto Passo - Executar o ataque no servidor

Posto isto, foi simplesmente uma questão de ativar a linha de código `r = remote('ctf-fsi.fe.up.pt', 0000)`, substituindo o último valor, pelo valor 4000 da porta do servidor deste desafio e comentar a linha `r = process('./program')`. Fazendo isto e ligando a VPN da FEUP, obtivemos a flag **flag{4dm1n_fun_w45_0wn3d}**, concluindo assim este CTF.

![Imagem 5](img/Imagem5_5.png)