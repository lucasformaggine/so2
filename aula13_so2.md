# Aula 13 - Sistemas Operacionais II
## Resumo da última aula
- Tipos de alocação de arquivo no volume:
    - I) Alocação contínua
    - II) Alocação encadeada
    - III) Alocação encadeada com estrutura de controle separada
        - Ex: FAT
    - IV) Alocação indexada - implementação no Unix
    - V) Alocação por extensão
- Alocação de arquivos esparsos
## Detecção de blocos livres
- Existe uma estrutura de controle que indica se um bloco está livre ou não
- Existem duas formas de controlar blocos livres:
    - I) Vetor de bits de blocos livres ou bitmap de blocos livres
        
        | 1  | 2  | 3  | 4  | 5  | 6  | 7  | 8  | 9  | 10 | 11 | 12 |
        |----|----|----|----|----|----|----|----|----|----|----|----|
        | 1  | 1  | 1  | 1  | 1  | 1  | 1  | 1  | 1  | 0  | 0  | 0  |
        - 1: Ocupado
        - 0: Livre
        - Em linguagem de máquina, não há uma operação explícita para manipular bit; é necessário combinar operações, como ANDs e ORs
        - Tamanho médio do bitmap:
            - Volume: 1TB
            - Bloco: 1KB
            - Qtd de blocos no volume = 1TB/1KB = 1G blocos
            - Qtd de elementos no vetor de bits = 1G bits
            - Espaço ocupado pelo vetor = 1G bits / 8 = 128MB
        - O espaço é sempre constante, dado o volume e o tamanho do bloco. 
    - II) Lista de blocos livres
        - Lembra a solução de blocos de indireção do Unix
        - Um bloco sabido pelo SO é chamado de bloco de controle e é usado para guardar uma lista do número dos blocos livres
        - Gera uma limitação na quantidade de blocos livres guardadas em um bloco de controle
        - Quando é necessário mais de um bloco de controle, o último campo do bloco de controle guarda a posição de outro bloco de controle para guardar mais números de blocos livres, gerando então uma lista encadeada de blocos de controle
        - Essa ideia de lista encadeada não é usada na alocação indexada no Unix pois seria necessário percorrer o encadeamento para acessar um bloco. Isso também ocorreria na alocação de blocos livres. Uma ideia para evitar isso seria separar o volume do disco em regiões de controle para facilitar saber onde um bloco estaria, mas ainda assim seria necessário varrer.
        - O espaço ocupado pela estrutura de controle varia onforme a quantidade de blocos livres presentes em um volume. Logo, um volume com muitos blocos livres necessitariam de mais espaço para guardar as estruturas de controle, mas em compnesação o disco está vazio; por outro lado, se o disco está cheio, então poucas listas existiriam. Essa vantagem quando o disco está cheio é a grande vantagem com relação à primeira estrutura vista
    - III) FAT
        - Só funciona quando a alocação utiliza FAT
        - Sabe-se que a FAT guarda, em cada posição, o número do próximo bloco daquele arquivo. Quando o valor é -1, significa que não há bloco para um mesom arquivo na posição seuginte. No entanto, como blocos livres não terão posiçã, por padrão elas ficam com 0. Isso é uma forma de detectar blocos livres usando a FAT
        - Existe um bloco livre que armazena a FAT. Ele também é armazenado na própria FAT com valor -2 para indicar que é um bloco de controle
- Existem duas formsa de identificar que um bloco está livre: usando os métodos anteriores, de forma explícita, ou por exclusão, o que não é comumente usado. Como há uma redundância e existe um risco de inconsistência, caso essas informações não batam. Essa problemática é chamada de inconsistência de bloco.
### Inconsistências de bloco
- I) Quando um bloco está presente na estrutura de controle de alocação de arquivo, porém também está marcado como um livre na estrutura de controle de blocos livres.
    - Ex. com alocação indexada e vetor de bits
    - *foto13-1*
