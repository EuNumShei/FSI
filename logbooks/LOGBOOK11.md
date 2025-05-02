# Questão 1

## Task1
Nesta tarefa, começamos por verificar os websites locais que temos atualmente na virtual machine com o comando `sudo nano /etc/hosts`

Em seguida, adicionamos 2 websites que vamos usar mais para a frente. Um deles é o website padrão bank32 e o outro somos nós que críamos. Usámos o apelido do Júlio: `santos` seguido do ano atual `2024`.

```
10.9.0.80       www.bank32.com
10.9.0.80       www.santos2024.com
```

Depois disso, usámos o comando `openssl version -d` para descobrir o path do openssl.cnf e críamos uma cópia no diretório do lab desta forma.

```shell
cp /usr/local/ssl/openssl.cnf ./openssl.cnf
```

Depois, abrímos a cópia no vs code, e modificamos no código o `default_CA` como prentendido no enunciado.
Críamos a estrutura do diretório demoCA com alguns ficheiros, incluíndo o index.txt vazio e o serial com o número 1000 no formato de string.

```shell
mkdir -p demoCA/certs demoCA/crl demoCA/newcerts
touch demoCA/index.txt
echo 1000 > demoCA/serial
```

Agora, gerámos o certificado de autoridade com as todas as flags mostradas:

```shell
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 \
-keyout ca.key -out ca.crt \
-subj "/CN=www.modelCA.com/O=Model CA LTD./C=US" \
-passout pass:dees
```

Com o certificado `x509` obtívemos os seguintes resultados com o comando `openssl x509 -in ca.crt -text -noout`:

- O certificado CA é verdadeiro.

![](img/CA_proof.png)


- É um certificado `self-signed` porque o `issuer` é igual ao `subject`.

![](img/SS_Indicator.png)


Com o certificado `rsa` obtívemos os seguintes resultados com o comando `openssl rsa -in ca.key -text -noout` e a pass `dees`:

- `módulo n`:
```
modulus:
    00:cd:e4:7f:8f:90:35:d1:22:9f:e3:82:ac:df:77:
    0e:f0:3c:f5:b1:65:8a:6a:4a:b9:42:11:73:47:4b:
    6a:7c:bd:05:f4:d3:1e:c6:10:bc:51:76:15:d1:cd:
    6c:77:49:73:1a:cd:da:af:b5:f8:b5:60:9b:e7:e2:
    cc:e7:9b:c2:f4:86:4d:2e:85:6d:96:5b:81:a7:23:
    73:e3:fc:1a:2b:e2:e5:c4:9b:b8:03:1a:1a:89:60:
    cc:4b:e3:08:15:0b:af:e0:3a:e9:84:60:b3:13:05:
    13:89:b5:9e:e6:e1:ad:3a:7a:88:e4:79:24:3c:18:
    e0:5e:c0:a0:00:e4:a9:85:74:4c:56:d1:3d:bd:01:
    72:af:40:b5:0c:70:02:3a:ce:13:e7:df:0b:a4:62:
    0f:2f:8a:8d:a1:71:5d:50:df:74:3a:ac:6b:0c:02:
    1c:7d:0e:57:ce:8e:a2:b0:67:a2:5b:1f:d0:de:05:
    51:6f:3b:55:b5:f9:fe:67:6f:09:e6:87:c1:4c:14:
    38:91:0e:7e:c0:c1:6b:58:45:f5:4d:21:36:c6:52:
    cf:dc:21:33:da:75:4a:d3:a5:b3:ec:09:19:11:51:
    a4:01:9c:ca:11:16:2c:79:fe:f7:22:98:4a:6d:48:
    f9:f0:bd:aa:81:fa:64:87:a6:8f:06:a6:d7:31:f7:
    95:9f:42:b5:d9:f1:eb:9d:ed:75:ec:eb:27:e0:9c:
    a9:03:32:32:ec:62:09:f6:9b:b1:46:49:1d:03:4b:
    fc:fd:bc:00:20:bc:fd:8e:08:f2:eb:91:38:46:09:
    0e:81:bd:75:7d:58:0f:a9:18:27:8a:5d:b4:7a:21:
    d6:21:49:d5:42:97:bc:68:bf:9d:91:60:1d:b4:6b:
    4c:3f:09:f7:bb:30:9e:ed:81:a0:86:fc:8a:0d:0b:
    f0:a5:c0:40:23:a8:ed:3d:fa:51:15:cd:80:0e:df:
    61:de:46:63:2a:95:0a:e9:ca:a9:3e:e0:45:ff:3d:
    57:74:ef:08:56:3a:54:2e:cd:f3:75:ab:29:d9:6c:
    dc:7e:40:69:1d:77:35:5e:2b:39:ac:84:94:b7:86:
    b9:ae:83:11:8f:6d:14:8a:ae:15:bc:a2:50:f8:86:
    eb:82:5c:2f:24:a1:dc:cc:76:77:ec:9c:35:45:3e:
    9c:21:dc:51:5a:b2:66:3a:84:c3:82:fa:41:71:70:
    72:d3:6e:5f:ac:98:e4:17:bd:a0:6d:34:8d:b4:76:
    66:a6:c0:bf:b9:65:bb:04:1e:c8:34:21:ad:f6:4b:
    85:11:a9:f6:a8:fd:0b:7a:d2:cf:9d:a0:59:16:d3:
    ed:bc:19:42:a2:5f:b8:b4:10:d8:8e:5b:60:62:d2:
    d8:8b:75
```

