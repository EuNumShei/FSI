# CTF 4 - Quarta Semana

## Primeira etapa - Encontrar o CVE

Ao entrarmos no site <http:/ctf-fsi.fe.up.pt:5002> pela primeira vez, verificamos que a página base não possui nada para além de html com 2 forms, sendo que um deles permite executar o comando ls -al, tomando como path o input do user. Isto permitiu-nos verificar que o servidor do site possui a sua própria bash de comandos.

 Dado isso, e o facto de que o tema deste CTF foi environment variables, pesquisámos uma mistura destes 2 temas na CVE List Search. Após alguns resultados, verificámos que muitos deles mencionavam o mesmo problema, uma falha na GNU Bash em que, colocando uma string no final de definições de funções, era possível executar código no meio destas funções. 
 
 Após tentarmos todas as vulnerabilidades deste estilo, verificámos que a [CVE-2014-6271](https://www.cve.org/CVERecord?id=CVE-2014-6271) era a vulnerabilidade pretendida.

## Segunda etapa - Executar a vulnerabilidade

Posto isto, percebemos que o ficheiro que continha a flag poderia ser aberto com um comando ``cat``, sendo assim possível aceder à flag. Logo, o passo que faltava era encontrar o ficheiro que continha a flag.

Após encontrar a flag, acabei por perceber que podia simplesmente efetuar o comando ``grep -r "flag{"`` no diretório base, pois sabíamos que a flag seria neste formato. No entanto, acabámos por encontrar a flag por um processo penoso e demorado de verificar manualmente todos os diretórios subsequentes do base em busca de algo suspeito. 

![Grep Commmand](img/grepctf5.png)

Depois de demasiado tempo passado neste processo, ao abrir o último destes diretórios, o **var**, este continha um folder **flag**, que por sua vez continha um ficheiro **flag.txt**. Posto isto, simplesmente adicionámos o comando ``cat /var/flag/flag.txt`` no fim do link do site, seguido de uma string qualquer, e obtivemos a **flag{Zggk3S7GPOyMOoNOCddh7Ylb2fpwyi}**.

![Location of the Flag](img/locationctf5.png)

![Flag](img/flagctf5.png)