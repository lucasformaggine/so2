# Aula 11 - Sistemas Operacionais II
## Resumo da última aula
- Diretórios (ou pastas)
    - Organizar arquivos
    - Tipicamente é um arquivo
    - Conteúdo do diretório (p/ cada arquivo no diretório)
        - A) Nome do arquivo, referência de infos de controle do arquivo
        - B) Nome do arquivo, infos de controle do arquivo
        - C) Nome do arquivo, infos de cointrole, dados do arquivo
- Link
    - Ligação diretório -> arquivo
    - Mais de um diretório pode conter um mesmo arquivo
    - Apagamento de arquivo
    - Link entre diretório
## Disco Físico
    - Partição: pedaço de um disco físico
    - Volume: aquilo que o usuário vê como disco
    - OBS: Normalmente, separar o disco físico em "n" partições significa visualizar "n" volumes diferentes (numa relação 1 pra 1). Porém, é possível fazer essa associação de maneira diferente
    - *foto11-1*
    - No Unix, é necessário baixar um Gerenciador de Volumes
    - No Windows, isso é possível pela interface mas apenas em versões mais robustas/caras do SO
    - Como um usuário utiliza arquivos que estão em diferentes volumes?
        - Usando o nome do volume explicitamente (ex: F:/PastaQualquer/Arquivo.exe ou escolhendo F:/ como o volume atual)
        - Sem usar o nome do volume explicitamente
            - O nome do volume é obtido na criação dos arquivos
                - Usado no Mainframe
                - SO mantem um catálogo geral dos arquivos
                - Limitação: não se pode ter o mesmo nome de arquivo em volumes diferentes
            - O nome do volume é obtido na montagem
                - Usa-se a operação de montagem
                - Quando o computador boota, apenas o volume raíz é visível. Outros volumes podem existir, mas não podem ser usados.
                - Para acessar um arquivo de um volume não visível, aplica-se o seguinte comando no cmd: "mount /dev/hdA2 /B", onde "/dev/hdA2" é o nome do volume e "B" é o diretório onde será mantido o volume
                - *foto11-2*
                - Nessa abordagem, o SO consegue mover os arquivos por baixo dos panos sem que os usuários saibam, pois verão a mesma árvore visível (mas talvez elas estejam em um volume diferente ao longo do tempo)
                - Embora a montagem seja tipicamente do Unix, hoje em dia já é possível fazer montagem no Windows
                - Ainda que o usuário possa ver a árvore visível como uma coisa só [mesmo que haja volumes diferentes representados], também se pode ver a separação dos volumes. Por exemplo: se houvesse uma tentativa e link entre um arquivo de um volume com um diretório do outro, isso não funcionaria, pois o diretório e o arquivo estão em volumes diferentes (até porque, a referência do inode é para o vetor de diretório que está no mesmo volume do diretório). Essa tentativa resulta em um erro e isso indica que os volumes são distintos (ex: "ln /A/x/ /C")
                ```
                - OBS: Existe um outro tipo de link chamado "link simbólico". Ele foi criado para resolver o problema acima. Ele tem uma imlpementação diferente de um link qualquer, mas permite conectar arquivos e diretórios de volumes diferentes.
                - Comando: "ln -s /A/x /C"
                - Esse link é um arquivo texto que é marcado como link simbólico (flag nos dados de controle). Quando o SO detecta que há um link simbólico, ele retorna o conteúdo do arquivo cujo caminho está no link simbólico
                ```
                - Uma questão problemática notável é em relação ao apagamento do arquivo. O contador de links só contabiliza links "normais". Caso o arquivo seja deletado dos diretórios com links normais, ele será de fato apagado. No entanto, o link simbólico continuará existindo, o que significa que tentar acessá-lo nessa situação daria erro de arquivo inexistente (em alguns casos, a interface indica arquivos linkados simbolicamente para que esse comportamento seja previsto)
                - Embora tenha sido criado no Unix, ele também foi implementado no Windows. No entanto, não há programas [default] que façam chamada de link simbólico. Seria necessário baixar um programa para que faça link simbólico em aruqivo (no caso de link simbólico para diretório, há um programa [default] que permite essa criação)
    
### Link Simbólico x Atalho do Windows (.lnk)
- Embora ambos de certa forma possuam o conteúdo do caminho do diretório para onde ambos referenciam
- Diferença: 
    - No link simbólico, o redirecionamento para o arquivo é implementado pelo núcleo do SO, logo ele sempre redirecionará o conteúdo do arquivo apontado pelo link simbólico
    - No atalho, o redirecionamento para o arquivo é implementado pela interface do SO. Logo, um acesso que não seja por interface gráfica - como uma tentativa de editar um atalho por linha de comando - ele editará o conteúdo do próprio atalho; isto é, o caminho para o arquivo ao qual ele aponta.

