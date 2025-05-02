# CTF3 - Terceira Semana 

## Primeira Parte - Encontrar o CVE

Dado o site http://143.47.40.175:5001, que é um servidor do WordPress, tentámos seguir o primeiro passo fornecido pelo guia, sendo este encontrar a versão do Wordpress usada, assim como a dos plugins existentes. Após a inspeção do site, isto foi o que encontrámos:

- **Wordpress 5.8.1**
- **Woocommerce 5.7.1**
- **Booster for WooCommerce plugin 5.4.4**
- **MStore API 3.9.0**

Posto isto, tentámos encontrar vulnerabilidades que afetassem estas versões e nos permitissem aceder como administrador do servidor. Achámos inicialmente que seria o [CVE-2024-9289](https://www.cve.org/CVERecord?id=CVE-2024-9289), que afeta tanto esta versão do Wordpress como a do Woocommerce também, e que dá ao utilizador acesso à conta do admin, desde que saiba o email dele. No entanto, após verificarmos que estava errado e continuarmos a procurar, descobrimos 3 CVE's que pareceram ser as respostas certas, sendo eles o [CVE-2023-2732](https://www.cve.org/CVERecord?id=CVE-2023-2732), [CVE-2023-2733](https://www.cve.org/CVERecord?id=CVE-2023-2733) e [CVE-2023-2734](https://www.cve.org/CVERecord?id=CVE-2023-2734). Todos utilizavam a MStore API para obter o acesso a admin, e todos funcionavam para a versão que possuíamos da API. Após tentar os 3, verificámos que o [CVE-2023-2732](https://www.cve.org/CVERecord?id=CVE-2023-2732) era a resposta certa, e assim prosseguimos à próxima fase.

## Segunda Parte - Obter a Flag

Depois de descobrir a vulnerabilidade, pesquisámos um [repositório](https://github.com/RandomRobbieBF/CVE-2023-2732/tree/main) que explicou exatamente como utilizá-la. O primeiro passo foi executar um [ficheiro em Python](https://github.com/RandomRobbieBF/CVE-2023-2732/blob/main/mstore-api.py) fornecido pelo autor do repositório com o seguinte comando:

`python3 mstore-api.py -u URL`

Substituindo o URL pelo nosso site, apareceu a seguinte mensagem:

    The plugin version is below 3.9.3.
    Select a user:
    1. admin
    Enter the user ID:

E após selecionar o número 1, apareceu a seguinte mensagem:

    Congratulations a vulnerable system has been found.

    How to Exploit:

    Visit the following url: http://wordpress.lan/wp-json/wp/v2/add-listing?id=1
    Visit  http://wordpress.lan and you should be logged in as the user you have chosen.

Por isso, após substituir os trechos http://wordpress.lan pelo URL do nosso site, visitámos o primeiro URL que executou o **add listing REST API request**, sendo que como este request não verifica suficientemente as credenciais do utilizador, apresentar o user id é suficiente para o request prosseguir. Graças a esse exploit, quando visitámos o site outra vez, estávamos a controlar a conta do administrador. Posto isto, ao aceder à página de administração do WordPress, encontrámos um post privado que continha a flag do desafio, **flag{byebye}**.