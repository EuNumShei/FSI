# CTF 10 - Décima Semana

Este CTF é baseado em Criptografia Clássica, sendo que o objetivo é explorar a fragilidade da cifra clássica fornecida no nosso repositório de forma a conseguir decifrar um criptograma, sem acesso à chave utilizada na sua criação. Esta cifra contém no seu fim algo do formato **{flag_cifrada}**, sendo que a flag esperada é **{flag_decifrada}**.

## Primeiro Passo - Decidir a estratégia de decifração 

A cifra fornecida contém apenas símbolos simples, sendo que cada símbolo representa uma letra do nosso alfabeto. Todo o texto fornecido está cifrado, exceto pelos caracteres `{` e `}`, que encapsulam a nossa flag.

Dada a cifra clássica fornecida, a estratégia que delimitámos e utilizámos com sucesso foi substituir cada caracter (exceto as chavetas curvas) por uma letra correspondente, sendo que esta substituição é baseada na frequência dessas letras na nossa língua, ou seja, os caracteres mais frequentes na nossa cifra estariam associados às letras mais usadas na nossa língua. 

Após isto, utilizaríamos um [substitution decoder](https://www.dcode.fr/substitution-cipher) para substituir os símbolos pelas letras, e um [substitution solver](https://www.guballa.de/substitution-solver) para tentar decifrar a cifra utilizando o texto já substituído. Posto isto, seria só pegar no texto que antes se encontrava dentro das chavetas, e submetê-lo como **flag{texto_decifrado}**.

![Imagem 1](img/Imagem10_1.png)

## Segundo Passo - Substituição dos símbolos por letras

A primeira etapa é contar a frequência de caracter na cifra fornecida. Embora pudéssemos elaborar um programa que fizesse a contagem por nós, como a cifra era pequena, decidimos fazer a contagem à mão. Após isso, pesquisámos as letras mais comuns na língua portuguesa, associando estas letras aos símbolos. No fim obtivemos estes resultados, em que cada linha contém `símbolo inicial/frequência do símbolo/letra correspondente`.

![Imagem 2](img/Imagem10_2.png)

De seguida, visitámos o site já mencionado para [efetuar a substituição](https://www.dcode.fr/substitution-cipher) destes símbolos, resultando no texto: `ARSTSVECAIVEISERAIHEDSBENAORMOEHISNLPTAIORCEIEAUECEEDAVECAPSTSNEMAUOINEMAMARNOIOESRTOUCAIQEROGLDMEUODTEPKEIUADSBEIACIONAMARNOIOESROUCAITLHEPEAROLCIONADENOOGEBODMAERRSUGLDNSADEIRELMEVOPUODTOAUOINEMAONISEDMANADMSNAORMONAUCOTOTSVSMEMOERSDMLRTISERLTSPSBEMAIEROECONLEISENADRLUSMAIEAIOCADREVOSRMANODTIAMEQSAUERREOMEMSIONNEAHOIEPMERGPAIORTERERRSDEIEUADTOUOUNASUQIELUCIATANAPANAUAAQZONTSVAMOORTLMEIAECIAVOSTEUODTAMEQSAUERREGPAIORTEPCEIEGSDRODOIHOTSNARNAUAEPTOIDETSVEEARNAUQLRTSVOSRNADVODNSADESREGEQISNEDENSADZPHSOMHZIUEKCGGV`

![Imagem 3](img/Imagem10_4.png)

## Terceiro Passo - Solução do texto substituído e submissão da flag

Posto isto, utilizámos o site já mencionado para [resolver o texto substituído](https://www.guballa.de/substitution-solver), e ao colocar o texto e escolher a opção para língua portuguesa, obtivémos o texto: `OSITIVAPORVARIASORGANIZACOESDEAGRICULTORESPARAOMAPAANOVAPOLITICADOMERCADODOSCEREAISTEMPORBASEFUNDAMENTALHARMONIZAROPRECODOSCEREAISEMPORTUGALAOSEUPRECONACEEFAZENDOASSIMFUNCIONARSAUDAVELMENTEOMERCADOECRIANDOCONDICOESDECOMPETETIVIDADEASINDUSTRIASUTILIZADORASEAPECUARIACONSUMIDORAOREPONSAVEISDOCENTRODABIOMASSAEDADIRECCAOGERALDASFLORESTASASSINARAMONTEMEMCOIMBRAUMPROTOCOLOCOMOOBJECTIVODEESTUDAROAPROVEITAMENTODABIOMASSAFLORESTALPARAFINSENERGETICOSCOMOALTERNATIVAAOSCOMBUSTIVEISCONVENCIONAISAFABRICANACIONJLGIEDGJRMAHPFFV`, sendo que o excerto que nos interessa é `JLGIEDGJRMAHPFFV`.

![Imagem 4](img/Imagem10_5.png)

Por fim, ao submeter a flag no formato **flag{jlgiedgjrmahpffv}**, ela foi aceite, resolvendo assim o CTF desta semana.