### Chamadas no SO para usar arquivos
- Dado que os arquivos são sequências de bytes, é possível que o SO chame os arquivos para lê-los integralmente ou apenas parcialmente
- Tentativa de chamada
    - ler("a.dad", pos_inicial, qtd_bytes, endereco_variavel)
    - O "endereco_variavel" se refere a uma variável que receberá o conteúdo do arquivo
    - Outro exemplo:
    ```c
    char buf[10000];

    ler("a.mpg", 0, 10000, buf);
    mostraVideo(buf);
    ler("a.mpg", 10000, 10000, buf);
    mostraVideo(buf);
    ler("a.mpg", 20000, 10000, buf);
    mostraVideo(buf);
    ```
    - No código acima, há uma ineficiência pois a cada chamada "ler()" está sendo verificado a permissão do usuário e verificando se o arquivo existe. No caso em que é o mesmo arquivo sendo executado, como acima, o arquivo é verificado desncessariamente mais de uma vez.
- Conceito de abertura de arquivo
    ```c
    char buf[10000];
    int fd; // File descriptor

    fd = open("a.mpg", O_RDONLY);

    read(fd, buf, 10000);
    mostraVideo(buf);
    read(fd, buf, 10000);
    mostraVideo(buf);
    read(fd, buf, 10000);
    mostraVideo(buf);
    ```

    - Dessa forma, é possível verificar se o arquivo existe e a permissão e ler os dados de controle na chamada open(), retornando um inteiro que, se for positivo, indica que é válido manipular esse arquivo. Esse inteiro é mapeado para um inode em uma tabela do SO. No Windows, ele também faz uma chamada ao SO para impedir que o arquivo seja apagado; no Unix, ele o remove do diretório, mas o permite ocntinuar existindo enquanto o arquivo está aberto
    - Quando o read() é chamado, ele já sabe qual é o arquivo através do mapeamento do fd com a tabela do SO que mapeia o inode correspondente
    - OBS: Arquivos pré abertos para qualquer processo (Unix, Windows)
        - Entrada padrão (fd = 0)
        - Saída padrão (fd = 1)
        - Saída de erro padrão (fd = 2)
    - OBS: A entrada as saídas padrões são arquivos especiais. Escrever na tela ou ler do teclado é como se fosse abrir um arquivo e ler/escrever nele. No Unix, eles estão no diretório /dev/; no Windows, eles não estão em diretório, porém possuem nomes fixos e não podem ser estendidos
    - Em geral, fluxo de dados são enviados por meio de read() e write(); no caso gráfico, por precisar de coordenada, janela e cor, o processo é diferente e mais complexo
    - Para que se possa ler os arquivos em pedaços, é guardado em memória a posição atual dos aruqivos abertos, de modo que chamra um read() para um determinado fd será sempre sequencial
    - Embora o uso parcial do arquivo seja comum através do acesso sequencial (byte a byte, um após o outro em ordem), é possível ler pedaços fora de ordem de um arquivo. Isso ocorre tipicamente quando o arquivo é uma base de dados, como tabela, que é indexado [pelos programas, não pelo SO]. Esse tipo de acesso é chamado de acesso randômico ou acesso aleatório
- Conceito de fechamento do arquivo
    - Através do comando "close(fd)", passando como parâmetro o file descriptor do arquivo, o arquivo deixa de ser usado, liberando o espaço interno [da memória do SO] que guardava informações de controle do arquivo, além de permitir que este seja deletado.
    - Não colocar o close para fechar um arquivo não é grave se o processo já está prestes a acabar
    - OBS: opção de menu "File/Open" é diferente da chamada ao SO "open()". Enquanto a opção de menu coloca todo o arquivo em memória, a chamada open verifica se o arquivo existe, trazendo apenas para a memória dados de controle
- Conceito de escrita em arquivo
    ```c
    char buff[1000];
    // Suponha b.dad um arquivo de 2500 bytes
    fd = open("b.dad", O_RDONLY);
    read(fd, buf, 1000);
    // Posição atual do arquivo: 1001
    ...
    read(fd, buf, 1000);
    // Posição atual do arquivo: 2001
    ...
    read(fd, buf, 1000);
    // Posição atual do arquivo: 2501
    ```
    - Embora na última chamada tenha sido solicitado ler 1000 bytes, o arquivo só tinha 2500 bytes, logo ele só vai ler o que coneseguiu. Sendo assim, o read() retorna a quantidade de bytes lidos.Se a quantidade de bytes lidos é menor do que a quantidade de bytes solicitados para serem lidos, então o arquivo terminou (é um teste possível)

    ```c
    char buff[1000];
    // Suponha c.dad um arquivo de 0 bytes (vazio)
    fd = open("c.dad", O_RDWR);
    write(fd, buff, 1000);
    // Nesse momento, o arquivo "cresce" de tamanho e passa a ter mais 1000 bytes (0 + 1000 = 1000)
    // Posição atual do arquivo: 1001
    ... 
    write(fd, buff, 1000);
    // Como a posição atual é o byte seguinte ao último, ele continuará crescendo (1000 + 1000 = 2000)
    // Posição atual do arquivo: 2001
    ...
    ```
