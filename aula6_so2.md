# Aula 6 - Sistemas Operacionais II

### Resumo da última aula
- Problema do tamanho da tabela de páginas
- Soluções:
    - Tabela de Páginas em Níveis
    - Tabela de Páginas Invertida

### Compartilhamento de Memória na Página
- Embora as memórias lógicas tenham uma "visão" sobre sua própria biblioteca compartilhada, na memória física elas estão apontando para o mesmo lugar. Isso significa que não há duplicação de memória quando se utiliza as memórias compartilhadas ainda que utilizando memória lógica
- O sistema operacional também possui sua própria memória lógica, que contém a 'pilha do SO', os 
'dados do SO' e o 'código do SO' (da mesma forma como um processo em execução)

- Localização do SO
    - I) Na sua própria memória lógica
        - Tratá-lo dessa forma significa ter uma TLB para guardar suas informações; no entanto, toda chamada ao SO (por exemplo no caso de interrupções) faria a TLB ser esvaziada, o que seria extremamente ineficiente. Uma alternativa seria usar a TLB que guarda o número da memória lógica, fazendo-a deixar de ser esvaziada. Por muito tempo, não existia essa coluna para guardar o número da memória lógica, logo essa solução é mais recente.
    - II) Na memória lógica de todos os processos
        - Parte da memória lógica é dedicada ao SO. Embora pareça que ela é duplicada para as memórias lógicas, elas apontam para o mesmo local na memória física. Sendo assim, a TLB só será esvaziada quando mudar o processo em execução; porém, as chamadas ao SO não necessitarão de esvaziamento da TLB. No entanto, parte da memória lógica é perdida para o SO, o que pode ser problemático para uma memória de 32 bits.
        - É necessário um mecanismo para evitar que o processo altere as variáveis do SO, visto que elas estão na mesma memória lógica. Isso é feito através de um bit "núcleo", presente na tabela de páginas. Dessa forma, o hardware consegue identificar, no caso de "1", se uma página é do SO e portanto não pode ser acessada nem editada por um processo (além de estar no modo não privilegiado)
    
    ```
        Relembrando: modo de operação do hardware
            - Usuário: não privilegiado
            - Núcleo ou sistema: privilegiado
    ``` 

- Acessar página inválida gera interrupção; no entanto, quando isso ocorre com uma pilha e justamente em uma página vizinha, o SO entende que é um estouro de pilha, e portanto, caso tenha espaço livre na memória física, ele valida a página vizinha, torna-a alterável e não executável e trata o estouro de pilha alocando mais memória lógica para ela.
- Quando não há memória física livre, a solução é escolher uma página para ser jogado para o disco, "desativando" o bit válido da página correspondente. Isso permite utilizar a memória quando esta está cheia, mas isso gera uma interrupção caso um processo tente acessar uma página recentemente invalidada [por ter sido jogada para o disco]. No entanto, essa situação em que a página é inválida mas existe [em disco] é diferente do caso em que a página é inválida por não existir. 
- Para o SO diferenciar essas situações, ele utiliza um mapa de memória. Se a página não aparece no mapa de memórias, então ela de fato não existe e o hardware geraria a interrupção para impedir o acesso; caso ela exista, então a "invalidação" indica que a memória está em disco, portanto ela pode ser trazida de volta jogando outro processo para o disco. Durante a interrupção, o SO atualiza a tabela de páginas com a nova numeração da página física e com a validação/invalidação das páginas envolvidas na troca e reexecuta a instrução que gerou a interrupção
- Diferentemente do swap de processos, apenas as páginas de um processo são jogadas para o disco; isso permite que um processo continue executando mesmo com uma página em disco, a menos que ele tente acessar uma página inválida


    - Mapa de Memória:

    | Página Inicial | Qtd. de Páginas | Tipo |
    |----------------|-----------------|------|
    |       0        |      2          |   C  |
    |       2        |      1          |   D  |
    |       10       |      3          |   P  |
    |       6        |      2          |   C  |

