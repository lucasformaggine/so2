# Aula 12 - Sistemas Operacionais II
## Resumo da última aula
- Como o usuário utiliza arquivos em diferentes volumes?
    - Nomeação explícita do volume
    - Sem nomeação. Unix: montagem. Z/OS: catálogo
- Links simbólicos x Atalhos do Windows
- Chamadas ao SO para arquivos

    ```c
    int fd = open(nome, modo_abertura)
    qtd_lida = read(fd, end_var, qtd_bytes)
    write(fd, end_var, qtd_bytes)
    lseek(fd, nova_pos, a_partir_de_onde)
    ```

- Posição atual do arquivo aberto
    - Variável global que aponta para onde o arquivo está sendo apontado enquanto
    está aberto. Quando aberto, esta posição aponta para o início do arquivo
- O SO garante que, enquanto o arquivo estiver aberto, ele existe. Isso é feito
para evitar múltiplas verificações; ela só precisa ser feita na chamada open()
- Fechar o arquivo garante que os processos passem a usar os arquivos
- Arquivos esparsos
    - Espaço ocupado no disco é menor que o tamanho lógico do arquivo
    - Em arquivos não esparsos, o espaço ocupado no disco é maior ou igual ao tamanho lógico do arquivo (maior quando há fragmentação interna em algum bloco)

## Alocação de arquivos no volume
### Tipos de alocação
- I) Alocação contínua
    - *foto12-1*

    | Nome do Arquivo | Bloco Inicial | Qtd. de blocos |
    |-----------------|---------------|----------------|
    |A|2|3
    |B|5|3
    |C|8|2

    - Nesse tipo de alocação, os arquivos
    precisam estar alocados de forma contínua,
    já que não seria possível com uma única linha e apenas esses campos de bloco e qtd de blocos dizer os diferentes trechos do arquivo
    - Não é possível aumentar o tamanho de um arquivo para além da alocação, pois por ser contínuo, há sempre um arquivo após ele (exceto no caso do último arquivo)
    - A leitura é bem rápida e evita fragmentação externa
- II) Alocação encadeada

    | Nome | Bloco Inicial |
    |A|2
    |B|5
    |C|8

    - Nessa alocação, o bloco possui parte de sua região destinada a armazenar o número do próximo bloco do arquivo. Esse campo só é utilizado e visualizado pelo SO
    - É possível guardar um arquivo de forma não-contínua, pois basta informar no bloco da descontinuidade onde está o próximo. No caso do último bloco, o campo guarda -1
    - Há uma ineficiência para encontrar um bloco do arquivo, pois é necessário varrer todos os blocos anteriores, consultando sempre o próximo até chegar no desejado
    - Nunca foi de fato implementado de forma pura, mas foi usado como base para outra alocação que apresenta consequências mais positivas

- III) Alocação encadeada com estrutura de controle separada 
    - *foto12-3*

    - Foi criado originalmente no MS-DOS e usado em Windows antigos
    - Utiliza uma tabela em memória que guarda, para cada bloco, a posição do próximo bloco. Por estar em memória e ser referenciada através de um ponteiro em algum bloco reservado do disco (no final ou no início), seu acesso é muito rápido
    - É mais rápido percorrer 1000 espaços no vetor em memória do que ler 1 bloco do disco. Com essa alocação, para qualquer acesso ao disco, esse "bloco especial" que guarda o ponteiro só precisará ser acessado uma vez, portanto é rápido e eficiente
    - Na Microsoft, esse vetor se chama File Allocation Table (FAT)
    - Pode ter problemas com volumes muito grandes, pois a FAT ficaria igualemente muito grande. Em disco, isso não seria um problema; no entanto, como a tabela fica em memória, isso geraria uma terrível perda de desempenho. Ex:
        - Volume: 1TB
        - Bloco: 1KB
        - Qtd blocos no volume = 1TB / 1KB = 1 giga de blocos (em base 2)
        - Espaço ocupado pela FAT = Qtd elementos * espaço ocupado por 1 elemento = 1G * 4bytes = 4GB
    - Tipos de FAT: FAT32, FAT16 e FAT12. Isso representa quantos bits são utilizados para armazenar cada elemento na tabela da FAT. Em disquetes, eram usados 16 bits (2 bytes); no Windows, tipicamente se usava 32 bits (4 bytes); em disquetes antigos muito pequenos, se utilizava 12 bits (1 byte e meio) 
    - Hoje em dia, o tamanho do bloco em dispositivos da Microsoft é escolhido de forma automática pelo sistema e normalmente os blocos são grandes para reduzir o espaço ocupado pela FAT. Porém, haverá mais fragmentação interna
    - Ex:
        - Volume: 1TB
        - Bloco: 256KB
        - Qtd. blocos no volume = 1TB / 256KB = 4 mega blocos
        - Espaço ocupado pela FAT = 4M * 4B = 16MB
