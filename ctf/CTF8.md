# CTF 8 - Oitava Semana

Este CTF é baseado em SQL Injection, sendo que o objetivo é obter o username e password do administrador de um site do Wordpress, utlizando endpoints na API do site que façam queries à base de dados e sejam dinamicamente geradas a partir de input do utilizador e abusar dos mesmos para executar o ataque. A flag esperada é algo do estilo **flag{password_do_admin}**.

## Primeiro Passo - Analisar as vulnerabilidades do servidor

Ao inspecionar o site, encontrámos a versão do Wordpress, além de um plugin existente:

- Wordpress 6.7.1
- Notificationx 2.8.1

Após pesquisarmos CVE's relacionados a estas versões, apareceu nos o [CVE-2024-1698](https://www.cvedetails.com/cve/CVE-2024-1698/), como a melhor vulnerabilidade a utilizar para o efeito, visto que é efetivo para a nossa versão do Notificationx, e para além disso utiliza SQL Injection para obter informações sobre os administradores, que é exatamente o nosso objetivo.

## Segundo Passo - Utilização do ficheiro malicioso

Ao pesquisarmos por exploits para esta vulnerabilidade, encontramos este [GitHub](https://github.com/kamranhasan/CVE-2024-1698-Exploit), que continha um ficheiro pré-feito para executar este ataque, sendo necessário apenas substituir os campos **url** e **delay** pelos valores desejados. Ao analisar a página de comments às notificações apresentadas e correr o comando `http://44.242.216.18:5008/wp-json`, descobrimos que o url para a API do plugin era `http://44.242.216.18:5008/wp-json/notificationx/v1/analytics`. Posto isto, foi uma questão de deduzir o delay desejado. Após inicialmente falharmos por usarmos a internet da FEUP, ao tentar o exploit com delays de valor 2, 3 e 4 segundos, obtivemos erros deste estilo:

![Imagem 1](img/Imagem8_1.png)

Finalmente, ao correr o programa com um delay de 5 segundos, conseguimos obter os dados do administrador, de username **admin** e password-hash **$P$BuRuB0Mi3926H8h.hcA3pSrUPyq0o10**.

![Imagem 2](img/Imagem8_2.png)

## Terceiro Passo - Decrifrar a password-hash obtida

Ao realizarmos o ataque, obtivemos o username e password-hash do admin, contudo o enunciado pedia pela password. Isto requeriu utilizar softwares como o **Hashcat** de modo a obter a password do admin. Após instalar o programa, tentámos utilizar um comando neste formato `hashcat -m 400 hash.txt wordlist.txt`, em que **hash.txt** é um ficheiro com a hash que pretendemos descodificar, e **wordlist.txt** é uma lista de passwords frequentemente usadas nas quais o programa se baseia para obter a password. Como sugerido por uma breve pesquisa sobre wordlists, utlizámos uma wordlist deste [repositório](https://github.com/danielmiessler/SecLists/blob/master), mais precisamente a [Ignis-10K](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Pwdb-Public/Wordlists/ignis-10K.txt).

Após tentar sem sucesso correr este programa na Virtual Machine do SeedLabs, ao tentar no OS raíz obtivemos a password **heartbroken**, sendo que ao colocar a flag **flag{heartbroken}** no site de CTF, este validou a resposta como certa, concluindo assim o desafio.

![Imagem 3](img/Imagem8_3.png)