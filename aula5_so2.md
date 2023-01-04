# Aula 5 - Sistemas Operacionais II

### Resumo da última aula
- Paginação
    - Motivação
- Divisão da Memória em Páginas
- Memória lógica
    - Páginas vistas em outra ordem
    - Algumas páginas não existem (páginas inválidas)
- Implementação:
    - 1) Quebra do endereço lógico em
        - N° página lógica
        - Deslocamento dentro da página
    - 2) Conversão do número da página lógica em página física; validações
    - 3) Concatenação do número da página física com o deslocamento gerando o endereço físico
- Tabela de páginas (por processo)
    - Número da página física
    - Bits: válido, alterável, executável
- Fragmentação interna
- TLB: Transalation Look Asside Buffer
    - Guarda, em hardware, as informações resumidas da tabela de páginas após um primeiro
    acesso

### Perda de Desempenho
- Exemplo:
    - Tempo de acesso à memória física: 100ns
    - Tempo de acesso à TLB: 10ns

- Em um computador sem paginação, o tempo total de acesso à memória é 100ns
    - tempoTotal = tempoAcessoMemoriaFisica = 100ns
- Em um computador com paginação, o tempo total de acesso à memória é a soma do tempo de conversão do endereço lógico em endereço físico mais o tempo de acesso à memória física
    - tempoTotal = tempoConversaoEnderecoLogicoEmFisico + tempoAcessoMemoriaFisica
- Existem dois casos para a conversão:
    - 1) Os dados da conversão estão na TLB
        - Nesse caso, ele só gasta o tempo de acessar a TLB: 10ns
        - tempoConversaoEnderecoLogicoEmFisico = 10ns
    - 2) Os dados da conversão não estão na TLB
        - Então, ele tentou acessar a TLB, e ainda precisou pegar os dados estão na tabela de paginação
        - tempoConvesaoEnderecoLogicoEmFisico = tempoAcessoTLB + tempoAcessoTabelaPaginas = 10ns + 100ns = 110ns
    - tempoMedioDeAcessoMemoriaLogica = p*10 + (1-p)*110 + 100
    - tempoMedioDeAcessoMemoriaLogica = 10p + 110 - 110p + 100 = 210 - 100p
    - p: probabilidade dos dados da conversão estarem na TLB
- Sempre haverá uma perda de desempenho, mas ela não é tão grave a ponto de não se utilizar a paginação

### Endereços de Memória
- Em CPUs de 32 bits, os endereços possuem 32 bits: o menor endereço é 0 e o maior endereço é 2^32 - 1 = 4G - 1
- OBS: Lembrando que, em computação:
    - 1K = 2^10
    - 1M = 2^20
    - 1G = 2^30
    - 1T = 2^40
- Ex: tamanhoMemoriaLogica = 4GB
- Lembre-se que espacoOcupadoPorRegistro = 4B; tamanhoPagina = 4KB
- qtdPaginasLogicas = tamanhoMemoriaLogica / tamanhoPagina
qtdPaginasLogicas = 4GB/4KB = 2^32 / 2^12 = 2^20 = 1M páginas lógicas
- qtdRegistrosTabelaPaginas = qtdPaginasLogicas
- espacoOcupadoPelaTabPaginas = qtdRegistros * espacoOcupadoPorUmRegistro
- espacoOcupadoPelaTabPaginas = 1M * 4B = 4MB

### Tabela de Páginas em Níveis para CPUs de 32 bits
- Problema da paginação: o espaço ocupado pela tabela de páginas é grande, já que para cada processo, seria necessário alocar esse espaço
- Solução: quebrar a tabela de páginas em pedaços, de modo que pedaços sem pelo menos uma página válida deixarão de existir. Isso é feito com um diretório de páginas que aponta para cada 'pedaço'. Essa estrutura é chamada de 'tabela de páginas em níveis'
- 1M registros / 1K pedaços = 1K registros/pedaço
- espacoOcupadoPeloPedaco = qtdRegistrosDoPecado * espacoOcupadoPorRegistro
- espacoOcupadoPeloPedaco = 1K * 4B = 4KB
- espacoTotalOcupado = espacoOcupadoPeloPedaco * pedacos = 4KB * 3 = 12KB
- À princípio, o tempo de acesso à tabela de páginas na memória se duplicará, pois será necessário primeiro consultar o diretório de páginas para depois acessar o pedaço correspondente
- Para resolver esse problema, há uma divisão do endereço em uma parte para a posição do diretório e outra para a posição no pedaço 

- Endereço lógico de 32 bits

| Posição do diretório de páginas | Posição em um dos pedaços da tabela | Deslocamento |
|---------------------------------|-------------------------------------|--------------|
|           *10 bits*             |         *10 bits*                   |   *12 bits*  |

### Tabela de Páginas em Níveis para CPUs de 64 bits

