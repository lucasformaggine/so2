# Aula 9 - Sistemas Operacionais II

### Resumo da última aula
- Entrada/Saída
    - Formas de Comunicação
        - Instruções epecíficas de E/S
        - E/S mapeada na memória
    - Tipos de informação na E/S
        - Dado (entrada/saída)
        - Controle (saída)
        - Status (entrada)
    - Sincronismo CPU-dispositivo
        - Verificação de status
        - Interrupção
        - DMA
            - Como é vantajoso, a CPU consegue trabalhar com outros processos enquanto há a transferência de dados. No entanto, caso haja um processo aguardando a E/S, é necessário que o processo seja desbloqueado. Como o SO não se envolve no processo do DMA, é necessário uma interrupção [gerada pelo DMA] ao finalizar a transferência de dados da memória para desbloquear um processo que esteja esperando algum dado

### Disco Magnético
- Na parte central no disco, há um eixo rotatório
- O disco é circular e é dividido internamente em anéis concêntricos chamados de trilhas
- Há uma segunda divisão em setores circulares chamados de setores
- A interseção dos setores e das trilhas são trapézios circulares também chamados de setores. Elas são essencialmente as unidades de armazenamento
- Ao salvar algo no disco, pretende-se recuperar esse dado no futuro. É necessário uma garantia de que o conteúdo salvo é igual ao conteúdo lido. Isso é feito através de uma estrutura no final de cada unidade de armazenamento que guarda um código de redundância. Esse código é baseado em um cálculo em cima do dado armzenado e existe uma comparação quando o dado é salvo e quando é lido; caso o valor não seja igual, há dado perdido.
- Para que um dado seja escrito ou lido, há uma cabeça magnética com uma estrutura que desliza entre as trilhas
- Tempo de acesso é a soma de três tempos:
    - Tempo de seek: tempo para movimentar a cabeça até chegar na trilha onde está o setor
    - Tempo de latência: tempo para o disco girar até que o setor comece a passar sob a cabeça
    - Tempo de transferência: tempo para a cabeça passar ao longo do setor
- Cabem destacar algumas grandezas que interferem no tempo de acesso:
    - Velocidade de rotação: em torno de 6000 rpm = 1000 rps = 100 Hz
    - Período de rotação: em torno de 1/100 s = 0,01 s
    - Temp de latência médio: média do tempo de latência no melhor caso (quando a cabeça magnética está no início do setor, logo não há latência) e do pior caso (quando a cabeça magnética está no fim do setor, logo é necessário agurdar uma volta) = (0 + 0,01)/2 = 0,005 s
    - Quantidade de setores por trilha: em torno de 100 setores
    - Tempo de transferência de um setor: tempo de 1 volta / qtd. setores por trilha = 0,01/100 = 0,0001 s
- Tipicamente, os discos magnéticos possuem 2 lados (frente e verso), e consequentemente duas agulhas, uma para cada lado. No entanto, é possível que o disco possua mais "pratos", possuindo uma agulha para cada face de cada prato.
- As agulhas de um mesmo prato se mexem juntos, ainda que uma esteja na parte de cima da face e a outra na parte debaixo.  
- Antigamente, para ler um conteúdo do disco, era necessario que o SO pedisse ao disco três informaççoes: número da superfície, número da trilha e número do esp tor. No entanto, hoje em dia, o SO não dialoga com o dicso em cima desses dados.
- Um conjunto de trilhas com o mesmo raio é chamado de cilindro
- Quando um dado está ocupando mais de um setor, seriam necessários "n" tempos de seek e "n" tempos de latência. Porém, caso os dados a serem lidos/guardados estejam juntos, o gasto de tempo de seek e de tempo de latência é reduzido para um único tmepo. Por isso, a estratégia para facilitar o trabalho do SO para reduzir o tempo de acesso é achar uma forma de guardar os dados de forma sequencial. Quando acabar os setores de uma trilha, o mais estratégico é passar a utilizar, na sequência, a superfície de trás para seguir a sequência em harmonia com o giro do disco, unificando ao máximo o tempo de seek e o tempo de latência durante a leitura
- A visão lógica do disco é comumente um vetor unidimensional de setores, onde os setores são nuemrados em uma trilha no sentido de rotação do eixo.
- Há uma preocupação do SO em salvar arquivos no setor em números contínuos, ainda que possa haver um tempo de seek entre diferentes arquivos
- A largura dos setores é diferente, o que implica na área e consequentemente na capacidade de magnetização, que acaba sendo maior na parte mais externa do disco magnético. Nos discos mais antigos, a mesma quantidade de bytes eram salvas em setores com diferenças satisfatórias entre as larguras, desperdiçando setores maiores. Isso também acontecia por conta da estrutura dos conversores analógico-digital, que tinha taxas de conversão constantes, portanto exigiam que os bytes fossem os mesmos em cada setor. Por outro lado, discos atuais possuem um número variado de setores por trilha: as trilhas mais internas possuem menos setores do que as trilhas mais externas, fazendo as áreas do setores serem praticamente iguais, evitando esse desperdício; nesse caso, são necessários conversores com taxas variáveis de conversão
- É comum agrupar setores do disco através do conceito de blocos. Isso tem uma razão histórica pelo tamanho pequeno dos setores antigamente (cerca de 512b), o que exigiam muitos setores (ex: arquivo de 12kb precisariam de 24 setores). Com o conceito de blocos, o SO consegue alocar com menos complexidade (em especial os blocos possuem 8 setores, que é o mesmo tamanho das páginas)

### Arquivos
- Possuem:
    - Nome
    - Conteúdo
    - Informações de controle
- Variações sobre o nome do arquivo
    - Maiúscula/minúscula:
        - Só maiúsculo (no caso de SOs antigos como MS-DOS, Z/OS Mainframe)
        - Não diferencia maiúscula de minúscula (no caso do Windows)
        - Diferencia maiúscula de minúscula (no caso do Unix)
- Variações sobre o conteúdo do arquivo
    - Sequência de registros (no caso do Z/OS Mainframe)
        - Exigia indexação no sistema operacional para melhorar a performance das buscas dos registros, mas isso tornava o SO mais complexo
    - Sequência de bytes
        - O SO não se preocupa em interpretar o que o arquivo guarda; isso será feito pelo programa que o utilizará
        - Ex: Unix, Windows antigo (até o Windows 98), MS-DOS
        - A interface do usuário é um processo externo ao núcleo do SO, e portanto respeita a extensão através de um programa default que visualiza o arquivo da forma convencionada.
    - Várias sequências de bytes
        - Um mesmo arquivo possui diferentes sequências de bytes independentes que podem ser acessadas através dos dois pontos (ex: notepad a.txt:B, onde B é o nome da sequência)
        - Parte de um arquivo executável é código e parte não é código. Essa parte que não é código é chamada de recurso, o que inclui ícones, textos, cursores diferentes, versões com diferentes traduções, janelas de erro 
        - Não foi muito utilizado, já que o Windows antigo não implementava essa ferramenta e tinha-se a intenção de que o Windows novo rodasse executáveis dos Windows antigos
        - Ex: Windows

### Diretórios (Pastas)
- Objetivo: organizar arquivos
- Tipicamente, um diretório é um arquivo. Isso facilita o fato de que, para dar a capacidade de ter um tamanho dinâmico ao diretório, é mais fácil aproveitar a capacidade já presente nos arquivos, permitindo a criação de uma árvore de diretórios que mistura diretórios e arquivos
- A diferença entre um diretório e arquivo estará fundamentalmente no conteúdo de um diretório. Esse conteúdo varia bastante inclusive entre diferentes SOs