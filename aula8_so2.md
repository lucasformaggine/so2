# Aula 8 - Sistemas Operacionais II

### Resumo da última aula
- Algoritmos de escolha de página vítima
    - FIFO
    - Ótimo
    - LRU
    - 2ª chance (do tipo NRU)
- Algoritmos do tipo NRU usam o bit acessado
- Solução qunado o hardware não tem o bit acessado
- Alocação de páginas físicas a processos
    - Alocação global
    - Alocação local
- Para deifnir a quantidade de páginas físicas (alocação local)
    - Working set
    - Princípio da localidade
- Situação ideal: quantidade de páginas físicas reservadas a 1 processo = tamanho do working set do processo
- Implementação por aproximação
    - Observa-se a quantidade de faltas de páginas de um processo)

### Uso da CPU e Thrasing
 
- Gráfico 'Uso da CPU (%) x Qtd. Processos Típicos (que não usam a CPU o tempo inteiro)'
- Na teoria, possui um aumento linear até chegar a 100% de uso
- Com paginação, por haver bloqueios novos para salvar processos em disco, na teoria também é um aumento linear, porém a reta tem um ângulo menor
- Pode ocorrer o "thrasing" (arrastamento): por conta da competição para usar a memória física, o aumento do número de processos faz o uso da CPU se estabilizar em um valor menor do que 100% (ou ainda o uso da CPU cai). Ele costuma acontecer na alocação global, justamente por não haver controle no número de páginas físicas alocadas aos processos. Na alocação local, haveria um limite no número de páginas físicas o que evitaria que um processo atrapalhasse a execução de outro processo, evitando o "thrasing".
- Solução em Unix: swap de processos
    - Embora já haja naturalmente o processo de jogar páginas para o disco, quando há muitos processos típicos se faz necessário também o swap de processos como forma de liberar a CPU

### Segmentação com uso de disco / Segmentação sob demanda
- Ex: tabela de segmentos do processo 3 com duas threads

    | N° segmento | Início do segmento | Tamanho do segmento | Executável | Alterável | Válido | Descrição |
    |-------------|--------------------|---------------------|------------|-----------|--------|-----------|
    |      0      |      200000        |       100000        |     1      |     0     |    1   |  Código   |
    |      1      |      600000        |       100000        |     0      |     1     |    1   |  Dados    |
    |      2      |     1000000        |       100000        |     0      |     1     |    1   |  Pilha 1  |
    |      3      |      400000        |       200000        |     0      |     1     |    1   |  Pilha 2  |

- Caso haja um estouro de pilha de um processo, um segmento será escolhido para ser enviado para o disco, compactando a memória para assim ser possível tratar o esoturo de pilha. Ao fazer isso, o bit 'válido' da tabela de segmentos passa a ser 0 no segmento 'vítima' (3). Com isso, será possível aumentar o tamanho da pilha [ex: do processo 2]

    | N° segmento | Início do segmento | Tamanho do segmento | Executável | Alterável | Válido | Descrição |
    |-------------|--------------------|---------------------|------------|-----------|--------|-----------|
    |      0      |      200000        |       100000        |     1      |     0     |    1   |  Código   |
    |      1      |      450000        |       100000        |     0      |     1     |    1   |  Dados    |
    |      2      |      850000        |       100000        |     0      |     1     |    1   |  Pilha 1  |
    |      3      |      400000        |       200000        |     0      |     1     |    0   |  Pilha 2  |

- Quando o processo 3 precisar usar o segmento 3, o hardware não permitirá por ele não estar em memória (percebido pelo 'bit válido' igual a zero), gerando uma interrupção. No caso de não haver espaço suficiente em memória para voltar com o segmento 3 para memória, um novo segmento será escolhido para ir ao disco. No exemplo anterior, suponha que só haja 170000 de memória livre mas o segmento 3 precisa de 200000. Então, digamos que seja escolhido o segmento 2 do mesmo processo 3. Será então feita uma nova compactação da memória, de modo que o espaço livre será 170000 + 100000 = 270000, permitindo que o segmento 3 volte à memória enquanto o segmento 2 vá ao disco

    | N° segmento | Início do segmento | Tamanho do segmento | Executável | Alterável | Válido | Descrição |
    |-------------|--------------------|---------------------|------------|-----------|--------|-----------|
    |      0      |      200000        |       100000        |     1      |     0     |    1   |  Código   |
    |      1      |      450000        |       100000        |     0      |     1     |    1   |  Dados    |
    |      2      |      850000        |       100000        |     0      |     1     |    0   |  Pilha 1  |
    |      3      |      400000        |       200000        |     0      |     1     |    1   |  Pilha 2  |

- Percebe-se que o problema é parecido com o que ocorre com paginação. No entanto, como o tamanho não é fixo, pode ser que mesmo após jogar o segmento ao disco, ainda não haja espaço contíguo para alocar o segmento, exigindo uma compactação de memória.
- Outro ponto de diferença é que caso o segmento que retorne ao disco seja grande, talvez seja necessário devolvê-lo através de mais do que 1 segmento.
- Enquanto na paginação um processo possua dezenas, centenas ou milhares de páginas, na segmentação um processo possui poucos segmentos (no máximo na casa das dezenas). Isso faz com que haja muito mais chance dele precisar de um segmento que está em disco, prejudicando o desempenho em comparação com a paginação. 