- Conceito de alterar a posição atual do arquivo aberto
    - Para escrever após o fim de um arquivo não vazio ou ler em acesso randômico, é necessário alterar a posição atual do arquivo aberto. Isso é feito com o comando "lseek(fd, nova_pos, a_partir_de_onde)"
    ```c
    // Suponha a.log um arquivo de log já escrito. Ao ser aberto, a posição atual é o byte 0
    fd = open("a.log", O_WRONLY);
    // Jeito mais complicado, precisando do tamanho do arquivo: lseek(fd, tamanho_do_arquivo, SEEK_BEGIN);
    lseek(fd, 0, SEEK_END); // A posição nova conta a partir do fim do arquivo
    write(fd, buf, 1000);
    ```
    - O parâmetro "a_partir_de_onde" tipicamente é o início do arquivo, o fim do arquivo e a posição atual
- No Windows, as chamadas de abertura, leitura, escrita e troca da posição taual dos arquivos são bem parecidas, mas possuem o prefixo "file" (no caso lseek é fileseek) e existe um parâmetro a mais no fileopen do Windows que controla o acesso simultâneo a arquivos
### Arquivo esparso
    ```c
    // Suponha um arquivo de 4kb (4096 bytes) chamado e.dad
    fd = open("e.dad", O_RDWR)
    ;
    lseek(fd, 2048, SEEK_END);
    write(fd, buf, 1024);
    ```
    - No exemplo acima, alguns SOs aceitam que sejam escritos bytes em uma parte após o fim, ficando um espaço vazio que não existe no disco. Caso ele seja lido, ele é lido com bytes zeros, como se não existisse o espaço vazio no disco. No entanto, o espaço ocupado no disco é menor que o tamanho lógico do arquivo. Isto é: ele ocupa 5kb, mas possui um tamanho lógico de 7kb. Esse tipo de arquivo com esse "espaço vazio" é chamdao de arquivo esparso.
    - Em um arquivo não esparso, o espaço ocupado no disco é igual ou maior que o tamanho lógico do arquivo. É trivial quando é igual, mas o espaço pode ser maior pois esse armazenamento é baseado em bloco, que possuem um espaço mínimo. O pedaço não ocupado é uma fragmentação interna
    - Importante distinguir a visão física e lógica do arquivo. Escrever após o fim do arqiuvo não significa roubar espaço de outro arquivo, pois na visão física, os arquivos podem estar completamente dispersos no volume do disco. Enquanto a visão lógica são bytes, a visão física são blocos. Se nada é salvo em um bloco, eles não existem fisicamente. Por outro lado, um bloco ter algum conteúdo implica em ele existir integralmente no disco, o que justifica a fragmentação interna supracitada
    - Os arquivos esparsos podem ser utilizados de forma elegante ou de forma "quebra-galho". 
    - Ex: matriz de inteiros ("m.dad")
        - *foto11-3* (ai,j = 0, exceto a3,3 = 3; a5,2 = 28)
        - Espaço gasto pela matriz = qtd_elementos * espaco_ocupado_por_um_elemento
        - Espaço gasto pela matriz = qtd_linhas * qtd_colunas * espacoOcupadoPorUmElemento
        - Ex: 1024 * 1024 * 4 (numa matriz de inteiros em uma CPU de 32 bits, o elemento inteiro tem 4 bytes) = 4MB

        ```c
        int m[1024][1024];
        fd = open("m.dad", O_WRONLY);

        for (int i = 0; i < 1024; i++) {
            if (linha_so_tem_zeros(m[i])) {
                lseek(fd, 4096, SEEK_CUR);
            } else {
                write(fd, m[i], 4096);
            }
        }
        ```
        
        - Nesse caso, o espaço ocupado em disco é muito menor, pois apenas as linhas com valores diferentes de zero serão escritas. No entanto, os espaços vazios serão lidos, como visto anteriormente.
    - Ex: rotacionar arquivo de log
        - Como os arquivos de log poderiam ficar gigantescos, eles normalmente são programados para serem salvos por dia, e aí caso o disco comece a encher (acima de um dado limite), os arquivos mais antigos são deletados. No Windows, existe uma chamada para destruir um pedaço de um arquivo, liberando o espaço em disco desse pedaço. Logo, para casos em que os arquivos não são rotacionados ou que o log é grande demais, pode-se deletar pedaços indesejados (como o início do arquivo de log) para torná-los esparsos.
        