- IV) Alocação indexada
    - Utilizada no Unix e no Z/OS
    - Há um vetor para cada arquivo que indica, com índices e de forma separada por arquivo, quais blocos estão alocados para o arquivo. Os índices são as possições desses valores de blocos
    - Como apenas uma fração dos arquivos estão sendo usados ("abertos") por fração de tempo, a alocação indexada permite que só seja carregado em memória estruturas de controle que estão em uso. Isso economiza bastante tanto o acesso quanto o uso de memória por parte da estrutura de controle
    - É dificultada pelo tamanho variável dessas estruturas. Arquivos pequenos terão estruturas de controle pequenas, enquanto arquivos grande terão estruturas de controle grande.
    - Para resolver o problema acima, o Unix implementa de modo que parte do vetor seja guardado no inode. No entanto, apenas uma quantidade fixa e pequena de elementos é destinada a guardar os blocos no inode. Quando o arquivo tem mais blocos do que isso, ele guarda "quais são os próximos blocos" em um único bloco no fim da alocação do arquivo - esse bloco se chama bloco de indireção. O número do bloco de indireção também é guardado no inode. Normalmente, esse número é 10, mas pode variar.
    - O problema é que o bloco de indireção possui um número máximo de "valores de blocos" que pode ser armazenado. Ex:
        - Tamanho do bloco = 1KB
        - Espaço de um elemento = 4B (número até 2^32)
        - Qtd. elementos em um bloco de indireção = (espaço ocupado pelo vetor) / (espaço ocupado por um elemento)
        - Qtd. elementos em um bloco de indireção = 1KB / 4B = 256 elementos
    - Caso o bloco de indireção não seja suficiente para guardar o número de blocos de um arquivo, é utilizada uma dupla indireção. Ex:
        - Suponha um arquivo que ocupe 1000 blocos, de 1000 até 2000. No bloco 2001, serão armazenados 256 elementos. Portanto, é preicso 4 blocos de indireção para que seja possível guardar os 1000 elementos, então o bloco 2002 será usado para indicar os blocos de indireção e os próximos blocos serão novos blocos de indireção (2003 e 2004), sendo o último com 232 elementos. Agora, o inode guarda o número do primeiro bloco de indireção mas também o bloco 2002 - chamado de bloco de dupla indireção - que guarda os números de blocos de indireção adicionais
    - O tamanho do bloco de dupla indireção limita a quantidade de blocos de indireção possíveis. Ex:
        - Qtd. elementos no bloco de dupla indireção =  (espaço ocupado pelo vetor) / (espaço ocupado por elemento) = 1KB / 4B = 256 números de blocos de indireção possíveis no bloco de dupla indireção MAIS o bloco de indireção no inode = 257 possíveis blocos de indireção.
        - Qtd. máxima de números de blocos guardados = qtd. de números no inode + qtd. de números no bloco de indireção do inode + (número de blocos de indireção guardados no bloco de dupla indireção * qtd. de números em cada bloco de indireção adicional) = 10 + 256 + (256 * 256) =~ 66000 blocos
        - Tamanho máximo do arquivo = qtd máxima de blocos * tamanho de cada bloco = 66000 * 1KB = 66MB 
    - Como um arquivo pode ser maior do que
    esse limite, a ideia é replicada para um bloco de tripla indireção. De maneira análoga, há um limite. Ex:
        - Qtd. máxima de blocos guardados = 10 + 256 + 256^2 + 256^3 ~= 16M blocos
        - Cada estrutura de um determinado nívle de indireção é o número de blocos do nível anterior que ficam no bloco do nível atual vezes a estrutura do nível anterior
        - Tamanho máximo de cada arquivo = 16M * 1KB = 16GB
    - Embora seria possível prolongar essa ideia para mais indireções, normalmente a implementação para na tripla indireção. Para estender o tamanho máximo de um arquivo, a solução é formatar o disco de modo a aumentar o tamanho do bloco. Ex:
        - Qtd. elementos em bloco de indireção = tamanho do vetor / espaço ocupadlo por um elemento = 2KB / 4B = 512 elementos
        - Qtd. máxima de blocos guardados = 10 + 512 + 512^2 + 512^3 = 128M blocos
        - Tamanho máximo de um arquivo = 128M * 2KB = 128GB
        - Dobrar o tamanho do bloco aumentou em 16 vezes o tamanho máximo do arquivo
    - Hoje em um dia, o tamanho comum de um bloco é 4KB. Existe fragmentação interna em média de 2KB, mas possibilita volumes grandes.