### Desempenho da Paginação usando o Disco
- Ex:
    - Tempo para acessar a página lógica (quando está na memória física): 100ns = 100 . 10^-9 s = 10^-7 s
    - Tempo para acessar o disco para ler a página: 10ms = 10 . 10^-3 s = 10^-2 s
    - Tempo para acessar a memória lógica (quando a página lógica está na memória física): 100ns = 100 . 10^-9 s = 10^-7 s
    - Tempo para acessar a memória lógica (quando a página lógica está em disco): 10ms = 10 . 10^-3 s = 10^-2 s
    - Tempo médio de acesso à página = p . 100ns + (1-p) . 10ms, onde p = probabilidade da página lógica estar na memória física
    - Tempo médio de acesso à página = 10^-7 . p + 10^-2 - 10^-2 . p

    - Digamos quea meta seja que o tempo méido de acesso à página lógica (na paginação usando o disco) 105 maior que na paginação sem o uso do disco

    - Tempo meta médio = 110ns = 1,1 . 10^-7 s

    1,1 . 10^-7 = 10^-7 . p + 10^-2 . 10^-2 . p

    p = (10^-2- 1,1 . 10^-7)/(10^-2 - 10^-7)

    p = (0,0099999/0,00999989) = 99,9999%

    - Como a diferença de tempo é grande entre o tempo de acessar a memória quando esta está em memória físico em relação a estar em disco, é necessário que na esmagadora maioria das vezes a página esteja em memória

### Ações para melhorar o desempenho da paginação usando o disco

- 1) Não salvar em disco páginas vítimas que sejam de código
    - Como o arquivo executável já contém o arquivo executável, não há por que salvar páginas que guardam códigos em disco; no entanto, é preciso que o SO garanta que o executável não seja apagado durante a execução do processo. No Windows, isso gera um erro; no Unix, ele remove o arquivo do diretório mas o mantém oculto quando o processo morrer
- 2) Não salvar em disco páginas de variáveis que já tenha sido vítima anteriormente e não foi alterada desde então
    - Se a página escolhida para ir ao disco é de dados, pode ser que ela não tenha alteração de contéudo em suas variáveis, o que significa que o valor seria salvo em disco desnecessariamente mais de uma vez; uma forma de solucionar isso é acrescentando um bit chamado "dirty/alterado/sujo" na tabela de páginas. O hardware liga o bit quando verifica alteração e desliga o bit sempre que a página volta à memória (após ir ao disco)
    - Como esse bit é diferente dos outros (por ser alterado pelo hardware, diferente dos outros bits que são alterados pelo SO), surge uma complicação para a TLB: para evitar o acesso desnecessario à memória - o que geraria perda de desempenho e não traria utilidade - a informação do bit "dirty" só é feita na TLB. Ela só vai para a tabela de páginas caso a TLB eventualmente se esvazie
    - Se o hardware não tiver implementado o bit "dirty", como ocorre nos processadores ARM dos celulares, o SO implementa um mecanismo que trabalha em cima do bit "alterável": ele zera esse bit quando a página volta à memória (após ir ao disco). Incialmente, o hardware gera uma interrupção, pois uma página não alterável não pode ser alterada, mas como o SO verifica que ela é uma página de variáveis, ele volta a ser 1, o tratamento da interrupção termina e a instrução que gerou a interrupção é reexecutada. Entretanto, por haver uma interrupção a mais, há uma certa perda de desempenho, mas ainda assim há ganho com relação a salvar em discos
- 3) Quando o processo começar, não carregar na memória física as páginas de código (paginação sob demanda / paginação com uso do disco)
    - Inicialmente, quando o processo começa, as páginas do código estão com o bit válido igual a 0, e portanto não foram carregadas na memória física. Como a execução do código vai cair em uma página inválida na tabela de páginas, surgirá uma interrupção e a própria rotina de interrupção do SO provoca o carregamento do código em memória física. Isso significa que o código é carregado em páginas, conforme as rotinas são chamadas. Por outro lado, não é carregado de 1 em 1 (existe uma empacotação que possibilita um equilíbrio de desempenho e de gerenciamento de memória)
- 4) Quando o processo começar, não alocar todas as páginas de variáveis na memória física
    - O SO adia a alocação de páginas físicas das variáveis até o momento que a página seja utilizada
    - Antes de alocar uma página física com variáveis de um processo, a página que estava lá precisa ser zerada. Em seguida, a memória física correspondente é mapeada. No entanto, a página de dados que não for utilizada não será salvo na memória física.
    

