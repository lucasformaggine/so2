# Aula 1 - Sistemas Operacionais II
- Professor: Eduardo Galúcio
- Email: galucio@ime.uerj.br
- Tópicos
    - Gerência de Memória (P1)
    - Entrada e Saída (P2)
        - Disco Magnético
    - Gerência de Arquivos (P2)

- Livros
    - Silberchatz
    - Tanenbaum

## História dos sistemas operacionais  
- Windows  
    - MS-DOS
        - CPU em um único chip, ao invés de placas (na época)
        - Foco técnico
        - Utilizado em computadores IBM-PC
        - Acesso por linha de comando, sem interface visual
        - Executa apenas um processo por vez
    - OS/2
        - Sucessor do MS-DOS
        - Posusi interface visual, mas é pesado
    - Windows 3.0
        - Uso de intreface visual ais leve
        - Popularizou os computadores, sendo mais barato que a concorrência
        - Uso de funções precarias do MS-DOS
    - Windows 95
        - Bootava, na frente dos panos, o DOS e o Windows
    - Windows 98
        - Alguns bugs
    - Windows NT
        - Mesma API (Application Program Interface) do W98
    - Windows XP
    - Windows Vista
    - Windows 7
    - Windows 8
    - Windows 10
    - Windows 11
- Linux
    - Software embarcado normalmente em Linux, por ser de graça
    - Unix(AT&T)
        - Linguagem C foi inventada para dialogar com o hardware e facilitar manutenções
        - Código-fonte aberto, facilitando a resolução de bugs e a evolução do SO, porém a AT&T era dona do código-fonte
        - Passou a vender o Unix após a empresa se separar, por lei
    - Posix
        - Pago
        - Usado pela Apple
    - Linux
        - Criado pelo Linus
        - Gratuito, porém inicialmente sem fabricante
        - Foi criado um mercado em cima da resolução de bugs e evolução do SO, integrando-o a outros programas através de pacotes chamadas distribuições Linux (pagas)
        - Git criado para se usar no Linix
        - Maioria dos servidores passaram a usar Linux
        - Android, roteadores, multimidia de carros rodam em Linux
        - Foi feito um suporte em Linux chamado container, no qual a máquina virtual simula múltiplos SOs em uma máquina física através de um software controlador. Facilita a administração dos SOs, mas divide memória para múltiplos SOs e demora o tempo de boot do SO
        - O container roda múltiplos SOs em um mesmo Linux, permitindo isolar os container sem duplicar memória e bem rapidamente (basta startar o processo)
        - O sucesso dos containers, por ser exclusivo de Linux, populariza ainda mais seu uso em servidores; a tentativa em Windows é startar uma máquina virtual em Linux, tornando o processo lento
    - Berkeley Unix
        - Inicialização do protocolo TCP/IP
        - Rede APPANET -> Internet (junto/para Unix)
    - Android
        - Buscou-se um suporte para, ao desligar a tela, conesguir inativar a CPU, porém rodar programas conforme a necessidade (dados que chegam da rede, por exemplo)
        - Isolamento maior entre os processos de cada usuário: dificulta conflitos e vírus
        - Suporte à memória cheia
- Mainframe IBM
    - Z/OS
        - Suporte a softwares antigos em mainframe
        - Muito usado em bancos
        - Nome original: MVS
        - Feito em Assembly
        - Alta cobertura de testes
        - Projeto falhou com prazo
        - Controle do código-fonte a pratir de versões dos arquivos
## Memória
- Cada região possui 1 byte de espaço e possui um endereço
- Ex: Intel

```
mov ah, [5] 
; Move o conteúdo do endereço de memória 5 para o registrador ah

call 100
; Chama a rotina no endereço 100; digamos que seja "CalculaValor()"
```

- Durante ac ompilação, os nomes de variáveis e rotinas são convertidos em endereços de memória para que a linguagem de máquina entenda (address binding: amarração de endereço)

- *foto1-1*

- Um programa, quanod executado, aloca espaço em memória para sue processo de acordo com suas variáveis alocadas
- Como carregar e executar um programa na memória a partir de um endereço que não é previamente conehcido é um problema, são necessárias soluções que podem ser simples ou sofisticadas

### Soluções para os problemas no address binding
- 1) Correção de endereços em tempo de carga
    - *foto1-2*
    - É feito em memória pelo SO
    - O compilador monta uma tabela que armazena o endereço inicial dos operandos. Quando o SO executa o código, ele varre a tablea e corrige os endereços dos operandos
    - Ex: MS-DOS, programas.exe
- 2) Registrador de base
    - Exige que o SO tenha esse suporte
    - O registrador de base é um registrador especial que o HW utiliza a cada acesso à memória
    - Feito em execução pelo HW
    - O cálculo é feito a cada execução, porém por ser feito com HW, o tempo gasto não é tão at
    - Ex: MS-DOS programas .com (+leve, podia-se somar)
- 3) Uso de endereçamento relativo à instrução atual
    - O valor endereçado, com + ou -, é somado ao endereço da instrução
    - Ex IRREAL (pois Intel não permite com mov):
    ```
    ; 10500 (move algo desse local da memória para bh)
    ; ...
    mov bh, [+400]   ;10100
    ```

    - Ex real:
    ```
    ; ...
    cmp ah, 0 ; 800
    ; ...
    jz 850 ; (jz [+44]) (806)
    ; ...
    ; 809 (primeira instrução do 'then')
    ; ...
    jmp 880 ; (jmp [+35]) 845
    ; ...
    ; 850 (primeira instrução do 'else')
    ; ...
    ; 880 (primeira instrução após o 'if')
    ```

    ```c
    if (i != 0) {
        // parte 'then'
    } else {
        // parte 'else'
    }
    ```

    - Quando escrevemos em linguagens de alto nível, o compilador converte o que for possível para endereçamento relativo
    - OBS: Independente do chipset, isso nunca é feito quando se usa ponteiros

- 4) Segmentação
- 5) Paginação

### Áreas da memória de um processo
- Código executável (ficam abaixo)
- Variáveis (ficam acima)
    - Variáveis globais: usados em mais de uma rotina
    - Variáveis locais: usadas no espaço da rotina onde foram criadas
    - Variáveis dinâmicas: usada por ponteiros; precisa ser explicitamente iniciada
    - Componente de hardware responsávle pelo acesso a memória: Memory Management Unit (MMU)
