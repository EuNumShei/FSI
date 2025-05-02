# CTF 12 - Décima Segunda Semana

Neste CTF, o objetivo é obter uma mensagem cifrada com RSA. Dado o facto que sabemos em que faixa se encontram os primos **p** e **q** no qual algoritmo do RSA se baseia, vamos tentar utilizá-los para decifrar todo o algoritmo e obter a flag escondida no formato `flag{flag_escondida}`.

## Tarefa 1 - Explicar o processo de encriptação

No enunciado do CTF foi-nos dado o ficheiro gen_example.py, que nos mostra como foi encriptada a flag. Este ficheiro mostra que a flag é uma combinação de 16 letras minúsculas selecionadas de forma aleatória, assim como mostra como todos os atributos necesários para a encriptação com RSA foram formados. Apenas os valores de p,q (primos usados como base do algoritmo, pois são valores privados cuja multiplação forma o modulus n) e d (chave privada formada com base no modulus) não tem a sua formação explícita, sendo este os valores que tentaremos descobrir.

```
def getParams(i, j):
    p = getRandomPrime(i)
    q = getRandomPrime(j)
    n = p*q
    phi = (p-1) * (q-1)
    e = 0x10001 
    d = extendedEuclidean(e, phi) 
    return (p, q, n, phi, e, d)

def enc(x, e, n):
    int_x = int.from_bytes(x, "little")
    y = pow(int_x,e,n)
    return y.to_bytes(256, 'little')

def addFlag(s, f):
	return s+"flag{"+f+"}"

def genFlag():
	return "".join([string.ascii_lowercase[random.randint(0,25)] for _ in range(16)])

def gen():
	f = open("LEICXGY.cph", "w")
	fl = genFlag()
	m = addFlag("", fl)

	(p, q, n, phi, e, d) = getParams(500+offset, 500+offset+1)
	c = enc(m.encode(), e, n)

	f.write("Public exponent: "+str(e)+"\n")
	f.write("Modulus: "+str(n)+"\n")
	f.write(hexlify(c).decode())
	f.close()
```

## Tarefa 2 - Encontrar os valores de p e q

Nesta fase, começamos o processo de desencriptar a mensagem ao encontrar os valores dos primos p e q. Segundo o enunciado, os valores de p e q são próximos de `2^500+(((t-1)*10 + g) // 2)` e `2^500+(((t-1)*10 + g) // 2)` respetivamente, sendo **t** o número da nossa turma (14) e **g** o número do nosso grupo (5). Logo, p será próximo de 2^567 e q próximo de 2^568. 

O enunciado diz também que podemos usar o algoritmo de Miller-Rabin para testar a primalidade de números, sendo que usamos isso mais a condição `p*q == n` para determinar um par de primos que possam ser a solução.

O algoritmo de Miller-Rabin é um teste em que a probabilidade de o resultado estar certo aumenta com o número de iterações feitas. É bastante mais prático para números enormes do que testes determinísticos, devido à sua complexidade muito mais reduzida.

Começa por correr testes iniciais em que trata de todos os casos em que o número é menor que 4 ou par. Depois, fatoriza o número para o formato `d*2^r + 1`, em que começamos por subtrair 1, depois dividir o resultado por 2 até que o número seja ímpar. Dessa maneira, o número de divisões é **r** e o resultado final é **d**. Esta parte é necessária para o passo seguinte.

Por fim, fazemos um número de iterações determinado pelo user como argumento da função, em que cada iteração fazemos `x = a^d mod n`. Se x for 1 ou n-1, avançamos para a próxima iteração. Caso contrário, fazemos `x^2 mod n` r-1 vezes. Se em alguma vez esta número for n-1, passamos para a próxima iteração, senão o número não é primo. Se todas as iterações forem feitas de maneira bem sucedida, o número é considerado provavelmente primo.

O nosso algoritmo para determinar os pares começa por analisar números entre 2^567 e 2^567 + 100000. Após isso verificamos se cada um destes números é primo, e se a divisão entre o modulus e esse número tem resto zero. Se estas condições forem verdade, e o resultado desta divisão também for primo, esse par é retornado como par solução. Este é o código deste algoritmo, que no caso deu p=483067190377157293086918986366498418037365916213304374832154406431439892786195053067024220822740322245307952003937772147170634832630373456967863584183385093587122601854091 e q=966134380754314586173837972732996836074731832426608749664308812862879785572390106134048441645480644490615904007875544294341269665260746913935727168366770187174245203707111. 

```
def is_prime(n, k=40):
    if n <= 1:
        return False
    if n <= 3:
        return True
    if n % 2 == 0:
        return False

    r, d = 0, n - 1
    while d % 2 == 0:
        r += 1
        d //= 2

    for _ in range(k):
        a = randint(2, n - 2)
        x = pow(a, d, n)
        if x == 1 or x == n - 1:
            continue

        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False

    return True

def find_primes(n, offset=67):
    p_start = 2**int(500 + offset)
    q_start = 2**int(501 + offset)

    for candidate_p in range(p_start, p_start + 100000):
        if is_prime(candidate_p) and n % candidate_p == 0:
            candidate_q = n // candidate_p
            if is_prime(candidate_q):
                return candidate_p, candidate_q
    return None, None
```

