# Aula 3 - Sistemas Operacionais II

### Resumo da última aula
- Áreas de memória de um processo
    - Código
    - Dados
        - Globais
        - Heap
    - Pilha
- Alocação das Áreas
    - Juntas
    - Separadas
- Quantidade de áreas por tipo
- Fragmentação de Memória
- Compactação de memória
    - Nem sempre é possível compactar

### Falta de Memória

- Soluções:
    - I) Overlay
        - Situação:
            - Memória de 64kb
            - SO consome 4kb de memória
            - Temos um porgrama .exe no disco de 80kb
            - A solução é dividir o programa em partes
                - Ex: Dividir o programa em 4 partes (A,B,C,D) de 20kb cada
                - Mantém parte do código em disco
                - Tanto instruções de máquina quanto necessitam ssão carregadas na memória
                - As partes são carregadas na memória no momento em que a parte da memória é necessitada
                - O programador que deveria controlar o overlay
                    - Controlar para antes de chamar uma nova prate, verificar se ela está carregada na memória e, se não estiver, cabe ao programador carregar
                - Ex:
                    ```c
                    if parte_carregada != 'c':
                        carregar_parte('c');
                    r();
                    ```

            - *foto3-1*

    - II) Swap de Processos
        - Situação:
            - Alguns processos em memória, outros em disco
            - Conforme a necessidade de executar um processo que não tem espaço, um processo
            da memória é colocado em disco
            - Quando a execução do processo é interrompida pelo escalonamento de processos, ele vai para o disco e outro processo chega à memória
            - É necessário que o processo vá para o mesmo lugar que estava na memória, pois os ponteiros não poderão ter seus endereços de memória reajustados durante a execução do mesmo
            - Opta-se por processos que estão no estado 'bloqueado' já por muito tempo para irem ao disco, visto que os processos do estado 'pronto' e processos recém bloqueados serão logo executados; enviá-los ao disco poderia gerar uma perda de tempo desnecessária caso sejam executados logo após, pois eles precisarão esperar o tempo de voltar à memória, pois processos em disco não podem ser executados
            - No Unix, existem estados adicionais para facilitar o swap de processos: "bloqueado em disco" e "pronto em disco"; dessa forma, o SO fica com a responsabilidade de levar processos "pronto em disco" para a memória
            - Atualmente não é mais usado por conta da paginação
            - *foto3-2*
    - II.2) Swap de processo no Android
        - Conforme os processos são executados, a memória fica cheia; quando o usuário precisa rodar um processo novo, a solução de swap de processos pode se complicar por conta da relação "pequena" entre o espaço em memória e o espaço em disco em celulares (isto é, o disco pode ficar cheio). 
        - Sendo assim, o SO pede, de tempos em tempos, para o processo salvar em disco dados importantes; assim, quando necessário, o SO mata algum processo que tenha dados salvos em disco.
        - Se o usuário tentar acessar a janela de um processo que já morreu, o SO executa um novo processo a partir do mesmo programa mas solicitando que leia os dados que o processo anterior salvou em disco
        - O espaço em disco é menor pois apenas dados importantes do processo são salvos (no entanto, dependem de rotinas do programador para salvar os dados importantes e chamá-los de volta quando o novo processo é executado)

    - RELEMBRANDO: Estados de um processo
        - Estado Executando -> Estado Bloqueado -> Estado Pronto <-> Estado Executando
        - Uma máquina com 100% de uso da CPU mas com poucos processos no estado 'pronto' não tem gargalo, diferente de uma máquina com 100% de uso da CPU mas com muitos processos no estado 'pronto'

    - III) Biblioteca Compartilhada
        - Nome alternativo no Windows: biblioteca de linkagem dinâmica (Dynamic Link Library -> DLL)
        - O linker, durante a geração normal de um executável subdividido em vários arquivos, preenche as lacunas de "call" de rotinas de outros arquivos, pois a principio não se sabe onde elas estão. Como durante a compilação existe a informação de onde começam as rotinas, o linker utiliza essa informação durante a ligação dos arquivos compilados
        - No caso de rotinas que estão na biblioteca (ex: printf), como as bibliotecas são incorporadas ao arquivo executável, o processo é o mesmo. Porém, se dois executávels usam a mesma biblioteca, ela ficará duplicada na memória
        - A solução é compartilhar a biblioteca, de modo que ela não faz mais parte do executável, mas sim é tratada como um arquivo separado. Por um lado, isso impede que o linker consiga encontrar o endereço das rotinas que não foram preenchidos nos arquivos compilados; portanto, quando o código é rodado em memória, ele não está acoplado à biblioteca, que também é carregado à memória. Apenas nesse momento em que são carregados em memória é que o SO consegue preencher as lacunas do call
        - Torna o código implementado pelo SO mais complexo, visto que o preenchimento das lacunas dos "calls" eram feitos antes pelo linker; porém, realiza um ganho de memória por evitar memória duplicada
        - Ocorre em tempo de carga (embora antigamente ocorria em tempo de execução, no caso de bibliotecas grandes com várias seções e que se fazia necessário, via código, fazer chamadas a seções específicas de acordo com a necessidade, de acordo com a execução do usuários)

        - *foto3-3*

    ### Ponteiro de Memória
    - Existe uma estrutura chamada ponteiro de memória, que impede que o processo acesse memória que não existe. Ele também verifica se o tipo de acesso a um segmento é coerente com esse segmento
    - Existem 3 tipos de acesso a um segmento: consulta, alteração ou execução
    - Uma chamada de "call" para um segmento com o bit "executável" desligado fará com que o hardware gere uma interrupção

    ### Segmentação
    - O endereço (em linguagem de máquina) tem dois campos:
        - Número do segmento
        - Deslocamento no segmento
    - Formato da chamada de rotina: "call 3: 1000" (3 -> número do segmento; 1000 -> deslocamento dentro do segmento)
    - As tabelas de segmentos se encontram no PCB dos processos
    - Ex: tabela de segmentos do processo 1


    |N° | Inicio  | Tamanho   | Executável | Alterável |
    |---|---------|-----------|------------|-----------|
    |0  |700000   |100000     | 1          |     0     |
    |1  |350000   |100000     | 0          |     0     |
    |2  |200000   |100000     | 0          |     1     |
    |3  |850000   |50000      | 1          |     0     |

    - O SO identifica o registro, encontra o início e verifica se o deslocamento pode ser feito (isto é, se ele continuará dentro do segmento ao fazer esse deslocamento)
    - Caso o deslocamento tente acessar um endereço fora do segmento, o hardware gera uma interrução e, tipicamente, mata o processo (Segmentation Fault). Normalmente isso ocorre com chamadas a ponteiros que estejam com valor errado; dificilmente estaria implementado uma chamada para um deslocamento maior que o tamanho dos segmentos (ex: call 3: ebx)
    - Em alguns casos, o processo pode pedir para o SO não abortá-lo e sim rodar alguma rotina programa (ex: caso o Word faça um acesso indevido à memória, é executada uma rotina que tenta encontrar na memória do Word o texto que estava sendo digitado e, caso encontrado, esse texto é salvo em disco em um arquivo temporário, e aí um novo processo é startado com esse conteúdo)

    - *foto3-4*
    
    - O problema de desempenho no acesso à tabela de segmentos é resolvido pela Intel com registradores especiais: os registradores de segmento. Existe um registrador de endereço que aponta para a tabela de segmentos do processo atual. Esse registrador é atualizado conforme o escalonador de processos altera o processo em execução.

    ```ass
    mov ds, 1 % carrega os dados do segmento 1 no registrador de segmentos 'ds'
    mov eax, [ds:100]
    inc eax
    mov [ds:100], eax
    ```

    - No caso de linguagens de alto nível, é esperado que isso seja feito pelo compilador.
    Para evitar inconsistências, boa parte das instruções proibem o uso do segmento explícito para evitar perda de desempenho (ex: 'mov eax, [1:100]' gera erro)