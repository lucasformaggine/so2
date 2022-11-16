# Aula 2 - Sistemas Operacionais II

### Resumo da última aula
- Como conesguir carregar e executar um programa na memória a partir de um endereço que não é previamente conhecido?
- Soluções:
    1) Correção de endereços em tempo de carga
    2) Uso de registrador de base
    3) Uso de endereçamento relativo à instrução atual
    4) Segmentação
    5) Paginação

### Áreas de memória de um processo
    - Código executável
    - Variáveis globais
    - Variáveis locais (pilha)
    - Variáveis dinamicas

#### Variáveis Locais

```cpp
int fatA(int n) {
    int i, acc = 1;

    for (i = 1; i <= n; i++) {
        acc = acc * i;
    }

    return acc;
}

int fatB(int n) {
    if (n > 1) {
        return fatB(n-1) * n
    } else {
        return 1;
    }
}
```

- *foto2-1*

```
...: push eax
...: pop ebx
100: call 500
105: mov eax, 1000
```

- Parte da memória é uma pilha que guarda as variáveis locais (juntamente com os parâmetros) e o endereço de retorno das rotinas. Conforme a execução das rotinas, as variáveis são empilhadas e desempilhadas
- No chipset da Intel, as pilhas crescem de cima para baixo

#### Variáveis Dinâmicas

```c
typedef struct {
    char nome[100];
    int matricula;
    Empregado * prox;
} Empregado;

void a() {
    Empregado * pe1;
    Empregado * pe2;

    pe1 = malloc(sizeof(Empregado))
    pe2 = malloc(sizeof(Empregado))

    pe1->matriclua = 123;
    pe2->matricula = 456;

    // ...

    free(pe1);
}
```

- Enquanto a pilha garante que todos os endereços acima do topo estão usados e abaixo do topo estão livres, a heap - utilizada pelas variáveis dinâmicas - mistura áreas livres e ocupadas, visto que dinamicamente diferentes espaços de memória são alocados ou desalocados (malloc e free), sem seguir a ordem de pilha.
- *foto2-2*
- Linguagens mais antigas precisam do comando 'free' para desalocar, enquanto linguagens mais novas fazem isso por baixo dos panos através de um controle automático de desalocação das variáveis dinâmicas (garbage collector - coletor de lixo)
- Em linguagens orientadas a objeto, os objetos das classes são naturalmente ponteiros, embor anão haja um símbolo explícito que indique isso (como o asterisco, em C e C++)

```java
class Empregado {
    public class nome[100];
    public int matricula;
    public Empregado prox;
}

void A() {
    Empregado e1;
    Empregado e2;

    e1 = new Empregado();
    e2 = new Empregado():

    e1.matricula = 123;
    e2.matricula = 456;
}

void B() {
    Empregado e3, e4;
    e3 = new Empregado();
    e3.matricula = 123;
    e4 = e3;
    e4.matricula = 456;
    System.out.println(e3.matricula);
    // Na função B, o 'ponteiro' e4 aponta para o mesmo local que o 'ponteiro' e3; ao modificar a matricula de e4, o objeto modificado é o mesmo criado por e3 e para onde ele aponta
}
```

- O código acima, embora não mostre de forma explícita, também possui a mesma representação na memória com as estruturas de ponteiros e variáveis dinâmicas, abstraídas pelos conceitos de classe, objeto e atributo da orientação a objeto

### Áreas da memória
- Código
- Pilha (variáveis locais): funções de empilhamento e desempilhamento
- Dados (globais + heap): mesmas instruções (mov)