- `Expoente público e`:
```
publicExponent: 65537 (0x10001)
```

- `Expoente privado d`:
```
privateExponent:
    05:6d:c0:ef:48:1e:23:25:86:91:b9:20:55:c4:0d:
    e0:c7:ba:b2:b2:ba:b3:92:c4:cf:b9:26:10:fb:2c:
    13:9d:e8:4a:4c:46:c3:72:2d:84:f8:58:1a:f0:0f:
    ac:15:83:b3:71:3a:12:e7:f9:66:ce:e5:4d:77:ed:
    6f:2d:ec:36:06:54:82:a5:81:fe:34:eb:76:3c:b9:
    11:89:d4:26:d4:14:ba:da:60:3f:b4:b0:7e:6e:ce:
    f4:31:48:45:45:c9:9d:5f:f0:48:4d:44:6b:7b:f6:
    c7:ea:c2:05:d4:6d:7c:dd:3e:3e:c7:f2:ec:a2:bb:
    47:c4:4c:73:b9:93:be:b1:12:52:37:b2:29:09:a0:
    bc:7e:38:47:db:ac:33:60:bb:a5:a1:7b:ed:19:a8:
    cc:d8:1a:63:e7:59:4e:88:95:c7:f6:43:dc:4d:31:
    f3:1a:2d:b6:84:8c:45:9a:75:ed:71:29:50:b1:53:
    34:5b:9c:26:f0:30:85:09:a8:79:2b:9b:2d:26:5b:
    57:77:ca:5f:ca:b9:c9:da:0e:5d:ba:42:77:78:37:
    82:d6:5d:47:a5:50:13:c4:8a:b9:ee:83:7a:b1:6c:
    61:9e:80:b5:72:a0:39:97:3f:6c:ea:17:0c:4a:a0:
    4c:78:b3:c6:39:93:ec:ff:b7:45:b5:a0:76:f0:d0:
    41:16:a4:98:96:fc:c4:95:db:c6:56:2b:1a:39:ca:
    89:be:a2:cc:a4:bb:15:3b:e2:51:4d:d6:d9:d7:5d:
    46:29:cf:2e:2e:77:97:e3:85:9f:90:86:12:a8:76:
    63:68:45:a8:5b:a7:30:76:2e:94:8a:56:33:7f:64:
    c1:61:8e:97:16:58:a9:dd:64:ce:cc:c8:60:38:92:
    b0:62:91:18:48:1e:05:d3:dd:6c:c0:46:c3:96:db:
    b6:29:75:ec:12:0f:b3:e6:02:12:bc:7f:8d:23:1a:
    20:c7:09:2e:f5:24:e4:04:33:fd:aa:86:f2:49:f9:
    7d:57:77:c2:b9:40:3f:b2:1d:3c:69:df:53:c4:ed:
    dc:d3:41:39:ae:a7:30:64:c8:46:1a:cc:97:39:28:
    71:b0:19:a2:4f:5d:6c:c8:44:72:c7:2c:9e:1f:fa:
    30:a7:c7:e8:fa:3c:da:a3:27:5d:ba:83:bb:a4:39:
    33:f0:92:b9:21:5a:97:ae:bd:51:2b:ac:15:ab:eb:
    f1:2e:e1:52:bc:b8:4d:01:5f:d5:d4:b3:84:0a:5a:
    83:55:9c:1c:70:ba:4e:00:98:62:78:1e:bc:4e:09:
    ea:83:89:c2:9f:7d:45:d0:ca:18:5e:87:5b:e5:28:
    c4:3e:f7:84:3f:58:3e:68:5c:87:1c:24:54:63:cc:
    30:01
```

