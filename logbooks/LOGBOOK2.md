
# Crafatar 2.1.4 - Path Traversal

## Identificação

- A vulnerabilidade [CVE-2024-24756](https://www.cve.org/CVERecord?id=CVE-2024-24756) afetou a Crafatar, que é uma API que fornece caras para diversos recursos do jogo MineCraft.
- Através de alguns inputs especiais, a pathname que utiliza esses mesmos inputs pode colocar os ficheiros que lhe pertencem diretamente do servidor para fora do domínio da API, expondo as suas informações.

## Catalogação

- Foi catalogada em fevereiro de 2024 por "jomo" (nome do usuário no github)
- Foi considerada de um alto grau de gravidade pela CVE, com um CVSS score de 7.5/10.

## Exploit

- A API utiliza inputs externos para formular o pathname identificador dos ficheiros/pastas provenientes dos seus servidores, sendo que esses ficheiros são sempre pertencentes a um diretório pai.
- Ao não neutralizar certos elementos especiais que podem pertencer ao pathname, esses mesmos elementos podem fazer com que o diretório se instale num lugar fora do domínio do diretório pai.
- Aquando do seu uso sobre a Cloudflare, a vulnerabilidade não é afetada. Contudo, se usada sobre um container Docker, todos as informações daquele container são vulneráveis.


## Ataques

- Nenhum ataque foi documentado, pois a vulnerabilidade foi descoberta por um colaborador da API, tendo sido consertada na versão 2.1.5.
- No caso de um ataque, o atacante poderia ter acesso a diversos tipos de informação confidencial dos sevidores do programa.

### Referências

- https://www.cve.org/CVERecord?id=CVE-2024-24756
- https://www.cvedetails.com/cve/CVE-2024-24756/
- https://github.com/crafatar/crafatar/commit/bba004acc725b362a5d2d5dfe30cf60e7365a373
- https://github.com/crafatar/crafatar/blob/e0233f2899a3206a817d2dd3b80da83d51c7a726/lib/server.js#L64-L67
- https://github.com/crafatar/crafatar/security/advisories/GHSA-5cxq-25mp-q5f2
