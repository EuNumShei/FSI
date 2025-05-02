# CTF 11 - Décima Primeira Semana

## Tarefa 1

O ficheiro ciphersec.py contém 3 funções, que passaremos a explicar:

- gen(): A função gen() gera a chave que será utilizada na função enc() para cifrar a mensagem, sendo que é esta função que contém o problema para a segurança. O problema é que dos 16 bytes de chave gerados, apenas 3 deles são aletórios, sendo os outros apenas `b'\x00'*OFFSET`. Por esta razão, a chave é bastante vulnerável a ataques de força bruta, pois apenas há 256^3 possibilidades diferentes, o que para um computador normal é fazível em minutos.

- enc(k, m, nonce): A função de cifragem usa AES-CTR (Counter Mode), que está bem implementado e supostamente seguro. Contudo, como já vimos, como a chave é vulnerável a ser descoberta, este método é consequentemente inseguro também, pois se sabemos a chave é facilimente decifrável, tendo em conta que o modo de cifrar é público.

- dec(k, c, nonce): A função de decifração segue o mesmo padrão da função de cifragem, mas com o objetivo de reverter o processo e obter a mensagem original. Um atacante utilizará uma função deste estilo para utilizar a chave que obteu e descodificar a mensagem.

Numa situação normal, usaríamos as funções enc() e dec() para codificar e descodificar a mensagem inicial, respetivamente. A chave que ambas utilizam seria fabricada pela função gen().

Dada a vulnerabilidade apresentada, podemos criar um script que automatize o processo de força bruta descrito, que pode parecer com algo deste estilo:

```
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
import os
from binascii import unhexlify

def dec(k, c, nonce):
    cipher = Cipher(algorithms.AES(k), modes.CTR(nonce), backend=default_backend())
    decryptor = cipher.decryptor()
    msg = b""
    msg += decryptor.update(c)
    msg += decryptor.finalize()
    return msg

nonce = unhexlify("4f67ad757631ea84a44df25cd5e9abf5")
ciphertext = unhexlify("772526c4d01ecba6797099e965ebaf5715d373e3a597")

def find_flag():
    for i in range(256**3):
        key = bytearray(b'\x00'*13)
        key.extend(i.to_bytes(3, 'big')) 
        decrypted = dec(bytes(key), ciphertext, nonce)
        try:
            decoded_message = decrypted.decode('utf-8')
            if decoded_message.startswith("flag{") and decoded_message.endswith("}"):
                print(f"Flag encontrada: {decoded_message}")
                return decoded_message
        except UnicodeDecodeError:
            pass
    return None

find_flag()
```

Este script é baseado à volta da função find_flag(), que executa o processo total de encontrar a flag. Esta função usa a função dec() fornecida no ciphersec.py para descodificar a mensagem, descodificando a mensagem com 256^3 chaves diferentes, cobrindo todas as possibilidades. 

Para cada caso, lança um try/catch em que, caso a mensagem descodificada comece com "flag{" e acabe com "}", é a flag pretendida e o programa retorna a flag. Em qualquer outro caso, a mensagem é ignorada e o for loop prossegue. De mencionar também que o nonce e o texto codificado estavam em hexadecimal para leitura mais fácil, sendo que utilizámos unhexlify para os colocar no formato de bytes.

## Tarefa 2

Com as funções fornecidas, o offset neste momento é de apenas 3 bytes, oferencendo apenas 256^3 possibilidades de chaves. Para verificar quantos bytes teria de ter o offset para que um computador demorasse no pior caso 10 anos a obter a resposta (sem paralelização), temos de saber quanto tempo o computador gasta por tentativas, descobrir o número de tentativas em 10 anos, e partir descobrir o número de bytes tal que `256^número > tentativas_em_10_anos`.

Para descobrir o tempo de uma tentativas, podemos simplificar o programa desta forma:

```
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
import os
from binascii import unhexlify
import time

def dec(k, c, nonce):
    cipher = Cipher(algorithms.AES(k), modes.CTR(nonce), backend=default_backend())
    decryptor = cipher.decryptor()
    msg = b""
    msg += decryptor.update(c)
    msg += decryptor.finalize()
    return msg

nonce = unhexlify("4f67ad757631ea84a44df25cd5e9abf5")
ciphertext = unhexlify("772526c4d01ecba6797099e965ebaf5715d373e3a597")

start_time = time.time()

for i in range(256):
    key = bytearray(b'\x00'*16)
    dec(bytes(key), ciphertext, nonce)

end_time = time.time()

time_per_attempt = (end_time - start_time) / 256

print(f"Tempo por tentativa: {time_per_attempt:.8f} segundos")
```

![Imagem_1](img/Imagem11_1.png)

Com este programa, o tempo gasto numa tentativa é calculado a partir do momento em que o nonce e mensagem são reformatadas, até que a função dec() termine a sua execução. Este tempo, tirando a primeira tentativa, revelou-se de aproximadamente **0.00025 segundos**.

Posto isto, e tendo em conta que 10 anos têm **315 532 800 segundos** (assumimos que havia 2 anos bissextos neste intervalo), em 10 anos seria possível executar **1262131200000 tentativas**.

Ou seja, o offset desejado neste caso é `log256(1262131200000)`. Este valor é de 5.02 bytes, ou seja, o offset teria de ter pelo menos **6 bytes** para que as máquinas pessoais usadas demorassem, no pior caso, 10 anos a obter a chave.

## Tarefa 3

Usar um nonce de 1 byte e não o incluir na rede parece uma solução eficaz para evitar os ataques de brute force, contudo este não é por caso por várias razões:

- Primeiro, no modo AES-CTR, o nonce é combinado com o contador para gerar o keystream, sendo esse keystream depois combinado através de um XOR com a mensagem cifrada para formar a cifra. Ou seja, mesmo ele não sendo incluído na rede, ele pode ser descoberto através de técnicas dedução como repetir chaves

- Para além disso, mesmo não se conhecendo o nonce, seria apenas preciso testar as 256 combinações de nonce para além das 256^3 já mencionadas da chave, o que, embora demore mais tempo, ainda é perfeitamente fazível pelas máquinas pessoais.

Devido a isto, utilizar um nonce de 1 byte e não o divulgar na rede não pode ser considerada uma medida eficaz de fortalecimento de segurança. 

## Execução do ataque

Posto isto, ao executarmos o script mencionado, após alguns minutos (isto foi um ataque de força bruta em que assim o resultado pretendido era obtido, o programa parava, logo o tempo necessário era imprevisível e aleatório), obtivemos a flag **flag{cddttsdnzdogsuqg}**, concluído assim o desafio desta semana.

![Imagem_2](img/Imagem11_2.png)