- Estratégias para organizar o processo na memória:
    - 1) Organização contínua
        - Alocar um espaço para cada processo, subdividindo-o em pilha, dados e código; a pilha e os dados podem ter áreas livres. As áreas sem processo alocado também têm áreas livres
        - É mais simples, porém terá espaços sobrando e pode ser difícil alocar tanto espaço junto
        - Não é possível aumentar o espaço, visto que os blocos de um dado processo são adjacentes
    - 2) Organização não-contínua
        - As áreas de um mesmo processo ficarão divididas na memória
        - É mais complexo, porém utilizará de forma mais econômica os espaços, além de ser mais fácil encontrar espaços livres
        - No caso da pilha, caso o espaço alocado não seja suficiente, o hardware gera uma interrupção para o SO, que tenta aumentar o espaço alocado. Esse espaço precisa ser contínuo com os endereços já alocados da pilha, visto que as operações de push e pop (empilhamento e desempilhamento) decrementam e incrementam o endereço de memória do topo da pilha 
        - No caso da heap, caso o espaço alocado não seja suficiente, o código do processo solicita ao SO espaço livre para aumentar a heap. Isso não precisa ser contínuo, logo o SO pode alocar espaço na memória em outra região que não seja contínua com a atual 
        - No caso do código, pode ser interessante separar a memória que o mesmo utiliza para facilitar a busca por um espaço livre
        - É possível utilizar um registrador de base caso cada área da memória possua um conjunto diferente de operadores em linguagem de máquina; porém, se houver mais de um tipo de uma mesma área da memória de um processo, então não é possível determinar o endereço através de um registrador de base, já que são "n" áreas em diferentes posições
    - *foto2-3*
    - As threads são pontos de execução diferentes em um mesmo código. 

    ```c
    void P() {
        int a;
        // ...
        Q();
        // ...
    }

    void Q() {
        int b;
        // ...
    }

    void R() {
        int c;
        // ...
        S();
    }

    void S() {
        int d;
        // ...
    }
    ```

    - Cada thread precisa ter sua própria pilha para um dado processo, caso contrário elas entrariam em conflito ao empilhar e desempilhar. Veja no exemplo abaixo:
    - Suponha que uma thread 1 (T1) comece rodando a rotina P(), enquanto a thread 2 (T2) comece rodando a rotina R. Suponha que só há uma CPU.
    - Inicialmente a pilha está vazia. Quando T1 passa pela variável 'a', ela é empilada na pilha. Em seguida, T1 é interrompida por time slice
    - T2 é escalonada pelo SO na rotina R() e passa pela variável 'c', que é empilhada na pilha. Em seguida, T2 é interrompida por time slice
    - T1 é escalonada pelo SO e chama a rotina Q(), passando pela variável 'b' que é empilhada, sendo em seguida interrompida por time slice.
    - T2 é escalonada pelo SO e chama a rotina S(), passando pela variável 'd' que é empilhada, sendo em seguida interrompida por time slice.
    - T1 é escalonada pelo SO e, como a rotina Q() terminou, a pilha deveria desempilhar a variável 'b', porém isso não é possível com uma única pilha pois nela já está alocada a variável 'd', que é uma variável local de uma outra rotina lida por outra thread.
    - Quando há várias threads executando um mesmo código, é possível ter um registrador de base, pois em cada instante de tempo apenas uma pilha está em execução, logo ela apontará para quem estiver naquele momento sendo executado. Durante o processo de interrupção por time slice e escalonamento, o registrador de base é reapontado para a pilha que entrará em execução pela thread naquele dado instante de tempo
- Após um determinado tempo, terão áreas livres e áreas alocadas. Caso haja memória livre, mas ela não esteja junta para permitir a alocação de um dado processo, ocorrerá uma fragmentação - ainda que determinadas áreas do processo tenham sido previamente subdivididas, visto que essa subdivisão é feita antes e com um tamanho estático, mas o controle das áreas livres e áreas alocadas varia com o tempo
- No momento que ocorre a fragmentação, para permitir a alocação de áreas de processo, é feita uma compactação, juntando adjacentemente os espaços alocados na memória para uma mesma região "compactada", o que eventualmente fará as áreas livres ficarem juntas e com um tamanho agregado maior
- *foto2-4*
- No entanto, não é possível realizar a compactação em tempo de carga, já que não se sabe os endereços dos ponteiros, logo não se pode acessá-los para corrigir os endereços para onde eles apontam. 
- Por outro lado, a compactação pode ser feita com registrador de base. O valor do registrador de base será modificado pelo SO conforme o novo endereço de memória de cada área de um dado processo, e esse valor será ajustado durante a execução do processo e conforme a compactação
