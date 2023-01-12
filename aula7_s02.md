### Resumo da última aula - SO2
- Compartilhamento de memória na paginação
- Localização do SO na paginação
    - Na sua própria memória física
    - Na memória lógica de todos os processos (bit núcleo)
- Paginação sem uso de disco
    - Necessidade de mais memória física; SO escolhe a página para ser descartada da memória física (página vítima) e esta página fica inválida
    - Futuro acesso a esta página gera interrupção; no tratamento da interrupção, o SO traz esta página de volta à memória
- Problema de desempenho na página usando o disco
- Melhorias na paginação usando od isco
    - Não salvar em disco página vítima de código
    - Não salvar em disco página de variáveis que já tenham sido vítima anteriormente e não foram alteradas (bit sujo)
    - Não alocar (e carregar) na memória física todas as páginas de código quando o processo começa
    - Não alterar na memória física todas as páginas de variáveis quando o processo começa

### Ações para melhorar o desempenho da paginação usando o disco
- I) Não salvar em disco páginas vítimas que sejam de código
- II) Não salvar em disco páginas de variáveis que já tenham sido vítima anteriormente e não foram alteradas desde então
- III) Quando o processo começar, não carregar na memória física as páginas de código (paginação sob demanda / paginação com uso de disco)
- IV) Quando o processo começar, não alocar todas as páginas de variáveis na memória física
- V) Pool de páginas físicas livres
    - O SO mantém uma quantidade mínima de páginas físicas livres. Quando a memória física está quase cheia e o disco onde são guardadas as páginas
vítimas está sem uso, o SO adianta a escolha de páginas vítimas.
    - Ex: Sequência de acesso às páginas lógicas do processo (supondo que a memória física tenha apenas 3 páginas livres)
    - 7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1
    - | t3  | t4  | t5  | t6  | t7  |  t8 | t9  | t10 | t11 | t12 | t13 | t14 |
      |-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
      |  7  |  2  |  2  |  2  |  4  |  4  |  4  |  0  |  0  |  0  |  7  |  7  | 
      |  0  |  0  |  3  |  3  |  3  |  2  |  2  |  2  |  1  |  1  |  1  |  0  |
      |  1  |  1  |  1  |  0  |  0  |  0  |  3  |  3  |  3  |  2  |  2  |  1  |

    - O método acima usado é a eliminação da primeira página da fila (FIFO)
    - 15 interrupções no total  

    - 7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1
    - | t3  | t4  | t5  | t6  | t7  |  t8 | t9  |
      |-----|-----|-----|-----|-----|-----|-----|
      |  7  |  2  |  2  |  2  |  2  |  2  |  7  |
      |  0  |  0  |  0  |  4  |  0  |  0  |  0  |
      |  1  |  1  |  3  |  3  |  3  |  1  |  1  |
    - O método acima é perceber qual página não será usada no futuro próximo. Este algoritmo é chamado de ótimo,
    e ele não existe, pois subentende-se prever o futuro.
    - 9 interrupções no total  

    - 7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1
    - | t3  | t4  | t5  | t6  | t7  |  t8 | t9  | t10 | t11 | t12 |
      |-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
      |  7  |  2  |  2  |  4  |  4  |  4  |  0  |  1  |  1  |  1  |
      |  0  |  0  |  0  |  0  |  0  |  3  |  3  |  3  |  0  |  0  |
      |  1  |  1  |  3  |  3  |  2  |  2  |  2  |  2  |  2  |  7  |
    - O método acima é verifica qual página teve o uso mais antigo, de forma a obter a página "menos recentemente usada" (LRU). Na prática, este algoritmo não pode ser implementado, pois o SO não possui a informação de qual página foi menos recentemente usada - ele simplesmente gera interrupções
    - 12 interrupções o total  

    *copiar-tabela*

    - 14 interrupções no total
    - O algoritmo acima é chamado de algoritmo de Segunda Chance. Ele simula o comportamento do LRU através do bit acessado/usado. O UNix implementa este algoritmo por ordem numérica de acesso, enquanto o Windows faz isso por fila.  

    - Há ainda o algoritmo de relógio: é uma versão do Unix para o algoritmo de Segunda Chance.  

    - Mas e se o SO não tiver o bit acessado/usado? O SO "mente" na tabela de página e "fala a verdade" em uma tabela auxiliar, e com isso, a página ganha uma segunda chance. Após a segunda chance da página lógica 2, quando o processo tentar usá-la, o HW gera uma interrupção, pois o bit válido é 0. No tratamento da interrupção, o SO liga o bit acessado na tabela auxiliar, e coloca o valor verdadeiro do bit válido na tabela principal.
    - Em alguns casos, quando o algoritmo executa com mais páginas físicas, ele gera um resultado pior do que quando executado com menos. É o caso do FIFO e do Segunda CHance. Isso se chama Anomalia de Bellady. 