- `1º Elemento`:
```
prime1:
    00:ec:28:8a:d6:9f:7e:83:12:c4:ba:8c:1a:e0:25:
    b3:c2:47:c5:09:f5:ab:2e:0a:5b:b3:3a:16:31:e0:
    59:23:0d:a9:8a:ab:c5:75:72:98:c6:66:93:6b:8e:
    3f:21:68:b1:0d:bf:5b:4e:02:78:e7:ad:39:fb:12:
    d5:74:74:37:6c:d2:f5:bc:0c:51:5b:88:b6:c2:fb:
    71:9a:18:6b:c9:50:7e:44:f2:2f:2c:97:c9:d7:98:
    d3:c1:d3:3b:8a:a8:75:fc:39:ac:ac:27:c2:44:84:
    92:e6:f7:77:16:b1:35:ca:b0:11:5f:5e:86:f8:7b:
    db:3c:20:17:4e:82:8a:c3:bb:7b:39:f3:b0:85:81:
    79:4b:6a:37:58:92:b1:90:f8:56:64:8f:b3:a7:e6:
    28:f3:03:50:4a:83:6a:6c:53:bb:69:2e:2d:01:11:
    7f:34:3b:82:57:24:ab:71:ce:2c:f3:b6:23:80:a5:
    3f:4b:3f:5a:8c:c7:d4:71:ff:ab:dc:4c:db:40:6d:
    ed:a7:d4:b1:6f:fb:12:6c:2b:a4:24:32:fc:aa:7b:
    64:f3:46:ce:1a:ab:ca:33:f4:de:1e:47:b7:49:cb:
    9e:0f:7c:01:d5:f4:53:53:5b:a9:37:66:3c:4d:dd:
    80:3d:7e:aa:7b:b6:ad:a0:98:5b:b8:96:a4:e6:85:
    9e:11
```

- `2º elemento`:
```
prime2:
    00:df:30:fa:6d:77:71:fd:4c:04:58:b1:0c:1f:5c:
    dc:4c:e0:00:4d:74:00:ea:29:b4:0b:32:61:83:b9:
    74:24:01:df:d7:63:ce:0a:63:6b:f4:a6:8f:46:d3:
    d5:c5:5f:aa:e9:32:10:8c:a8:38:6f:b2:3c:f3:c3:
    2e:74:32:b9:d7:fc:61:86:22:68:14:1c:8b:bd:37:
    c5:8c:18:f6:55:c7:a0:bc:32:fe:14:2a:94:26:39:
    72:69:ac:16:57:c8:a3:97:47:e4:99:e5:18:10:4e:
    7f:59:34:4b:71:ed:ad:e9:74:28:fb:7a:e7:8a:39:
    54:d0:f7:54:1a:c9:34:48:4d:87:9c:82:5c:6b:da:
    c2:6b:cd:93:2c:ba:84:e7:31:80:56:23:87:c2:6e:
    5c:00:27:f6:b0:d4:11:30:fb:a0:af:be:74:52:d7:
    8c:58:75:e4:58:13:d9:e7:eb:fd:02:5a:aa:0b:78:
    0b:c8:f8:db:fc:c5:5b:35:94:72:3a:ce:76:2f:c7:
    b2:64:fc:84:ee:62:d6:d9:09:0d:21:ee:c1:97:fc:
    31:31:1c:2b:c2:f2:f8:4d:6f:53:91:71:71:43:2c:
    55:2a:70:42:3a:11:a0:ac:80:d1:f2:78:6b:a1:d1:
    35:f8:d6:74:80:f8:e0:6c:42:a5:7f:e8:7a:59:46:
    83:25
```

## Task2

Nesta tarefa, pedimos de assinatura de certificado para o nosso servidor web `www.santos2024.com`. Para isso, usámos este comando:

