# Aula 4 - Sistemas Operacionais II

### Resumo da última aula
- Falta de memória: soluções
    - Overlay
    - Swap de processos
    - Biblioteca compartilhada
    - Segmentação sob demanda
    - Paginação sob demanda
- Segmentação: 
    - Endereço, em linguagem de máquina, com dois campos:
        - Número do segmento
        - Deslocamento dentro do segmento
    - Tabela de segmentos (por processo):
        - Início do segmento
        - Tamanho do segmento
        - Alterável; executável
    - Passos
        - Identificação do registro na tabela
        - Validações (tamanho, alterável, executável, ...)
        - Soma o início do segmento com o deslocamento
    - Perda de desempenho co a tabela de segmentação
        - Solução Intl: registradores de segmento

### Paginação
- A memória é subdividida em pedaços iguais denominados páginas
- Na Intel, cada página possui 4kb
- As páginas são visíveis através de memórias lógicas ("ilusórias"), onde nelas existe uma alocação contígua e integral dos dados, do código e da pilha; no entanto, essas informações apontam para posições quaisquer da memória, que não precisam mais ter continuidade
- Esse mecanismo precisa ser implementado pelo hardware
- Exemplo de instrução do processo 1
```ass
mov ah, [8200]
```
- Se na memória lógica do processo os dados começam no endereço 8192, o 8200 está
"8" bytes à frente. Isso significa que a página na memória física correpondente deve avançar 8 bytes para chegar ao endereço desejado
- numeroDaPaginaLogica = int(enderecoLogico / tamanhoDaPagina)
- deslocamentoDentroDaPagina = enderecoLogico % tamanhoDaPagina
- enderecoFisico = numeroDaPaginaFisica * tamanhoDaPagina + deslocamentoDentroDaPagina
- Passos da paginação
    - 1) Quebrar o endereço lógico em 2 campos: número da página lógica e o deslocamentro dentro da página
        - Ex: página 2, deslocamento de 8 bytes
    - 2) Converter a página lógica no número da página física
        - Ex: a página lógica 2 é a página física 0
    - 3) Calcular o endereço físico
        - Ex: o endereço físico é 8 bytes

- Para converter a página lógica no número da página física, é necessário um mapeamento que é organizado em uma tabela de páginas para cada memória lógica.
    - Ex: 
        | N° Lógico | Válido | N° Físico | Executável | Alterável |   
        |-----------|--------|-----------|------------|-----------|
        |    13     |   1    |    14     |     0      |     1     |
        |    12     |   1    |    10     |     0      |     1     |
        |    11     |        |           |            |           |
        |    10     |        |           |            |           |
        |     9     |        |           |            |           |
        |     8     |        |           |            |           |
        |     7     |        |           |            |           |
        |     6     |        |           |            |           |
        |     5     |        |           |            |           |
        |     4     |        |           |            |           |
        |     3     |        |           |            |           |
        |     2     |   1    |    0      |     0      |     1     |
        |     1     |   1    |    6      |     1      |     0     |
        |     0     |   1    |    3      |     1      |     0     |

- Páginas lógicas que não estão sendo utilizadas não são endereçadas para nenhum campo na memória física (para que não haja um gasto desnecessário de memória) e são chamadas de "páginas inválidas"
- Para evitar que haja uma tentativa de acesso à uma página inválida, o hardware gera uma interrupção nas tentativas de acesso a partir do bit "válido". Tipicamente, o SO na rotina que trata
esse tipo de interrupção, aborta o processo

### Bits 'Executável' e 'Alterável'
- Tipicamente, os bits 'executável' e 'alterável' estão com valores opostos: se um segmento ou uma página é executável, ela não é alterável (e vice-versa)
- No entanto, é possível que haja simultaneamente um bit executável e alterável ativo

- Linguagem Compilada Pura
    - O compilador converte o código-fonte em linguagem de máquina
    - A CPU executa a linguagem de máquina
    - Mais rápido para ser executado