- V) Alocação por extensão
    - Foi inventado no Z/OS (Mainframe) que utilizava alocação contínua: o bloco inicial e a quantidade de blocos
    - O Windows estendeu a ideia para permitir alocação descontínua
    - Como o SO preferencialmente aloca o arquivo de modo contínuo para simplificar e tornar menor a estrutura de controle. No entanto, precisa que haja uma ferramenta de desfragmentação para que os arquivos se tornem contínuos
    - Há um registro na estrutura de controle para cada pedaço contínuo do arquivo, indicando o bloco inicial e a quantidade de blocos. A estrutura é feita para cada arquivo
    - Como não há índice, encontrar um bloco não contínuo é um pouco complicado. Seria necessário ir percorrendo a quantidade de blocos linha por linha até encontrar em qual "continuidade" ele está e então percorrê-la. Para facilitar, o Windows tem mais um campo que contém o número do bloco no arquivo - quase como um índice - e então é feita uma pesquisa binária; ao encontrar o bloco inicial da "continuidade", basta percorrer a diferença para o bloco desejado
    - Em casos que há descontinuidade, a estrutura de controle é maior e o processo se torna mais complexo. Porém, com arquivos contínuos, o processo é mais simples e a estrutura de controle é menor
    - Atualmente, o Linux alterna entre o uso da alocação indexada ou por extensão dependendo da continuidade do arquivo. Há um limite de "pedaços de continuidade" para o Linux atual passar a usar a alocação indexada, porém gasta um certo tempo para alternar entre essas alocações.
### Alocação de arquivos esparsos
- Suponha um arquivo A.DAD de 4KB que teve escrito 1KB após 2KB do fim do arquivo. Dessa forma, sua memória física é 5KB, mas sua memória lógica é 7KB. Este, portanto, é um arquivo esparso, com 2KB esparsos.

```c
fd = open("A.DAD", O_WRONLY);
lseek(fd, 2048, SEEK_END);
write(fd, buf, 1024);
```

- Se cada bloco tivesse 1KB, esse arquivo teira os 4 primeiros blocos físicos existentes, o quinto e sexto blocos apenas lógicos (pois são esparsos) e o sétimo bloco existente
- Na alocação indexada, os índices 5 e 6 são pulados, logo ficam com valor "0" (bloco não válido), mas o índice 7 existe (e representa o bloco com conteúdo escrito no fim do arquivo)
- Na alocação por extensão, só é guardado um registro com o bloco que de fato tem conteúdo. No entanto, não é tão simples ver que o arquivo é esparso olhando para a estrutura de controle. Ele precisa checar se a quantidade de blocos mais o número de bloco de um registro corresponde a um índice presente na estrutura; se não tiver esse índice e o bloco não for o último, é porque esse bloco está esparso
- Caso ocorresse uma desfragmentação, ainda seria visível que o arquivo é esparso pela alocação indexada, pois os índices 5 e 6 continuariam com o 0, mas os demais números de bloco estariam sequenciais. Já na alocação por extensão, haveria 2 registros: a checagem mostraria que o segundo registro está após um certo espaço que é justamente o "pedaço esparso".