- II) Quando um bloco não está presente na estrutura de controle de nenhum arquivo, porém nõa e´tsa marcado como livre na estrutura de controle de blocos livres
    - Ex. com alocação indexada e vetor de bits
    - *foto13-2*
- III) Um bloco está presente nea strutura de controle de acesso de mais de um arquivo, indicando que um arquivo sobrescreveu partes do outro
    - Ex. com alocação indexada
    - *foto13-3*
    - Ocorre por conta do primeiro problema, com como o bloco está marcado como livre na estrutura de controle de blocos livres, existe a possibilidade de um arquivo ser alocado naquele número de bloco
```
OBS (ERRATA): O arquivo B é "4, 5, 6, 9"; o arquivo C é "7, 8" e o arquivo E é "8, 10"
```
- A inconsistência de bloco ocorre pois os momentos em que as informações explícitas ou implícitas sobre o bloco livre são salvas são diferentes. Por exemplo: salvar no vetor de bits e salvar o valor na alocação indexada. Caso haja algum problema interno, o SO pde não ter conseguido salvar em uma das duas, gerando a inconsistência
    - Ex. 1: SO não conseguiu salvar o vetor de bits no disco, mas salvou oi node
    - Ex. 2: SO não conseguiu salvar o inode do arquivo no disco, mas atualizou o vetor de bits
### Soluções para as inconsistências de bloco
- I) Um programa que percorre as estruturas de controle do volume e corrige as inconsistências
    - Solução antiga
    - Precisa ser usado pelo administrador
    - A ideia é sempre supôr que o arquivo existe, mas houve um probelma na hora de salvar a informação. Logo, se o problema foi no controle de blocos livres, a ideia é marcar o bloco como ocupado lá. Ou, se o problema foi na alocação, a ideia é alocar o arquivo para tais blocos inconsistentes. Ou ainda, num terceiro caso, se o SO acabou salvando o bloco para a alocação de dois arquivos, ele copia o conteúdo do bloco repetido para outro bloco e altera um dos copiados para apontar de fato para esse bloco - tipicamente o segundo arquivo copiado foi quem sobrescreveu o primeiro
    - No Unix, para o segundo caso. citado acima, o arquivo é criado em um diretório de achados e perdidos. O programa se chama fsck.
    - No Windows, é criado um arquivo FILE001.CHK na raiz do volume. O programa se chama CHKSDK ("Check Disk"), e precisa rodar com o disco em uso para evitar falso positivo
    - Para que o programa não encontre uma falsa inconsistência (por exemplo, durante o período entre o disco salvar as informações), ele só roda em situações anormais de desligamento. Quando o desligamento é normal, o SO marca no volume. Se o SO percebe que o PC ligou sem essa marcação, ele roda o programa sozinho para verificar as inconsistências
    - O programa pode ficas horas rodando se há muitos arquivos e o volume é grande; como o computador não pode ser usado, acaba se tornando complicado de se usar
- II) Transação para as alterações nas estruturas de controle do SO
    - As alterações feitas em uma transação ou são todas feitas ou nenhuma é feita, sem salvar parcialmente. 
    - Ex: transferência de dinheiro. É necessário decrementar o saldo de uma conta e incrementar outra; se isso não for feito em uma transação, ou o dinheiro é mutuamlmente decrementado e incrementado, ou nada acontece. Se isso for parcial, o dinheiro pode sair de uma conta e ficar perdido, ou quem sabe chegar em uma conta sem sair de outra. Quando o SGBD reinicia, as transações "incompletas" são desfeitas, voltando ao estado original; já as transações terminadas, elas são commitadas no disco
    - Existe um local temporário para guardar as informações durante uma transação antes de serem commitadas. Só com o commit é que as informações vão para o destino final, que nesse caso é o disco. Isso é feito fcom um arquivo de log