- Linguagem Interpretada
    - Programa intermediário lê o código-fonte e o interpreta
    - A CPU executa o programa
    - Mais lento para ser executado
- Linguagem Compilada com Interpretação
    - O código-fonte é 'compilado' em uma linguagem intermediária, de mais baixo nível e próximo à linguagem de máquina
    - A linguagem intemediária é interpretada por um programa interpretador
    - A CPU executa o programa interpretador
    - Meio-termo de tempo para ser executado
    - Em algumas linguagens, existe a "compilação just in time", em que métodos e variáveis que são chamados um certo número de vezes passam a ser converitdos em linguagem de máquina, acelerando a execução de um programa para variáveis e métodos comumente chamados
    - É necessário um local especial em memória que possa ser alterado, a partir da conversão de linguagem intermediária em linguagem de máquina, mas também executado, durante a execução de um programa
    - No caso de um código maior do que o tamanho da página, não seria permitida a utilização dessa ideia, já que permitiria a alteração do conteúdo de uma metade da página e a execução da outra metade da página. Para evitar isso, cria-se "espaços vazios" em uma página, mas garante o preenchimento correto da mesma e evita a alteração ou execução de algo indesejado. Os espaços vazios são chamados de 'fragmentação interna'
    - Como a perda mínima por fragmentos intenros é 0 e a perda máxima é o tamanhoDaPagina-1, a perda
    média é de tamanhoDaPagina/2 quando o tamanho do código em memória não é um múltiplo do tamanho da página

    ### Voltando para Paginação
    - O tamanho das páginas é sempre uma potência de base 2, já que realizar divisões com a base utilizada (binária) é mais fácil (analogia: como usamos base decimal, é fácil dividir
    e pegar resto de divisões com 10)
    - Ex: 8200 = 0b0010000000001000 = 2^13 + 2^3 = 8192 + 8 = o8200
        - Dividir por 2^12 e pegar o resto da divisão por 2^12 -> 0b0010 (número da página -> 2) e
        0b000000001000 (deslocamento -> 8)
    - Ao converter o número da página lógica em física e concatenar com o deslocamento, é encontrado
    o endereço da página física. As contas são bem facilitadas
    - A principio, quando há paginação, uma instrução de 'mov' deveria ter dois acessos à memoria: um para realizar as conversões das informações memória lógica (além das validações, a partir da tabela de páginas da memória lógica do processo) e outro para acessar o endereço da memória física
    - Para evitar esse acesso repetitivo, assim como na segmentação, existe uma tabela chamada TLB que resume as informações da tabela de páginas. Ela é presente no hardware, logo seu acesso é mais rápido
        | N° Lógico | N° Físico | Executável | Alterável| 
        |-----------|-----------|------------|----------|
        |     2     |     0     |      0     |     1    |
        |     0     |     3     |      1     |     0    |

    - Caso uma página não esteja na TLB, ela é consultada na tabela de páginas para um primeiro acessado e então é copiada para a TLB. Assim, para próximos acessos, ela é consultada por lá; embora não use um registrador, o fato de não precisar buscar e validar os valores em memória já é um grande ganho de tempo durante a execução
    - Existe um registrador que guarda o endereço da tabela de páginas ativa para que as utilizações sejam corretas durante os eventuais escalonamentos dos processos. Nessas situaçoes, a TLB precisa ser descartada.
    - Caso o hardware possua uma TLB mais completa, ela permite guardar também o número do processo; sendo assim, não é necessário esvaziar a TLB

        | N° Lógico | N° Físico | Executável | Alterável| N° Processo |
        |-----------|-----------|------------|----------|-------------|
        |     2     |     0     |      0     |     1    |      1      |
        |     0     |     3     |      1     |     0    |      1      |
        |     1     |     4     |      0     |     1    |      2      |
        |     13    |     15    |      0     |     1    |      2      |
