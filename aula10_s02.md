# Aula 10 - Sistemas Operacionais II
### Resumo da última aula
- Discos magnéticos, divisão
    - Superfícies, trilhas, setores
- Tempo de acesso ao disco
    - Seek + latência + transferência
- Cilindro
- Linearização do disco
    - Vetor de setores
- Desperdício da capacidade de armazenamento [nas trilhas mais externas]
    - Solução: fim do setor circular; tamanhos adaptados dos setores (trilhas externas -> mais setores -> aproximadamente mesma área -> mesma capacidade de armazenamento -> taxa de conversão variável)
- Arquivos
    - Elementos: nome, conteúdo, informações de controle
    - Variações para o conteúdo:
        - Sequência de registros (Z/OS)
            - Sem indexação, precisaria percorrer os arquivos até encontrar o registro ideal
            - Com indexação, era possível anexar campos índice para que o SO acesse esses campos em O(1), porém alta complexidade de implementação
        - Sequência de bytes (Unix, Windows antigos)
            - Simplificação, minimizando as informações de modo que o núcleo do SO não conhece a informação; essa responsabilidade é com o processo que o executa
            - Convencionalmente, a extensão do arquivo é informada no fim do nome do arquivo (.extensao) e a interface com o usuário respeita essa convenção
        - Várias sequências de bytes (Windows novos)
            - É possível programas lerem diferentes conteúdos de um mesmo arquivo
        - Diretórios
            - Organizar arquivos
        
### Diretório
- Tipicamente um diretório é um arquivo
- Conteúdo do diretório pode ser guardado de diferentes formas
    - A) Para cada arquivo no diretório
        - Nome do arquivo
        - Referência às informações de controle do arquivos

        - *foto10-1*
        - *foto10-2*

        - De forma simplificada, o conteúdo do diretório é um vetor de blocos/cluster
        - Aplicados no Unix e Windows
        - No Unix, a referência às informações de controle é chamada de número de inode
        - No Windows, a referência às informações de controle é chamada de posição na MFT (Master File Table)
    - B) Para cada arquivo no diretório
        - Nome do arquivo
        - Dados de controle (de fato, e não referências como no caso acima)
        - Aplicado no MSDos e e Windows antigos
    - C) Para cada arquivo no diretório
        - Nome do arquivo
        - Dados de controle
        - Conteúdo do arquivo
        - Aplicado no Z/OS (Mainframe IBM)
        - Os diretórios, no Z/OS, são chamados de arquivos particionadis

    |               | Mover arquivos| Criar link    |
    |---------------|---------------|---------------|
    |  A            | Muito rápido  |  Funciona     |
    |  B            | Rápido        |  Dá problema  |
    |  C            | Lento         |  É uma cópia  |

    - Mover arquivos dentre diretórios é uma tarefa fácilquando o conteúdo do diretório é apenas o nome do arquivo e a referência às informações de controle; no caso B, também é rápido, mas esse conteúdo das informações de controle está explicitamente indicado. No caso C, aí assim essa transferência é lenta, visto que todo o conteudo do arquivo precisa ser movido
    - A criação de link serve para organizar os arquivos de duas formas diferentes, permitindo-o estra referenciado em diretórios diferentes.
        - Comando para criar link: ln /A/y.txt /B  (cria-se um link, em B, do arquivo y.txt do diretório A)
        - Ex: suponha uma máquina fotográfica digital com cartão de memória. Na hora de transferir os dados do cartão de memória para o computador, o cartão de memória era inserido no computador e as fotos eram copiadas. Algumas ferramentas criam uma árvore de diretórios por data, organizando as fotos dessa forma. Na teoria, isso dificulta o acesso ao arquivo pois é necessário saber qual a data; em um SO que utiliza link dos diretórios, é possível acessar as mesmas fotos através de outras classificações sem gerar espaço gasto duplicado (isto é, sem precisar gerar cópia dos arquivos)
        - No Windows, os arquivos referenciados por link não são mostrados na interface; seriam necessárias extensões ou softwares para permitir a visualização desses arquivos via links dos diretórios. Essas extensões e softwares também facilitam a criação desses links por interface e indicam quantos links cada arquivo tem.
        - No Unix, a uso de links é mais comum e eles já estão presentes por padrão na interface
    - A remoção de um arquivo é feita deletando o link do diretóroi atual. Isso não é feito para todos os diretórios pois não há um ponteiro do arquivo para o diretório, logo seria necessário buscar todos os diretórios para encontrar os links que o arquivo possui para se deletar. Por isso, é mais efetivo só deletar o link do diretório atual e decrementar, nas informações de controle do arquivo, a quantidade de links
        - Comando para remover arquivo: rm /A/y.txt
    - No Windows, não é possível gerar link entre diretórios, pois poderia ser criado um loop de link, o que geraria problemsa recursivos na hora de varrer a árvore de dire´torois
    - No Unix, isso é permitido, mas a 