- No caso de CPUs de 64 bits, o endereço lógico teria 64 bits. Com 52 bits para o número da página lógica, teríamos 26 para identificar o registro no diretório e 26 para identificar um registro no pedaço da tabela, o que daria 2^26 * 4B = 256MB de espaço ocupado pr cada pedaço na tabela de páginas
- Para resolver esse problema, é necessário criar mais níveis, quebrando em pedaços menores
- As tabelas em cada nível (incluindo o diretório de páginas) armazenam 512 registros (2^9)
- Nesse caso, são usados apenas 48 bits: os outros 16 bits ficam com bit 0
- Tentar acessar a página lógica de um pedaço inválido gerará uma interrupção de página inválida, assim como
tentar acessar uma página inexistente de um pedaço válido
- Aplicado nos CPUs da Intel e da AMD

| Pos. diretório de páginas | Pos. tabela 2° nível | Pos. tabela 3° nível | Pos. pedaço | Deslocamento |
|---------------------------|----------------------|----------------------|-------------|--------------|
|           *9 bits*        |       *9 bits*       |        *9 bits*      |  *9 bits*   |  *12 bits*   |

- Seguindo a mesma ideia nas CPUs de 32 bits, existem 4 acessos à memória para o hardware descobrir onde um dado está em memória, o que significa que o tempo de conversão será aumentado

### Tabela de Páginas Invertida

| N° Página Física | N° Página Lógica | N° Processo | Em Uso |
|------------------|------------------|-------------|--------|
|       6          |        2         |     2       |   1    |
|       5          |        0         |     1       |   1    |
|       4          |        1         |     2       |   1    |
|       3          |                  |             |        |
|       2          |                  |             |        |
|       1          |        0         |     2       |   1    |
|       0          |        2         |     1       |   1    |

- A grande vantagem da tabela de páginas invertida é o fato de ser única para cada memória física. Com as informações do número da página lógica, do número do processo e da informação se está em uso, o tamanho é limitado
- O problema, à primeira vista, é encontrar o endereço físico correpsondente de um endereço lógico, visto que ela exige uma busca de 1 em 1 elemento da tabela até bater as informações do número da página lógica e do processo, retornando então o índice de tal linha encontrada
- Para facilitar a procura, é implementada uma tabela hash associada a uma função hash
- f(campo1, campo2, ...) -> inteiro
- A função retorna o número da página lógica mais o número do processo mod 13
- f(x,y) = (x+y)%13
- Exemplos: 
    - f(0,1) = (0+1)%13 = 1%13 = 1
    - f(1,1) = (1+1)%13 = 2%13 = 2
    - f(2,1) = (2+1)%14 = 3
    - f(13,1) = (13+1)%14 = 0
    - f(0,2) = (0+2)%14 = 2
    - f(1,2) = (1+2)%14 = 3
    - f(2,2) = (2+2)%14 = 4
    - f(13,2) = (13+2)%14 = 1 
- O resultado da função hash será o índice na tabela hash que apontara para um número da memória física livre. Ele é guardado tanto na tabela invertida quanto na tabela hash pelo SO no momento que o processo é iniciado

| Índice Hash | Índice da Tabela Invertida |
|-------------|----------------------------|
|   12        |                            |
|   11        |                            |
|   10        |                            |
|   9         |                            |
|   8         |                            |
|   7         |                            |
|   6         |                            | 
|   5         |                            |
|   4         |                            |
|   3         |             0              |
|   2         |             9              |
|   1         |             5              |
|   0         |             12             |

- A partir o número da página lógica, o hardware realiza o cálculo necessário para encontrar o pareamento chave-valor da tabela hash, identificando onde procurar na tabela de página invertida o endereço físico correpsondente
- Caso diferentes páginas resultem em um mesmo valor na função hash, ocorre colisão. Por isso, existe um tratamento de colisão chamada 'lista de colisão' - é uma lista encadeada que esclarece qual é o porcesso de um dado valor hash, além de um campo a mais na tabela de página invertida chamada "próximo da lista de colisão"

| N° Página Física | N° Página Lógica | N° Processo | Em Uso | Próx. da Lista de Colisão |
|------------------|------------------|-------------|--------|---------------------------|
|       12         |        13        |     1       |   1    |           -1              |
|       11         |        13        |     2       |   1    |           -1              |
|       10         |                  |             |        |                           |
|       9          |        1         |     1       |   1    |            1              |
|       8          |                  |             |        |                           |
|       7          |                  |             |        |                           |
|       6          |        2         |     2       |   1    |           -1              |
|       5          |        0         |     1       |   1    |           11              |
|       4          |        1         |     2       |   1    |           -1              |
|       3          |                  |             |        |                           |
|       2          |                  |             |        |                           |
|       1          |        0         |     2       |   1    |           -1              |
|       0          |        2         |     1       |   1    |            4              |

- Caso os dados da tabela para o valor indicado no hash não seja o indicaod na tabela de páginas invertida, é necessário consultar o próximo da lista de colisõa para realizar mais um acesso à memória; isso é feito repetidamente até não haver mais colisões e enfim encontrar o valor desejado
- A escolha da função hash pode ser tal que reduza suficientemente o número de acessos à memória; no pior caso, o número de acessos seria igual ao número de acessos à tabela de 4 níveis, sendo o número de acessos médio menor ao número de acessos à tabela de 4 níveis
- O benefício de desempenho não é tão grande, o que justifica só ser utilizado pela IBM por 2 chips. Ela gera uma melhoria, mas é extremamente complicada de ser implementada.