```shell
openssl req -newkey rsa:2048 -sha256 \
-keyout server.key -out server.csr \
-subj "/CN=www.santos2024.com/O=Santos2024 Inc./C=US" \
-passout pass:dees
```

De seguida, inspecionamos o valor da chave publica e privada com estes comandos respetivamente:

```
openssl req -in server.csr -text -noout
openssl rsa -in server.key -text -noout
```


Depois, adicionamos nomes alternativos ao subject. Além do apelido `santos`, usámos `teixeira`e `cunha`. Com isto, criámos um novo pedido com os dados completos:

```shell
openssl req -newkey rsa:2048 -sha256 \
-keyout server.key -out server.csr \
-subj "/CN=www.santos2024.com/O=Santos2024 Inc./C=US" \
-addext "subjectAltName = DNS:www.santos2024.com, DNS:www.teixeira2024.com, DNS:www.cunha2024.com" \
-passout pass:dees
```

No fim, com o comando req usado anteriormente, obtivemos o seguinte resultado.

- O `subject` do servidor Web

![](img/task2_p1_lb11.png)


- Os nomes alternativos do `subject` conforme pretendido:

![](img/task2_p2_lb11.png)


## Task3

Nesta tarefa, começammos por criar uma cópia do ficheiro `openssl.cnf` e dé-mos o nome de `myCA_openssl.cnf`.

Depois, abrímos o ficheiro, descomentamos a variavel `copy_extensions`, substituímos a `policy` atual por `policy_anything` no vs code e sálvamos o ficheiro.

De seguida, críamos um pedido de certificado para o nosso server com este novo ficheiro.

```shell
openssl ca -config myCA_openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in server.csr -out server.crt -batch \
-cert ca.crt -keyfile ca.key
```

E observamos os resultados com o comando `openssl x509 -in server.crt -text -noout`:

- Cabeçalho do certificado com `ìssuer`, `validity`e `subject`.

![](img/task3_p1_lb11.png)

- Extenções e nomes alternativos do `subject`.

![](img/task3_p2_lb11.png)


## Task4

Nesta tarefa, começamos por criar um container com os comandos do `docker compose`. Verificamos que foi criado um container com `dockps` e depois entramos nele com `docksh <id>`.

Dentro do container, críamos a nossa configuração apache e guardámos neste path `/etc/apache2/sites-available/`.

![](img/taks4_conf.png)

De seguida, pegamos nos certificados da anterior: `server.key` e `server.crt`, colocamos na pasta volumes que é usado para passar dados da virtual machine para o container atual.

Depois, copiamos os certificados para a pasta `certs` com este comando `cp /volumes/certs/server.* ./certs`, porque o website vai buscar os certificados a esta pasta.

Ainda dentro do container, ativamos o modulo sssl e os sites descritos na configuração de apache e abrimos no servidor assim:

```shell
a2enmod ssl
a2ensite santos2024_apache_ssl 
service apache2 start
```

Se abrirmos o website `www.santos2024.com` com `https`, o site aparece que não é seguro.

![](img/task4_https_lb11.png)

Se testarmos o mesmo website com `http`, obtémos o outro html criado.

![](img/task4_http_lb11.png)


Por último, para o firefox reconhecer o website como seguro, vamos executar os passos seguintes:

- ir a `about:preferences#privacy`
- `view certificates` nos `certificates`
- `authorities` and `import`
- selecionamos o `ca.crt`

Agora, se formos ao mesmo website, novamente com `https` obtemos o resultado pretendido.

![](img/task4_https_safe_lb11.png)

## Task5

Nesta tarefa, começamos por adicionar este website local nos hosts da VM usando o comando mostrado na tarefa 1. Escolhemos o website da Caixa Geral de Depósitos `www.cgd.pt` para os labs seguintes.

```
10.9.0.80 	www.cgd.pt
```

De seguida, críamos uma `document root` nova, com `mkdir -p /var/www/cgd_fake`. Dentro da pasta `cgd_fake`, adicionamos um html básico para o nosso website num novo ficheiro `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CGD Fake Site</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 50px;
        }
    </style>
</head>
<body>
    <h1>Bem-vindo ao CGD</h1>
    <p>Este é um site falso para fins de simulação.</p>
</body>
</html>
```

Depois, adicionamos a seguinte VirtualHost dentro do ficheiro `santos2024_apache2_ssl.conf`:

```
<VirtualHost *:443>
    DocumentRoot /var/www/cgd_fake
    ServerName www.cgd.pt
    DirectoryIndex index.html
    SSLEngine on
    SSLCertificateFile /certs/server.crt
    SSLCertificateKeyFile /certs/server.key
</VirtualHost>
```

Se abrirmos o browser com o url `www.cgd.pt`, obtémos o seguinte resultado.

![](img/task5_p1_lb11.png)

Se carregarmos em aceitar riscos e continuar, vamos para o website que eu criei, e o browser mostra que ele não é seguro.

![](img/task5_p2_lb11.png)


## Task6

Nesta tarefa:, vamos repetir alguns passos das tarefas anteriores. Começamos por criar um certificado novo com este comando
```shell
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 \
-keyout compromised_ca.key -out compromised_ca.crt \
-subj "/CN=www.modelCA.com/O=Model CA LTD./C=US" \
-passout pass:dees
```

Depois, criamos um servidor com o url `www.cgd.pt` e com nomes alternativos. 

```shell
openssl req -newkey rsa:2048 -sha256 \
-keyout fake_site.key -out fake_site.csr \
-subj "/CN=www.cgd.pt/O=CGD Inc./C=US" \
-addext "subjectAltName = DNS:www.cgd.pt, DNS:www.santos2024.com, DNS:www.teixeira2024.com"
-passout pass:dees
```

A seguir, configuramos o certificado com `compromisedCA_openssl.cnf`(uma cópia do ficheiro .cnf usado anteriormente).

```shell
openssl ca -config compromisedCA_openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in fake_site.csr -out fake_site.crt -batch \
-cert compromised_ca.crt -keyfile compromised_ca.key
```

De seguida, movemos os ficheiros `fake_site.key` e fake_site.crt` para a pasta volumes e executamos este comando para colocar as chaves do servidor na pasta certa do container (semelhante à tarefa 4):

```shell
cp /volumes/certs/fake_site.* ./certs
```

E ao verificar a pasta `certs` no container, obtivemos isto:

![](img/task6_p1_lb11.png)


Ao contrário da tarefa 5, vamos criar e colocar a VirtualHost num ficheiro à parte `cgd_apache_ssl.conf` mas fazendo algumas alterações (adicionamos os nomes alternativos usados no servidor criado nesta tarefa).

```
<VirtualHost *:443>
    DocumentRoot /var/www/cgd_fake
    ServerName www.cgd.pt
    ServerAlias www.santos2024.com
    ServerAlias www.cunha2024.com
    DirectoryIndex index.html
    SSLEngine on
    SSLCertificateFile /certs/fake_site.crt
    SSLCertificateKeyFile /certs/fake_site.key
</VirtualHost>
```

No fim, executamos os seguintes comandos para abrir o servidor:
``` 
a2enmod ssl
a2ensite cgd_apache_ssl 
service apache2 start
``` 

Além disso, para o browser reconhecer o site como seguro, seguimos os seguintes passos:
- ir a `about:preferences#privacy`
- `view certificates` nos `certificates`
- `authorities` and `import`
- selecionamos o `compromised_ca.crt`

E se abrirmos o browser novamente, obtemos a imagem pretendida.

![](img/task6_p2_lb11.png)

# Questão 2

Quando uma autoridade de certificação (CA) é comprometida, os certificados emitidos por ela precisam de ser imediatamente revogados. Um dos mecanismos mais usados consiste em usar uma `white-list` como o TSL (Trusted Service Provider Lists) e atualizar frequentemente os certificados que não foram comprometidos.

Os adversários que pretendam tirar proveito de uma CA comprometida podem começar por bloquear ou manipular as atualizações das TSL, impedindo que os sistemas da vítima reconheçam a revogação da CA. Além disso, podem falsificar respostas de confiança, como respostas OCSP, para enganar o sistema de validação TLS.

Outra estratégia é aproveitar sistemas ou navegadores desatualizados, que podem não ter as últimas listas de CAs revogadas, e redirecionar usuários a servidores maliciosos por meio de ataques DNS ou alterações no arquivo `/etc/hosts`.

Além disso, os adversários podem usar sub-CAs ou intermediários confiáveis dentro da hierarquia de certificação para emitir certificados fraudulentos. Para ampliar o alcance, podem ainda criar servidores de atualização falsos que distribuem TSLs adulteradas.