## Tarefa 3 - Descobrir o d e responder a perguntas

Continuando o algoritmo, a fórmula do d é publicamente conhecida como **d≡1/e modϕ(N)**, sendo **ϕ(N) = (p-1)(q-1)**. Logo, ao simplesmente usar os p e q descobertos anteriormente e substituindo esta fórmulas, chegamos ao valor d=89279581760578228563686811889041480207960340092403430746615035345394567087398329983271465850425249131164702419517288443923108360973186644630430680134154864386632737345078899628964622164834799590701972958532990118010835801900547079810841194541336244745633834793614573624344279005026903676945042945001902167360096451924675346334205774512953373.

```
def compute_private_key(e, p, q):
    phi = (p - 1) * (q - 1)
    d = mod_inverse(e, phi)
    print(f"Computed private key:\n  d = {d}")
    
    return d
```

- Como consigo usar a informação que tenho para inferir os valores usados no RSA que cifrou a flag?

    - Como já demonstrado, a informação dada foi suficiente para obter o p e q, senod que a fórmula do d é pública. Como estes são os únicos termos de origem desconhecida, temos toda a informação necessária para a decifração

- Como consigo descobrir se a minha inferência está correta?

    - Todas as fórmulas se interligam nas outras, e tudo vem dar ao p e q. O d depende do ϕ(N), o ϕ(N) depende do p e q, e sabendo que o modulus é público, se p e q forem próximos dos valores indicados no enunciado, forem primos e a sua multiplicação der o modulus, podemos assumir com bastante certeza que muito provavelmente são os valores certos.

## Tarefa 4 - Obter a flag

Por último, basta-nos usar os dados obtidos e desencriptar a mensagem utilizando este código

```
def decrypt(ciphertext_hex, d, n):
    ciphertext = int.from_bytes(unhexlify(ciphertext_hex), "little")
    plaintext_int = pow(ciphertext, d, n)
    plaintext_bytes = plaintext_int.to_bytes((plaintext_int.bit_length() + 7) // 8, 'little')

    return plaintext_bytes.decode('utf-8')
```

Sendo isto o código completo

```
from random import randint
from math import isqrt
from sympy import mod_inverse
from binascii import unhexlify

def is_prime(n, k=40):
    if n <= 1:
        return False
    if n <= 3:
        return True
    if n % 2 == 0:
        return False

    r, d = 0, n - 1
    while d % 2 == 0:
        r += 1
        d //= 2

    for _ in range(k):
        a = randint(2, n - 2)
        x = pow(a, d, n)
        if x == 1 or x == n - 1:
            continue

        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False

    return True

def find_primes(n, offset=67):
    p_start = 2**int(500 + offset)
    q_start = 2**int(501 + offset)

    for candidate_p in range(p_start, p_start + 100000):
        if is_prime(candidate_p) and n % candidate_p == 0:
            candidate_q = n // candidate_p
            if is_prime(candidate_q):
                return candidate_p, candidate_q
    return None, None

def compute_private_key(e, p, q):
    phi = (p - 1) * (q - 1)
    d = mod_inverse(e, phi)
    print(f"Computed private key:\n  d = {d}")
    
    return d

def decrypt(ciphertext_hex, d, n):
    ciphertext = int.from_bytes(unhexlify(ciphertext_hex), "little")
    plaintext_int = pow(ciphertext, d, n)
    plaintext_bytes = plaintext_int.to_bytes((plaintext_int.bit_length() + 7) // 8, 'little')

    return plaintext_bytes.decode('utf-8')

e = 65537
n = 1519024457286571723257909900287222115391422064333079714236994549458424260073467481189583095617605120199421767864493921122328115578378433216115639431717442130353391
ciphertext = "e7ea678ed36f62d8a0ee6db21c7c038e6a34d2d7c7bdb4d97dd303e4c15d6f812c6908ea0b454210575e5412796371c68b6a23846d21ab62382dc74600302676072e64010000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"

offset = 67
p, q = find_primes(n, offset)
print(f"Found primes:\n  p = {p}\n  q = {q}")
assert p * q == n, "p * q does not equal n!"
assert is_prime(p), "p is not prime!"
assert is_prime(q), "q is not prime!"

d = compute_private_key(e, p, q)

plaintext_bytes = decrypt(ciphertext, d, n)

print(f"Flag found: {plaintext_bytes}")
```

Apenas executar o código, obtivemos a flag **flag{sieumktogjrmtuwg}**, resolvendo assim o último CTF.

![Imagem_1](img/Imagem12_1.png)