<h3 align="center">	
Escalonamento | MINIX 3 <p>

</h3>

<h1></h1> 

Documentação sobre o sistema operacional Minix 3 [Wiki :scroll:](https://github.com/leonardo8787/minix-1-documentation/wiki/MINIX-3-%7C-Guia) 

Neste repositório iremos dicertar sobre organização estrutural, escalonamento e política nos processos do sistema operacional MINIX 3, desenvolvido pelo professor Tanembaum.
Também iremos falar sobre o mudança na política do sistema além de abrir "issues" falando mais detalhadamente sobre cada processo ao qual fomos submetidos a erros
incluindo a compilação do kernel e a virtualização da ISO gerada no processo. Este repositório tem fins acadêmicos e foi produzido para comunidade do CEFET/MG.

<h2>Processos</h2>

É inevitável falar sobre processos quando estamos tratando sobre escalonamento, há uma convergência nos assuntos, visto que os processos são uma sombra dos escalonadores. Tendo isso em vista é hora de dissertar sobre o que é um processo e como o sistema operacional MINIX 3 gerencia os seus processos, para que possamos 
adentrar nos escalonadores. O sistema operacional MINIX 3 gerencia seus processos através de uma árvore, assim tendo processos filhos, os quais carregam as informações dos processos pais, assim o sistema distribui seus processos de forma que se um deles morrer outras vertentes da árvore, que não estavam conectadas ao processo morto, não morram junto.

<h2>Exemplo de processo em Minix 3</h2>

O exemplo adotado é a inicialização do sistema operacional, ela retrata bem o funcionamento dos processos em Minix 3. Primeiramente, ao inicializar o sistema executará o script/etc/rc para buscar por drivers e servidores. Logo em seguida os drivers e servidores encontrados tornan-se filhos do servidor de inicialização, tendo este processo concluido, o sistema lê a /etc/ttytab para ver quais terminais e terminais virtuais existem na máquina, depois o init bifurca o processo getty, o getty executa um processo de login executando o shell do usuário. Nessa sucessão de acontecimentos é possível ver que o shell é filho do init e neto do inicializador. 

<h2>Implementação do Processo</h2>

Os processos são armazenados em tabelas de processos no MINIX 3, essas tabelas guardam informações desses processos, informações do tipo: Estado do processo, Contador de programa, Ponteiro de pilha, alocação de memória, status de arquivos abertos, informações de confiabilidade, alarmes e outros sinais. Essas tabelas de processos são divididas em 3 sub áreas, que são: Kernel, gerenciamento de processos e por fim gerenciamento de arquivos.

<h3> Design Inteligente </h3> 
 
O MINIX utiliza um design inteligente, dentro do nicho de Sistemas Operacionais, que utiliza como base um Microkernel com 15.000 linhas de código (para comparação, o Kernel do LINUX possui mais de 15 milhões de linhas). Existe uma estimativa de que a cada 1000 linhas de código um a dez bugs podem aparecer, alguns sendo menos sérios como erros de gramática em mensagens. Drivers, por outro lado, possuem a estimativa de terem entre 3 a 7 vezes mais bugs do que o resto do kernel, e 70% do código são drivers. 
 
O MINIX também é altamente modular, o que significa que algumas partes do núcleo do sistema estão em pastas diferentes e independentes, que podem ser chamadas e adicionadas ao sistema durante a execução do mesmo. 
 
Para atingir esse tipo de design, algumas partes foram removidas do kernel e transformadas em módulos. Primeiramente, todos os módulos que precisam ser carregados foram isolados do kernel e passaram para o espaço de usuário, incluindo todos os drivers e sistemas de arquivo. Assim, durante a execução, cada módulo é tratado como um processo diferente e executado utilizando o princípio de menor autoridade (POLA), garantindo que nenhum processo tenha mais poder do que necessário para cumprir sua tarefa, assim tendo também menos poder para causar algum dano ao processo como um todo. Por exemplo, algum port de entrada e saída poderia ter acesso ao disco, talvez causando um bug, mas nesse modelo esse estaria em módulo no espaço de usuário, logo não poderia causar tão problema por falta de acesso. 
 
<h3> Arquitetura em Camadas do MINIX </h3> 
 
O MINIX é composto de quatro camadas principais que formam uma hierarquia de acesso. A primeira camada é o Kernel, que trata serviços de baixo nível necessários para a execução do sistema, como escalonamento de processos, gerenciamento de interrupções, traps, e comunicação entre processos. As camadas acima desta estão no espaço de usuario. 
 
A segunda camada contém os drivers, que são responsáveis pelo funcionamento de dispositivos como disco, terminais, placas de rede, relógio, além de dispositivos E/S. 
 
A terceira é a dos servidores, e contém processos que fornecem serviços úteis para os processos de usuário como, por exemplo, gerenciamento de memória (MM), sistema de arquivos (FS), rede, etc. 
 
Por fim, a quarta camada diz respeito aos processos de usuário como shells, editores e compiladores. 


<h2>Escalonamento</h2>

![Captura de tela de 2022-10-08 12-16-59](https://user-images.githubusercontent.com/78819692/195196906-5d5e0cfd-2cbd-4253-8714-acb1788139fd.png)


O escalonamento do MINIX 3 é uma união entre [múltiplas filas de prioridades](https://pt.wikipedia.org/wiki/M%C3%BAltiplas_filas) sendo que em cada fila é usada a política do [Round-Robin](https://pt.wikipedia.org/wiki/Round-robin). O nível de prioridade da fila é da ordem da menor fila ter a mais alta prioridade ( nível 0) e a maior fila ter a menor prioridade (nível 15).

Normalmente, a fila 0 é referente a processos relacionados ao kernel. Da fila 1 até a fila 4 temos algumas tarefas do sistema e drivers. As filas de prioridade 7 até a 14 é destinada para tarefas de usuários. Por último e de menor prioridade, a fila 15 é responsável pelos processos de [idle](https://github.com/leonardo8787/minix-1-documentation/blob/master/minix/kernel/proc.c#L45) ( “Processos vazios”, no qual quando não há nenhum processo para ser executado, enviamos um processo idle para que o sistema operacional continue mantendo mapeada a arquitetura da máquina).

No MINIX 3 temos o escalonador tanto em espaço de usuário quanto em espaço de núcleo. Na imagem a seguir, temos algumas pastas onde estão localizadas as funcionalidades referente a cada espaço.

### Definições importantes:

[Definição sobre a quantidade de filas de prioridade](https://github.com/leonardo8787/minix-1-documentation/blob/master/minix/include/minix/config.h#L66)

[Definição sobre o quantum](https://github.com/leonardo8787/minix-1-documentation/blob/master/minix/include/minix/config.h#L74)


<h1></h1>

### Escalonador Usuário
![Captura de tela de 2022-10-08 12-40-22](https://user-images.githubusercontent.com/78819692/195197343-185bcd3e-8008-497b-bdfa-0eae2c918ec5.png)

- No espaço de usuário, teremos funções voltadas à prioridade das filas, que como dito anteriormente, é informado que o processo realizou seu quantum, então é feito o decréscimo na prioridade desse processo. Esse espaço de usuário contém também a função de balanceamento, onde a cada tempo de CLOCK definido pelo MINIX, é feito o rebalanceamento entre as filas.

### Algumas funções do escalonador no espaço de usuário:

 [Função do_quantum](https://github.com/leonardo8787/minix-1-documentation/blob/master/minix/servers/sched/schedule.c#L87)  : Função referente ao decrescimo da prioridade do processo após recebé-lo do escalonador no espaço de kernel depois que o processo realiza o seu quantum.
 
 [Função balance_queues](https://github.com/leonardo8787/minix-1-documentation/blob/master/minix/servers/sched/schedule.c#L353) : Função que [a cada clock definido pelo sistema operacional](https://github.com/leonardo8787/minix-1-documentation/blob/master/minix/servers/sched/main.c#L47) irá rebalancear todos os processos.

<h1></h1>

### Escalonador Usuário

![Captura de tela de 2022-10-08 12-39-52](https://user-images.githubusercontent.com/78819692/195197474-fa88105e-0082-49e5-a42c-c53536009442.png)

- No espaço de núcleo, teremos funções responsáveis por gerenciar as filas individualmente, sendo onde ocorre o processo de enfileiramento e desenfileiramento dos processos, e a comunicação com o escalonador no espaço de usuário, como por exemplo o aviso de quando um processo realizou o seu quantum. 

### Algumas funções do escalonador no espaço de núcleo:

[enqueue de processos](https://github.com/leonardo8787/minix-1-documentation/blob/master/minix/kernel/proc.c#L1595) : Função responsável por enfileirar os processos dentro da fila

[dequeue de processos](https://github.com/leonardo8787/minix-1-documentation/blob/master/minix/kernel/proc.c#L1716) : Função responsável por desenfileirar os processos.

[Função pick_proc](https://github.com/leonardo8787/minix-1-documentation/blob/master/minix/kernel/proc.c#L1785) : Essa função seleciona qual o processo dentro da fila que será executado.

<h1></h1>

## Referências

[Documentação Oficial](http://minix3.org/doc/) [1]

[Gerenciador de Dispositivos | UEPG](https://deinfo.uepg.br/~alunoso/2019/SO/MINIX/DISPOSITIVOS/site%20rea/#:~:text=Entrada%20e%20saida%20minix%20No%20Minix%2C%20drivers%20de,pode%20fazer%20e%20aumente%20a%20estabilidade%20do%20sistema) [2]

[Código do MINIX](https://github.com/Stichting-MINIX-Research-Foundation/minix) [3]

#### Autores
	
- Gabriel Henrique Silva Gontijo
- Leonardo de Oliveira Campos
- Lucas Ribeiro Silva
- Thomás Teixeira Pereira

