# CTF 7 - Sétima Semana

Este CTF é baseado em XSS injection, estando a flag num ficheiro **flag.txt** disponível no server do serviço **Copyparty**, estando no entando bloqueada com a mensagem `Nice try, I am only accessible via JavaScript shenanigans.`. Utilizámos XSS injection para obter acesso ao ficheiro e à flag lá existente.

## Primeiro Passo - Analisar as vulnerabilidades do servidor

Ao analisarmos os ficheiros disponíveis no repositório base do servidor, verificámos que o ficheiro **help.txt** continha todas informações sobre as versões do servidor e plugins, incluindo:

- CopyParty 1.8.6
- CPython v3.10.12
- jinja2 v2.11.3
- pyftpd v1.5.7

Após pesquisarmos CVE's relacionados a estas versões, apareceu nos o [CVE-2023-38501](https://www.cvedetails.com/cve/CVE-2023-38501/), como única vulnerabilidade documentada do CopyParty 1.8.6, na categoria de Cross-Site-Scripting (XSS). Como tal, percebemos imediatamente que esta era a vulnerabilidade a explorar no contexto do problema.

![Imagem 1](img/Imagem7_1.png)

## Segundo Passo - Criação do script malicioso

Ao procurar por XSS Injection Bypass Techniques, encontrámos este [OWASP XSS_Filter_Evasion_Cheat_Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#img-onerror-and-javascript-alert-encode), que incluía várias técnicas para fazer o ataque. Decidimos usar a técnica que utiliza o campo **OnError** de um atributo img para colocar o script, que será utilizado quando a imagem não conseguir ser aberta.

Primeiro, temos de adicionar o parâmetro `?k304=...` especificado na vulnerabilidade, de modo a que o exploit funcione.

Depois, injetamos um script, que vai abrir o ficheiro flag.txt (que inclui um campo **credentials** que permite que as cookies sejam guardadas, que permite driblar as restrições) e guardar o seu conteúdo num alerta (foi a maneira mais fácil que arranjámos para guardar esta informação com JavaScript).

Depois de isto não funcionar por si só, Descobrimos que ao usar `%OD` e `%0A` antes do script, isto permitia driblar certos inputs que impediam a execução do ataque.

No final, ficou com este aspeto:

```?k304=y%0D%0A%0D%0A<img src=copyparty onerror="fetch('/flag.txt', { credentials: 'include' }).then(response => response.text()).then(data => alert(data)).catch(error => console.error('Error:', error));">```

## Terceiro Passo - Executar o ataque

Por fim, ao colocar o script construído no fim do link do servidor, apareceu um alert, como pretendido, com o seguinte conteúdo ``, confirmando o sucesso do ataque

![Imagem 2](img/Imagem7_2.png)