## Desempenho em arquivos
- Ex: arquivo A.DAD
```c
int fd, i;
char c;

fd = open("A.DAD", O_RDONLY);

for (i = 0; i < 1024; i++) {
    read(fd, &c, 1);
    // faço alguma coisa com a variável 'c'
}

close(fd);
```
- Embora o loop leia caractere por caractere 1024 vezes, a pergunta "Quantas vezes, durante a execução do loop, haverá a leitura real do disco?" tem como resposta "Uma vez". Isso ocorre pois a leitura real do disco é feita em blocos. Como o bloco já foi lido para o primeiro caractere, ele é guardado na memória do SO para subsequentes leituras, que é no caso das outras 1023 iterações do loop. Obviamente supõe-se que o bloco tem um tamanho [tipicamente mínimo] de 1KB. Se houvesse mais iterações ou tamanho de bloco diferente, o resultado poderia ser diferente.
- O local na memória do SO que guarda os blocos que vieram do disco é o cache de disco
- Ex: arquivo B.DAD
```c
int fd, i;
char c;

fd = open("B.DAD", O_WRONLY);

for (i = 0; i < 1024; i++) {
    // faço um cálculo e atribuo à variável "c"
    write(fd, &c, 1);
}

close(fd)
```
- Embora o loop leia caractere por caractere 1024 vezes, a pergunta "Quantas vezes, durante a execução do loop, haverá a escrita física no disco?" tem como resposta "Depende de como a cache do SO se comporta na escrita". Isso se dá pelo fato de que existem duas situações possíveis
    - Cache 'write through' (cache de escrita através): cada chamada write escreve tanto na cache quanto fisicamente no disco. Nesse caso, a resposta seria que a escrita ocorre 1024 vezes, para cada chamada no disco
    - Cache 'write back' (cache de escrita pelas costas): as chamadas de escrita fazem o SO acumular as alterações somente na memória cache de disco. Nesse caso, a resposta seria 0, pois não há nenhuma chamada física de fato no momento do write. Os blocos alterados somente na cache de disco só serão salvos no disco posteriormente
- Uma pergunta válida para essa situação é "Quando um bloco alterado somente na cache de disco é salvo no disco?". A resposta inclui duas situações:
    - Quando a cache de disco fica cheia, ela precisa liberar a cache para ler um bloco novo. O bloco que sai da cache é chamado de "bloco vítima", e caso só esteja salvo na cache, ele é salvo no disco. O algoritmo para escolha do bloco vítima é o LRU
    - De tempos em tempos, os blocos alterados somente na cache de disco são salvos no disco. Ex: em Unix, é de 30 em 30 segundos
### Cache de disco X Memória virtual (paginação)
- Sobre a cache de disco: o SO usa memória para simular o disco; isso aumenta a velocidade de transferência
- Sobre a memória virtual: o SO usa o disco para simular a memória; isso aumenta a capacidade de armazenamento
- Surge um problema concreto: cnforme os processos vão usando a memória, a memória fica cheia e começa a salvar páginas no disco. Como existem blocos livres na memória de SO para um futuro uso de cache de disco, não faz tanto sentido o SO ficar salvando as páginsa no disco, e sim usar esses blocos livres. 
- Antigamente, quando a memória ficava cheia, o tamanho da cache de disco diminuia para não precisar guardar as páginas no disco e por não estar usando naquele momento os blocos de cache de disco (justamente por estarem livros), mas balancear a velocidade e o limite do encolhimento.
- Hoje em dia, tanto os arquivos quanto as páginas dos processos são vistos como uma coisa só. Dessa forma, tratando tudo como página, o único critério do SO para decidir se a página fica em disco ou não é o NRU, ficando na memória física os conteúdos que tiveram uso recente (podendo ser página de código de processo, página de dados, página de pilha ou página de cache de disco). Dessa forma, independentemente do conteúdo da página, ainda que sejam blocos de cache de disco, o mais justo é guardar o que tem uso recente. As blocos de cache de disco, portanto, possuem uma grnade memória lógica, mas um uso de memória física automaticamente da mesma forma que outras páginas