### Alocação de Páginas Físicas a Processos
    - Alocação global: o algoritmo de escolha de página vítima executa "livremente", podendo escolher página vítima de qualquer processo
    - Alocação local: o algoritmo de escolha de página vítima só escolhe uma página do processo atual
        - Existe uma regra que diz quantas páginas físicas, no mínimo, cada processo deve ter.
        - Processo grande != processo que usa muita memória
        - Dificuldade: conseguir definir quantas páginas físicas cada processo deve ter
        - Solução: Working Set
            - Conjunto de páginas lógicas usadas por um processo durante um certo intervalo de tempo
            - Sequência de acesso às páginas lógicas
            |  1  |  2  |  3  |  4  |  5  |  6  |  7  |  8  |  9  |  10  |  11  |  12  |  13  |  14  |  15  |  16  |  17  |  18  |  19  |  20  |  21  |  22  |  23  |
            |-----|-----|-----|-----|-----|-----|-----|-----|-----|------|------|------|------|------|------|------|------|------|------|------|------|------|------|
            |  5  |  7  |  3  |  5  |  9  |  3  |  7  |  5  |  9  |  5   |  7   |  3   |  2   |  0   |  9   |  3   |  0   |  2   |  9   |  3   |  9   |  2   |  0   |

            - Exemplo: intervalo de observação para definição de working set seja de oito acessos à memória

            - ws(1) = {5}
            - ws(2) = {5, 7}
            - ...
            - ws(8) = {3, 5, 7, 9}
            - ws(9) = {3, 5, 7, 9}
            - ...
            - ws(12) = {3, 5, 7, 9}
            - ws(13) = {2, 3, 5, 7, 9}
            - ws(14) = {0, 2, 3, 5, 7, 9}
            - ws(18) = {0, 2, 3, 7, 9}
            - ws(19) = {0, 2, 3, 9}
            - ...
            - ws(23) = {0, 2, 3, 9}  

        - Princípio da localidade: um processo tende a usar as páginas que já está usando

        - A quantidade de páginas físicas que um processo deve ter é igual ao tamanho de working set do processo

        - Na prática, o working set não é implementado, pois o SO não tem informações para o tamanho deste

        - Quantidade de páginas físicas do processo VERSUS tamanho do working set do processo
            - Se é igual, há poucas interrupções de páginas
            - Se é menor, há muitas interrupções de páginas, e a ação do SO é aumentar a quantidade de páginas fisicas alocadas ao processo
            - Se é maior, há poucas interrupções de páginas, resultantes de páginas que ainda não foram carregadas.
            A ação do SO é, inicialmente, nenhuma; porém, quando o SO não consegue reservar páginas físicas (por exemplo, para um novo processo), o SO
            diminui a reserva de páginas físicas de todos os processos e observa os efeitos. Essa é a lógica que a Microsoft usa no Windows.

### Resumo desta aula

- Algoritmos de escolha de página vítima
    - FIFO, Ótimo, LRU, 2ª chance (do tipo NRU)
- Algoritmos do tipo NRU usam o bit acessado
    - Solução para quando o hardware não tem esse bit: tabela auxiliar
- Alocação de páginas físicas a processos: global vs local
- Para definir quantidade de páginas físicas em alocação local: working set e princípio da localidade
- Situação ideal: quantidade de páginas físicas reservada a um processo igual ao tamanho do working set do processo
- Implementação por aproximação, observando a quantidade de faltas de páginas em um processo

- *gráfico do thrashing*

- Thrashing é mais comum em alocação global, pois não há controle do uso de páginas físicas por processo.
- Solução no Unix: swap de processos. Objetivo: diminuir de N para N-1, que é a melhor situação para uso da CPU
- Efeito prático do thrasing: você mexer o mouse e o ponteiro não mexer; tentar fechar a janela e ela não fechar etc. (comum em versões antigas do Windows)


    

    
     