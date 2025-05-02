# Task1 

Nesta tarefa, começamos por chamar o ficheiro `freq.py` que contém o top 20 da frequência de cada letra, de cada bigrama e trigama.

Usando os sites da wikipedia para analisar a frequência e o comando `python3 freq.py`, conseguimos desencriptar os caracteres, começando pelo mais frequentes. Para isso usamos o comando `tr`, por exemplo:

```shell
tr 'n' 'E' < ciphertext.txt > partial_plaintext.txt
```

Depois, fomos decifrando um caracter de cada vez (ou 2 caractéres) e colocando em ficheiros diferentes até chegarmos ao ultimo ficheiro com o texto todo encriptado.

![logbook9_t1](img/logbook8_task1.png)

Por fim, vou mostrar 2 parágrafos do texto encriptado:
```
THE OSCARS TURN  ON SUNDAY WHICH SEEMS ABOUT RIGHT AFTER THIS LONG STRANGE
AWARDS TRIP THE BAGGER FEELS LIKE A NONAGENARIAN TOO

THE AWARDS RACE WAS BOOKENDED BY THE DEMISE OF HARVEY WEINSTEIN AT ITS OUTSET
AND THE APPARENT IMPLOSION OF HIS FILM COMPANY AT THE END AND IT WAS SHAPED BY
THE EMERGENCE OF METOO TIMES UP BLACKGOWN POLITICS ARMCANDY ACTIVISM AND
A NATIONAL CONVERSATION AS BRIEF AND MAD AS A FEVER DREAM ABOUT WHETHER THERE
OUGHT TO BE A PRESIDENT WINFREY THE SEASON DIDNT JUST SEEM EXTRA LONG IT WAS
EXTRA LONG BECAUSE THE OSCARS WERE MOVED TO THE FIRST WEEKEND IN MARCH TO
AVOID CONFLICTING WITH THE CLOSING CEREMONY OF THE WINTER OLYMPICS THANKS
PYEONGCHANG
```

## Task2

Nesta tarefa, começamos por gerar um texto com pelo menos 1000 bytes com ajuda de AI e depois guardámos num ficheiro chamado `plaintext.txt`.

Para a criptografia foram usadas algumas `flags`:

- `-ciphertype`: modo da cifra desejada (e.g. aes-128-ecb);
- `-e`: flag para cifrar o ficheiro;
- `-d`: flag para decifrar o ficheiro;
- `-in`: o ficheiro de entrada com o texto original;
- `-out`: o ficheiro de saída contendo o texto cifrado;
- `-K`: a chave de criptografia em hexadecimal;
- `-iv`: o vetor de inicialização em hexadecimal.

Modo `AES-128-ECB`:

Este modo divide o texto em blocos de 16 bytes e cifra cada bloco de forma independente, o que pode resultar numa
repetição de blocos idênticos, revelando padrões. Ele usa um padrão de inicialização para o primeiro bloco.

Para encriptar usamos o comando seguinte:
``` shell
openssl enc -aes-128-ecb -e -in plaintext.txt -out cipher1.bin -K 00112233445566778889aabbccddeeff
``` 
Para desencriptar usámos este comando:
``` shell
openssl enc -aes-128-ecb -d -in cipher1.bin -out decrypted1.txt -K 00112233445566778889aabbccddeeff
```

Modo `AES-128-CBC`:

Este modo também trabalha com blocos de 16 bytes mas cada bloco de texto faz um XOR com o bloco cifrado anterior antes de ser cifrado. Por isso não expõem padrões como o modo anterior, no entanto, se um bloco for corrompido, os próximos também serão.

Para encriptar usamos o comando seguinte:

```shell
openssl enc -aes-128-cbc -e -in plaintext.txt -out cipher2.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

Para desencriptar usámos este comando:

```shell
openssl enc -aes-128-cbc -d -in cipher2.bin -out decrypted2.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

Modo `AES-128-CTR`:

Este modo não depende de blocos anteriores, permitindo processamento paralelo e funciona com tamanhos aleatórios de dados ao contrário do modo anterior.

Para encriptar usamos o comando seguinte:

```shell
openssl enc -aes-128-ctr -e -in plaintext.txt -out cipher3.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```
Para desencriptar usámos este comando:
```shell
openssl enc -aes-128-ctr -d -in cipher3.bin -out decrypted3.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

Para verificar que conseguimos encriptar e desencriptar com sucesso os 3 modos, usámos estes 3 comandos.

![](img/logbook8_task2.png)


## Task5
Nesta tarega, vamos encriptar o ficheiro usado na tarefa anterior `plaintext.txt` nos 4 modos diferentes: ECB, CBC, CFB e OFB.

```shell
# ECB
openssl enc -aes-128-ecb -e -in plaintext.txt -out cipher_ecb.bin -K 00112233445566778889aabbccddeeff
```
```shell
# CBC
openssl enc -aes-128-cbc -e -in plaintext.txt -out cipher_cbc.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```
```shell
# CFB
openssl enc -aes-128-cfb -e -in plaintext.txt -out cipher_cfb.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```
```shell
# OFB
openssl enc -aes-128-ofb -e -in plaintext.txt -out cipher_ofb.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

De seguida, alteramos o byte 250 (50 * 5) em cada ficheiro encriptado. 

Para encontrar o byte certo usamos um programa de desenho para um dos ficheiros e registamos a posição do byte 250 (neste caso foi (4,7)). 

![](img/logbook8_task5.png)


Com isso, conseguimos identificar rapidamente o byte nos outros 3 ficheiros e mudar o mesmo com a ajuda do programa `Bless Hex Editor`.

Depois, desencriptamos os ficheiros.
```shell
# ECB
openssl enc -aes-128-ecb -d -in cipher_ecb.bin -out decrypted_ecb.txt -K 00112233445566778889aabbccddeeff
```
```shell
# CBC
openssl enc -aes-128-cbc -d -in cipher_cbc.bin -out decrypted_cbc.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```
```shell
# CFB
openssl enc -aes-128-cfb -d -in cipher_cfb.bin -out decrypted_cfb.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```
```shell
# OFB
openssl enc -aes-128-ofb -d -in cipher_ofb.bin -out decrypted_ofb.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

Vamos agora registar os resultados para os 4 modos.

### ECB:

![](img/test_ecb.png)

Foram corrompidos 16 bytes (que corresponde a um bloco).

Com base nesta imagem, podemos confirmar que quando um byte é alterado, apenas o bloco onde esse byte está inserido é corrompido, porque neste modo cada bloco é encriptado de forma independente.


### CBC:

![](img/test_cbc.png)

Foram corrompidos 17 bytes.

Com base nesta imagem, podemos confirmar que o bloco com um byte corrompido é alterado e o bloco seguinte também, porque
este modo utiliza o texto encriptado de um bloco para processar o seguinte durante a decifragem.


### CFB:

![](img/test_cfb.png)

Foram corrompidos 15 bytes.

Com base nesta image, podemos afirmar que com o byte corrompido, só é alterado o próprio byte e o bloco seguinte, porque neste modo a propagação é mais ampla do que inicialmente esperado, atingindo o próximo bloco inteiro e não apenas o próximo byte.


### OFB:

![](img/test_ofb.png)

Foi corrompido só 1 byte.

Com base nesta imagem, podemos confirmar que quando um byte é corrompido, só esse byte é alterado quando é desencriptado.
O erro não se propaga porque a keystream gerada é independente do texto encriptado, resultando em apenas um byte corrompido.

