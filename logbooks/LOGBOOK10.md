## Task1

Para esta tarefa, começamos por adicionar o website `www.seedlab-hashlen.com` que vamos usar para esta tarefa à lista de hosts. 

Para isso, usamos este comando `sudo nano /etc/hosts` para obter a lista dos hosts atuais. Nela, escrevêmos e salvamos o ficheiro. Depois, construímos o ficheiro e iniciamos o container com `docker-compose build` e `docker-compose up`.

Com o setup concluído, vamos ao ficheiro `key.txt` do labsetup desta semana e escolhémos o uid `1005` e a key correspondente `xciujk`.
Juntamente com o nome de um elemento do nosso grupo (e.g. `JulioSantos`), podemos criar o mac como aparece na imagem abaixo para enviar uma solicitação para aceder a uma lista de ficheiros.

![](img/task1_p2_get_mac.png)

Com este mac, podemos contruir a solicitação e enviá-la ao servidor no browser usando este comando:

```
http://www.seedlab-hashlen.com/?myname=JulioSantos&uid=1005&lstcmd=1&mac=7504a4b7c2e53841ef7e641204baa328fdb66ae19bbf1fedaf7d0a523e7df652
```
Se usármos este mac, obtémos a seguinte mensagem.

![](img/mac_valid.png)

Se tentarmos usar outro mac, vamos obter uma mensagem diferente.

![](img/mac_not_valid.png)


## Task2 

Nesta tarefa, vamos usar o nome `JulioSantos`, a uid `1005` e a key correspondente `xciujk`.
Ou seja, a mensagem fica assim:

```
xciujk:myname=JulioSantos&uid=1005&lstcmd=1
```
Vamos calcular o tamanho da mensagem.
```
6 bytes (key) + 1 byte (:) + 18 bytes (name) + 9 bytes (uid) + 9 bytes (lstcmd) = 43 bytes
43 * 8 = 344 bits 
```

Tendo em conta que temos de reservar 8 bytes para o comprimento da mensagem, vamos calcular o tamanho do padding: `64 - (43 + 8) = 13 bytes`.

Dos 13 bytes, o primeiro byte fica a 0x80 e os restantes 12 vão ficar a 0x00 conforme explicado no enunciado.
Dos 8 bytes do comprimento da mensagem, 6 vão ser 0x00 e os outros 2 fica `0x0158` (344 em hexadecimal).

Ou seja, o padding vai ficar assim:
```
\x80\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01\x58
```

## Task3
Nesta tarefa vamos usar os seguintes dados calculados anteriormente:
- o mac da Task1
- o padding da Task2

Começamos por alterar o ficheiro `length_ext.c` com o mac referido acima da seguinte maneira:

```c
// MAC of the original message M (padded)
c.h[0] = htonl(0x7504a4b7);
c.h[1] = htonl(0xc2e53841);
c.h[2] = htonl(0xef7e6412);
c.h[3] = htonl(0x04baa328);
c.h[4] = htonl(0xfdb66ae1);
c.h[5] = htonl(0x9bbf1fed);
c.h[6] = htonl(0xaf7d0a52);
c.h[7] = htonl(0x3e7df652);
```

Depois, substituímos a mensagem extra pelo comando de transferẽncia desta forma:

```c
// Append additional message
SHA256_Update(&c, "&download=secret.txt", 21);
```

Vamos agora criar o novo mac conforme explicado no tutorial.

![](img/t3_get_mac_lb10.png)


Assim, podemos contruir o URL, com a mensagem, o comando, o novo mac e substituindo o `/x` do padding por `%`.

```
http://www.seedlab-hashlen.com/?myname=JulioSantos&uid=1005&lstcmd=1%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%01%58&download=secret.txt&mac=43203c1f01a899cee936c89f3746851b478d159e7d0a8c9e9d02bbe72ee15906
```

E abrir no browser. Como esperado, aparece o conteúdo do ficheiro `secret.txt`.

![](img/task3_logbook10.png)