- *FIM DA MATÉRIA DA P1*

### Entrada e Saída

- São feitas pelos módulos do SO chamados 'device drivers'. Servem para intermediar a comunicação
com os dispositivos, tirando a responsabilidade do código com cada dispositivo e evitando erros de programação por usuários normais,
já que exigem um modo de operação privilegiado
- Formas de comunicação com o dispositivo
    - 1) Através de instruções específicas de entrada/saída em linguagem de máquina
        - Ex: Intel
        ```ass
        in registrador_destino, identificador_dispositivo 
        out identificador_destino, registrador_fonte 

        ; Máximo de 8 bytes no envio
        ```

        - OBS: identificador_dispositivo = número de porta        
        - As instruções específicsa de E/S são instruções privilegiadas, portanto
        só podem ser usadas no modo de operação núcleo
    - 2) Através de entrada/saída mapeada na memória
        - Comum em CPUs que não continham instruções específicas para entrada/saída
        - Um intervalo de endereços de memória é reservado para fazer E/S
        - Ex fictício: 
            - Intervalo 1000 a 1999
            - mov [1000], eax ; Equivale a um 'out'
            - mov ebx, [1004] ; Equivale a um 'in'
        - No caso da placa de vídeo, é interessante utilizar esse mapeamento mesmo com instruções específicas, pois ela
        precisaria de uma instrução para o "x", "y" e a "cor" do pixel desejado a ser modificado; com o mapeamento de memória,
        é possível utilizar o endereço de memória mapeando-o para um "(x,y)" especificando, só precisando passar como parâmetro a "cor",
        simplificando esse processo e melhorando o desempenho
- Tipo de informação em uma E/S
    - Dado (entrada e saída)
    - Controle/Ordens (saída)
    - Status (entrada)
- Tipicamente os dispositivos tem pelo menos duas portas: uma porta de dados
e uma porta de controle e status

### Sincronismo CPU x Dispositivo
- A CPU não pode enviar dados para um dispositivo que não é capaz de receber naquele momento
- A CPU não pode mandar ler dados de um dispositivo que não possui dados para serem lidos (caso contrário, seria
lido um dado repetido)
- Portanto, são feitos passos para garantir o sincronismo entre a CPU e o dispositivo:
    - 1) Consultando a porta de status
        - a) Espera ocupada
            - Porta de status: 3F8H
            - Porta de dados: 3F9H
            - Para leitura:
            ```ass
            loop: in eax, 3F8H ; Porta status
                cmp eax, 0 ; Se o status  for 0, o dispositivo não tem dado novo
                jz loop
            in ebx, 3F9H ; Porta dados
            ```
            - Problema: a CPU fica presa no loop enquanto não houver dado novo, usando-a desnecessariamente
        - b) Polling:
            - Verificar o status de tempos em tempos
            - Problema: pode ser que a entrada ou saída esteja pronta em um intrevalo não observado, deixando-o ocioso
    - 2) Por interrupção
        - Código de uma rotina tratadora de interrupção
        ```ass
        push ebx ; Preserva o valor original do registrador ebx
        in ebx, 3F9H ; Porta de dados
        mov [2000], ebx
        pop ebx ; Restaura o valor original do ebx
        iret ; fim do tratamento da interrupção
        ```
        - OBS: O código acima está simplificado. Na realidade, o "in" e o "mov" precisam estar em um loop que rode em um número de vezes
        de acordo com o tamanho desejado do dado que chega - já que o registrador só guarda 4 bytes de dado - além de outras adaptações.
        No entanto, isso só faz sentido para uma quantidade razoável de bytes (dezenas), pois senão o desempenho seria comprometido com tantos loops.
    - 3) Por DMA (Direct Access Memory)
        - OBS: Sem DMA
            - Dispositivos\==== fios ==== [CPU] ===== fios ==== [Memória]
            - O conjunto de fios conectam a CPU à memória chama-se barramento de memória (memory bus)
            - Tipicamente, cada bit é enviado por um fio
            - O conjunto de fios que conectam a CPU aos vários dispositivos chama-se barramento de E/S (I/O bus)
            - Cada dispositivo possui um número de porta, e de acordo com a instrução, é verificado se é um 'in', um 'out' e qual a porta
            correspondente na comunicação; só aí o dado é enviado ou recebido do dispositivo
        - Com DMA, existe um componente de hardware chamado 'controlador de DMA', que se conecta ao barramento de memória e ao barramento
        de E/S. Ele transfere os dados da memória e do dispositivo (seja de entrada ou saída) de forma mais rápida do que a CPU faria pelo loop
        - O controlador de DMA é um pouco mais complexo do que apenas sem o DMA. É necessário uma ordem do SO para o dispositivo, um aviso de que a comunicação será por DMA, uma ordem para o controlador de DMA (para dizer quantos bytes e qual a posição de memória), exeuctando a transferência de dados entre o dispositivo e a memória através do controlador de DMA. Isso permite que a CPU execute outros processos. No entanto, se a CPU precisar ler da memória algo que ainda está sendo transferido pelo controlador de DMA (barramento em uso), há um pequeno gargalo. Ainda assim, é uma boa solução. No fim da transferÊncia dos dados, o controlador de DMA interrompe a CPU para que uma rotina do SO trate essa interrupção, desbloqueando o processo que estava bloqueado aguardando essa tranferência de dados.
