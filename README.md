# minix-1-documentation-and-change-politcs
Official MINIX sources - Automatically replicated from gerrit.minix3.org
<h1 align="center"> 
 MINIX 3 | Guia
</h1>

<h4 align="center">	
  Esse projeto é referente a um guia/documentação do sistema operacional Minix 3 ...<p>
  Código do MINIX: https://github.com/Stichting-MINIX-Research-Foundation/minix<p>
</h4>
<h1></h1>

* [Sobre](#Sobre)
* [Instalação](#Instalação)
* [Processos](#Processos)
* [Gerenciamento de Memória](#Gerenciamento-de-Memória)
* [Sistema de Arquivos](#Sistema-de-Arquivos)
* [Entrada e Saída (E/S)](#Entrada-e-Saída-(E/S))
* [Referências](#Referências)
* [Autores](#Autores)

## Sobre 

MINIX 3 é um sistema operacional gratuito, de código aberto, projetado para ser altamente confiável, flexível e seguro. Ele é baseado em um pequeno micronúcleo rodando no modo kernel com o resto do sistema operacional funcionando como uma série de processos isolados, protegidos, no modo usuário. Ele é executado em CPUs x86 e ARM, é compatível com NetBSD e executa milhares de pacotes NetBSD. 

<strong>Sua principais características são:</strong>

* Sistema operacional compatível com POSIX com um usuário NetBSD
* Código aberto, com licença BSD
* Funciona em PCs x86 e também em máquinas virtuais x86 (VMware, etc.)
* Funciona no ARM Cortex A8 (por exemplo, BeagleBoard XM, Bearglebones)
* Networking com TCP/IP
* Memória virtual
* Sistema de arquivos virtuais
* Cache de bloco unificado compartilhado por sistemas de memória virtual e arquivos
* Ligação dinâmica
* Pequena pegada de memória (Kernel é de 600 kB; o SO completo é de 25 MB)

<strong>Recursos específicos do MINIX</strong>

* Micronúcleo minúsculo que funciona no modo kernel
* A maioria do sistema operacional é executada em processos protegidos pelo modo usuário
* Cada driver de dispositivo é um processo separado do modo usuário
* Servidor de reencarnação pode recarregar drives com falha

<strong>Recursos de confiabilidade</strong>

* Tamanho reduzido do kernel
* Insertos são enjaulados
* O acesso à memória dos drivers é limitado
* Referências ruins de ponteiro nem sempre são fatais
* Loops infinitos nem sempre são fatais
* Os excessos de buffer nem sempre são fatais
* O acesso às chamadas de função do kernel é restrito
* O acesso aos portos de I/O é restrito
* A comunicação com componentes do SO é restrita
* Drivers mortos ou doentes podem ser reencarnados
* Interrupções e mensagens são integradas

<strong>Idiomas e Compiladores</strong>

* Idiomas: C, C++, clisp, mawk, Perl, Python, tcl, etc.
* Compiladores: gcc e clang/LLVM
* Compilação nativa (auto-hospedagem) em x86
* Compilação cruzada para x86 e ARM

<strong>Pacotes</strong>

* Conchas (por exemplo, bash, mksh, mudsh, pdksh, zsh)
* Editores (por exemplo, elvis, joe, jove, pico, uemacs, vim)
* Jogos (por exemplo, astutos, exchess, ioquake)
* Correio (por exemplo, buscar correio, getmail, vira-lata, thunderbird)
* Mais de 4000 outros pacotes NetBSD

### Visão Geral 

Os sistemas operacionais monolíticos (por exemplo, Windows, Linux, BSD) têm milhões de linhas de código de kernel. Não há como tanto código ser corrigido. Em contraste, o MINIX 3 tem cerca de 4000 linhas de código de kernel executável. Acreditamos que este código pode ser feito bastante próximo ao bug free. Em sistemas operacionais monilíticos, os drivers de dispositivo residem no kernel. Isso significa que quando um novo periférico é instalado, código desconhecido e não confiável é inserido no kernel. Uma única linha de código ruim em um driver pode derrubar o sistema. Este design é fundamentalmente falho. No MINIX 3, cada driver de dispositivo é um processo separado do modo usuário. Os Drivers não podem executar instruções privilegiadas, alterar as tabelas de página, executar I/O ou escrever para memória absoluta. Eles têm que fazer chamadas de kernel para esses serviços e o kernel verifica cada chamada de autoridade. 

Em sistemas operacionais monolíticos, um motorista pode escrever para qualquer palavra de memória e, portanto, acidentalmente destruir programas de usuários. No MINIX 3, quando um usuário espera dados do sistema de arquivos, ele constrói um descritor dizendo quem tem acesso e quais endereços. Em seguida, ele passa um índice para este descritor para o sistema de arquivos, que pode passá-lo para um driver. O sistema de arquivos ou driver pede então que o kernel escreva através do descritor, tornando impossível para eles escrever em endereços fora do buffer.

O diferenciamento de um ponteiro ruim dentro de um motorista vai travar o processo do motorista, mas não terá efeito no sistema como um todo. O servidor de reencarnação reiniciará o motorista acidentalmente automaticamente. Para alguns drivers (por exemplo, disco e rede) a recuperação é transparente aos processos do usuário. Para outros (por exemplo, áudio e impressora), o usuário pode notar. Em sistemas monolíticos, diferenciar um ponteiro ruim em um driver (kernel) normalmente leva a uma falha no sistema.

Se um driver entrar em um loop infinito, o agendador gradualmente diminuirá sua prioridade até que se torne o processo ocioso. Eventualmente, o servidor de reencarnação verá que ele não está respondendo a solicitação de status, então ele vai matar e reiniciar o driver de looping. Em um sistema monolítico, um motorista de looping trava o sistema.

O MINIX 3 usa mensagens de comprimento fixo para comunicação interna, o que elimina certos excessos de buffer e problemas de gerenciamento de buffer. Além disso, muitas explorações funcionam ultrapassando um buffer para enganar o programa para retornar de uma chamada de função usando um endereço de retorno empilhado substituído que aponta para o buffer invadido. Na MINIX 3, esse ataque não funciona porque as instruções e o espaço de dados são divididos e apenas o código no espaço de instrução (somente leitura) pode ser executado.

Os drivers do dispositivo obtêm serviços de kernel (como copiar dados para os espaços de endereço dos usuários) fazendo chamadas de kernel. O kernel MINIX 3 tem um mapa de bits para cada driver especificando quais chamadas ele está autorizado a fazer. Em sistemas monolíticos, cada motorista pode chamar todas as funções do kernel, autorizadas ou não.

O kernel também mantém uma tabela que diz quais portas de I/O cada driver pode acessar. Como resultado, um motorista só pode tocar em suas próprias portas de I/O. Em sistemas monolíticos, um driver de buggy pode acessar portas de I/O pertencentes a outro dispositivo. Nem todo motorista e servidor precisa se comunicar com todos os outros Drivers e servidores. Assim, um mapa de bits por processo determina para quais destinos cada processo pode enviar.

Um processo especial, chamado servidor de reencarnação, periodicamente pinga cada driver do dispositivo. Se o driver morrer ou não responder corretamente aos pings, o servidor de reencarnação substitui-o automaticamente por uma cópia nova. A detecção e substituição de drivers não funcionais é automática, sem qualquer ação do usuário necessária. Este recurso não funciona para drivers de disco no momento, mas na próxima versão o sistema será capaz de recuperar até mesmo drivers de disco, que serão sombreados em RAM. A recuperação do driver não afeta os processos de execução.

Quando ocorre uma interrupção, ela é convertida em um nível baixo para uma notificação enviada ao motorista apropriado. Se o motorista estiver esperando uma mensagem, ele recebe a interrupção imediatamente; caso contrário, ele recebe a notificação da próxima vez que fizer um RECEIVE para receber uma mensagem. Este esquema elimina interrupções aninhadas e facilita a programação do driver.

<h3>Vídeos recomendados</h3>

[The impact of MINIX](https://youtu.be/86_BkFsb4eI), por Tanenbaum.<p>
    <img src="https://i.ytimg.com/an_webp/86_BkFsb4eI/mqdefault_6s.webp?du=3000&sqp=CKCa-JgG&rs=AOn4CLBDazV83khZs4yfElJ87MQ-p4-B7g" width="350"></img><p>
	
[Andrew Tanenbaum - MINIX 3: A Reliable and Secure Operating System - Codemotion Rome 2015](https://youtu.be/jiGjp7JHiYs), por Tanenbaum.<p>
    <img src="https://i.ytimg.com/an_webp/jiGjp7JHiYs/mqdefault_6s.webp?du=3000&sqp=CPyW-JgG&rs=AOn4CLCfvF8sqy0bH0jyhwzpKvAQwlz7Ww" width="350"></img><p>
	
[https://youtu.be/MG29rUtvNXg](https://youtu.be/MG29rUtvNXg), por Tanenbaum.<p>
    <img src="https://i.ytimg.com/an_webp/MG29rUtvNXg/mqdefault_6s.webp?du=3000&sqp=CPih-JgG&rs=AOn4CLAIp2xbwSN1q85ixu74s1pN6B96rg" width="350"></img><p>

## Instalação

## Noções básicas de Minix

A idéia inicial do Minix era ser um clone do Unix. Desta forma, os comandos básicos do Minix são os mesmos disponíveis em sistemas Unix como, por exemplo no Linux. Os dois principais editores de arquivos do Minix são o <strong>vi</strong> e o <strong>elle</strong> (ELLE Looks Like Emacs). Um manual de referência para o vi pode ser obtido em [drumlin](http://drumlin.thehutt.org/vi).

A seguir são mostrados alguns comandos Minix/UNIX necessários para começar a utilizar o sistema.

Criar um diretório:
~~~
$ mkdir diretorio
~~~

Acessar um diretório:
~~~
$ cd diretorio
~~~

Listar o conteúdo do diretório corrente:
~~~
$ ls -l
~~~

Apagar um arquivo:
~~~
$ rm arquivo
~~~

Compilar um programa:
~~~
$ cc hello.c
$ ./a.out

ou 

$ cc -o hello hello.c
$ ./hello
~~~

Mudar as permissões de um arquivo:
~~~
# neste caso, adicionar a permissão de execução a um arquivo
$chmod +x arquivo
~~~

Executar um programa:
~~~
# em segundo plano (ou background)
$ ./programa &

# em primeiro plano (ou foreground)
$ ./programa
~~~

Executar um shell script:
~~~
$ sh script.sh
~~~

Exibe o estado e PID dos processos em execução:
~~~
$ ps -x
~~~

Mata um processo:
~~~
$ kill PID
~~~

Para mais informações:

* Lista completa dos manuais disponíveis do Minix 3: [manpages](http://www.minix3.org/manpages/index.html)
* Programming in the MINIX 3 Environment: [doc](http://www.minix3.org/doc/environ.html)

## Compilando um Kernel

O código-fonte do Minix está localizado no diretório /usr/src. Após alguma modificação no código, é necessário recompilar e criar um novo kernel. Para compilar:

~~~
# cd /usr/src/tools
# make
~~~

Nesse ponto o kernel apenas foi compilado, agora é necessário gerar e instalar a imagem do kernel.

~~~
# cd /usr/src/tools
# make install
~~~

A imagem do kernel é armazenada no diretório /boot/image. Cada vez que um kernel é compilado, uma cópia da imagem é armazenada neste diretório. Quando o computador for reiniciado, o Minix irá carregar a versão mais nova (última kernel compilado).

~~~
# ls -a /boot/image
. .. 3.1.2a 3.1.2ar0 3.1.2ar1
~~~

Se forem realizadas modificações no gerenciamento de processos, também é necessário recompilar o comando ps para exibir corretamente os processos no novo ambiente.

~~~
# cd /usr/src/commands/ps
# make 
# cp ps /usr/bin
# chown root /usr/bin/ps
# chmod 4755 /usr/bin/ps
~~~

Para que as alterações tenham efeito reinicie o sistema:

~~~
# shutdown -R
~~~

<strong>O básico do básico do editor VIM</strong>

O ambiente disponibilizado para a execução do Minix é uma console texto sem nenhum suporte a ambientes gráficos. Neste caso é necessário realizar as tarefas que habitualmente se está acostumado a fazer com ferramentas gráficas com seu equivalente em modo texto.

O VIM é um editor de texto. Ele é uma evolução de um outro editor bastante conhecido no mundo UNIX, o vi. Apesar de parecer rudimentar ele oferece uma série de funcionalidades bastante úteis. A grande vantagem de se conhecer um editor de textos como o vi é pode interagir com um sitema instalado quando a interface gráfica, por uma ou outra razão, não funciona. Normalmente, para repará-la, é necessário editar arquivos de configuração. Por tanto, o vi (e similares) resta como a única alternativa (Em caso de dúvida de sua utilidade, pergunte a um administrador de redes competente).

## Processos

MINIX usa uma abordagem de microkernel, que implementa somente mecanismos mais básicos, como o de gerenciamento de processos, comunicação entre processos, escalonamento de processos e algumas system calls disponíveis aos servidores e drivers. 
Assim, temos no MINIX uma quantidade bem pequena de processos que são executados em modo de kernel, sendo grande parte do seu sistema executado em modo de usuário. Diferente do Linux que tendo um kernel monolítico realiza uma grande quantidade de funções em modo kernel.
Para o MINIX então, o escalonador de processos de usuário tem uma importância extra já que terá que lidar com uma quantidade bem maior do que o escalonador de processos de kernel. 

![minix](https://user-images.githubusercontent.com/78819692/193430338-d7040c92-9629-4d39-8a3c-b86546ecb60d.png)
- Modelo em camadas do MINIX

Como visto na imagem acima, servidores e drivers são processos que rodam em nível de usuário, sendo assim possível que eles se comuniquem com o kernel e os demais processos de usuários se comunicarem apenas com os servidores.
Com todo esse sistema, elevamos a confiabilidade do sistema, pois o kernel terá uma interface mais segura, sendo que terá menor quantidade de linhas e consequentemente reduzindo a taxa de possíveis bugs e evitaremos o kernel de erros de implementação devido os processos de usuários não terem acesso direto e sim por meio de comunicaçẽs entre processos (interprocess communication ou IPC)

<h3> Gerenciamento de processos: </h3>

Os processos no MINIX tem um modelo padrão de processos, em que estes podem criar subprocessos que por sua vez podem criar outros subprocessos, tendo assim uma árvore de processos.

Para trabalhar com processos, o Minix 3 oferta duas chamadas de sistemas principais, o fork e o exec. Falando mais sobre essas duas chamadas:
* Fork: Único modo para criar um novo processo
* Exec: Possibilita carregar um programa num processo já criado

A comunicação entre processos ocorre em três primitivas, send, receive e sendrec, sendo que cada uma recebe dois parâmetros ( um contendo o processo destino/origem e o segundo o local de memória que contém a informação da mensagem ). 
Em relação a troca de mensagem, o MINIX 3 utiliza uma técnica chamada rendezvous. Essa técnica permite que quando um processo A envia uma mensagem para o processo B, o processo A ficará bloqueado até que o processo B recebe a mensagem, sendo desnecessário o uso de buffer para controle de mensagem, pois todas as mensagens possuem um tamanho fixo e erros de estouro de buffer são prevenidos.

<h3> Escalonador: </h3>
	
- No MINIX 3 o escalonador trabalha em um sistema de filas em multi-níveis, são dezesseis filas com prioridades definidas pelo sistema. A camada de prioridade mais baixa é utilizada em processos de segundo plano, sendo executados apenas quando não há  processos de níveis de prioridades maiores, nos quais são os de camadas mais inferiores. 
* Os processos de  Serves possuem prioridade acima dos processos de usuários e os processos de Drivers possuem prioridade acima dos de Servers e os processos do Kernel ocupam a fila de maior prioridade. Os processos podem se mover entre as filas (filas essas que não são estáticas) utilizando o mesmo comando nice que contém nos sistemas UNIX
* Cada processo possui um quantum. Os processos de usuários possuem um valor de quantum baixo. Já os processos de Drivers e Servers tinham que ser executados até serem bloqueados (tempo real), mas por questão de segurança são processos preempíveis entretanto possuem um alto valor de quantum. O processo de clock fica responsável pelo controle do tempo de execução do processo.
* Para garantir a justiça dos processos, no qual o processo de mais baixa prioridade possa também ser executado, o MINIX 3 possui um sistema específico. Caso um processo A que utilizou todo o quantum for reescalonado para a CPU, fazendo que isso impeça que outros processos sejam executados, então A terá sua prioridade reduzida, fazendo assim que em algum momento todos os processos sejam executados. Caso A venha a utilizar todo seu quantum sem que isso impeça outros processos a serem executados, A terá sua prioridade elevada ao máximo permitido para o processo.

Dentro das filas o algoritmo de alternância obrigatória é utilizado, com uma pequena alteração. Se   um   processo,   durante   sua   execução for bloqueado, quando este se tornar pronto novamente,será colocado no início da fila e irá utilizar o restante de seu quantum . Agora, se um processo utiliza todo o seu quantum, então será colocado no final da fila. O escalonador escolhe um processo da fila de mais alta prioridade para executar, se não tiver nenhum processo pronto na fila, então a fila de prioridade  subsequente   será   analisada.   Pela dependência dos processos Drivers com os Servers e deste último com os processos  usuários, os processos Drivers e Servers, normalmente, ficam em estado bloqueado, permitindo a execução dos processos usuários.

## Gerenciamento de Memória

## Sistema de Arquivos

Um processo executando armazena informações, porém esta informação é perdida quando o processo termina. Assim, a gravação destas informações na forma de arquivos garante que elas possam ser lidas e utilizadas posteriormente por outros processos. Ou seja, os arquivos são uma forma de armazenar as informações em disco, para utilização futura.

<h3>Conceitos iniciais</h3>

Arquivos comuns contém informações do usuário. Diretórios são arquivos de sistema para manter a estrutura do sistema de arquivos. Arquivos especiais de caracteres relacionam-se com a entrada/saída e são utilizados para modelar dispositivos de entrada/saída seriais como terminais, impressoras e redes.

Os arquivos mais comuns são ASCII e binários. Os arquivos ASCII possuem a vantagem de que podem ser exibidos e impressos como são, além de poderem ser editados com um editor de texto comum. Assim, quando vários programas utilizam arquivos ASCII, torna-se possível conectar a saída de um com a entrada de outro, como, por exemplo, em pipelines.

O sistema de arquivos no Minix funciona como um servidor de arquivos que é executado na própria máquina do usuário. Ele possui 39 tipos de mensagens para solicitar trabalho. Quando chega uma mensagem, é obtido o seu “tipo” para que seja chamado o procedimento adequado para este trabalho.

<h3>Arquivos</h3>

Este sistema de arquivos utiliza a estrutura de nó-i, que armazena os atributos e as localizações dos blocos de dados. Os primeiros endereços de disco ficam armazenados no próprio nó-i, assim, todas as informações de um arquivo pequeno estão diretamente no nó-i. Porém, quando há arquivos maiores, é necessário utilizar um dos endereços do nó-i para endereçar um “bloco indireto simples”. Caso o arquivo seja muito grande, também está disponível um “bloco indireto duplo” e um “triplo”, conforme mostra a Figura 1 a seguir.

![imagem-processo-arquivo](https://deinfo.uepg.br/~alunoso/2019/SO/MINIX/ARQUIVO/images/no-i.png)

Cada sistema de arquivos começa com um bloco de inicialização, que contém o código a ser executado quando o computador é ligado. Além disso, existe o superbloco, que armazena as informações que descrevem a organização do sistema de arquivos, conforme mostra a Figura 2 a seguir.

![](https://deinfo.uepg.br/~alunoso/2019/SO/MINIX/ARQUIVO/images/org-disco.png)

![](https://deinfo.uepg.br/~alunoso/2019/SO/MINIX/ARQUIVO/images/mapa-bits.png)

Utilizando um mapa de bits (Figura 3), o Minix controla quais nós-i e zonas estão livres. Se um arquivo é removido, o sistema procura o bloco do mapa de bits que corresponde ao nó-i e configura ele como 0. Quando um arquivo é criado, o sistema procura o primeiro nó-i livre nos blocos do mapa de bits. O superbloco possui um campo que aponta para o primeiro nó-i livre. Se nenhum nó-i estiver livre, é retornado valor 0.

Quando um arquivo é aberto, é necessário localizar seu nó-i para que ele seja carregado na tabela “inode” na memória, sendo removido quando o arquivo for fechado. As informações desta tabela permitem ao sistema de arquivos saber onde regravar o arquivo, caso seja modificado. A tabela também tem um contador, para guardar quantas vezes o arquivo for aberto, porém apenas uma cópia é mantida na memória. O nó-i também guarda informações sobre qual é o tipo do arquivo.

O sistema Minix usa um "cache" de blocos para melhorar o desempenho do seu sistema de arquivos. Este "cache" é feito com uma matriz de buffers, onde cada um possui um cabeçalho com ponteiros, contadores e sinalizadores. Os “buffers” que não estão sendo utilizados são colocados em uma lista duplamente encadeada, do mais para o menos recentemente utilizado. Também é utilizada uma tabela “hash” para saber se um bloco está ou não no "cache".

Quando o sistema de arquivos deseja acessar um bloco, ele chama o procedimento "get_block", juntamente com um número de dispositivo e um número de bloco. Se um buffer com o bloco é encontrado, incrementa-se o contador no cabeçalho, mostrando que ele está em uso, e retorna um ponteiro para ele. Se o bloco não é encontrado, pode-se utilizar o primeiro “buffer” livre. Se o bloco foi modificado, quando o sistema remover ele, deve gravá-lo no disco.

Quando o procedimento que estava usando o bloco termina, ele chama o procedimento "put_block", que libera o bloco (ou decrementa o contador, caso o bloco tenha sido aberto mais vezes). Se ele for removido, é gravado no disco. Se ocorrer a chamada de sistema SYNC, mesmo se o bloco não foi removido, mas foi modificado, ele é gravado no disco.

<h3>Diretórios</h3>

Um diretório no sistema Minix corresponde a um arquivo com entradas de 16 bytes, onde os dois primeiros formam o número do nó-i de 16 bits (1 byte = 8 bits, logo, 2 bytes = 16 bits) e os outros 14 bytes correspondem ao nome do arquivo.

![](https://deinfo.uepg.br/~alunoso/2019/SO/MINIX/ARQUIVO/images/diretorio.png)

Quando um arquivo é aberto, é retornado um descritor de arquivo (file descriptor) para ser utilizado pelas chamadas READ e WRITE. O número do descritor de arquivo (que é um dos campos da tabela de processos) indexa uma matriz, que é usada para localizar o arquivo correspondente ao descritor de arquivo informado.

Não podemos fazer uma entrada dessa matriz apontar diretamente para o nó-i do arquivo. É necessário utilizar uma nova tabela compartilhada, chamada "filp", onde estão todas as posições de arquivo (convém também colocar nesta tabela o ponteiro de nó-i). Assim, a matriz indexada pelo descritor de arquivo contém apenas um ponteiro para uma entrada na tabela "filp".

O sistema Minix também utiliza outra tabela, chamada "file_lock", para guardar o registro de todos os bloqueios. Cada uma das entradas da tabela indica se o arquivo está bloqueado para escrita ou leitura, o ponteiro para o nó-i do arquivo, o ID do processo que bloqueou, e os deslocamentos do primeiro e último bytes bloqueados.

Quando um processo tentar ler ou gravar em um "pipe" ("pipeline" ou canalização), o sistema de arquivos do Minix verifica o estado do "pipe" e, se a operação não pode ser feita, o sistema de arquivos registra os parâmetros da chamada de sistema na tabela de processos para reiniciar este processo quando for possível.

Em relação aos terminais e arquivos especiais de caracteres, o nó-i de cada arquivo especial possui dois números: o dispositivo principal, que indica a classe do dispositivo, por exemplo: disquete, terminal, disco rígido, disco de RAM; e o dispositivo secundário, que indica qual dispositivo deve ser usado.

## Entrada e Saída (E/S)

Uma das funções principais de um sistema operacional é controlar todos os dispositivos de entrada e saída de um computador, tratar erros, interceptar interrupções, fornecer uma interface entre o dispositivo e o sistema, emitir comandos para os dispositivos.

<h3>Módulos de entrada e saída</h3>

Um módulo de entrada e saída é a entidade dentro de um computador responsável pelo controle de um ou mais dispositivos externos e pela transferência de dados entre aqueles dispositivos e a memória principal e os registos da CPU. Assim, o módulo de E/S tem de ter uma interface interno ao computador (da CPU e a memória principal) e uma interface externa para o computador (ao dispositivo externo). As categorias principais de funções ou requisitos para um módulo de E/S caem dentro das seguintes:

* Temporização e controle
* Comunicação com o processador
* Comunicação com dispositivos
* Armazenamento temporário dos dados
* Detecção de erros

Durante qualquer período de tempo, a CPU pode comunicar com um ou mais dispositivos externos de forma imprevisível, dependendo das necessidades de E/S. Os recursos internos, tais como, a memória principal e o barramento de sistema, têm de ser partilhados entre um certo número de atividade incluindo o processamento de informação de E/S. Assim, a função de E/S inclui um requisito de temporização e controle, para controlar o fluxo de tráfego entre os recursos internos e os dispositivos externos.<p>

<strong>A comunicação com a CPU envolve:</strong>

Descodificação de Comandos: O módulo de E/S aceita comandos da CPU. Estes comandos são geralmente enviados como sinais no barramento de controle.

Dados: Os dados são trocados entre a CPU e o módulo de E/S através do barramento de dados.

Relato de status: Uma vez que os periféricos são lentos, é importante saber o estado do módulo de E/S.

Detecção de Erros: Cada dispositivo de E/S possui um endereço, tal como acontece com cada palavra na memória. Assim, um módulo de E/S tem de reconhecer um único endereço para cada periférico sobre o seu controlo. Numa outra perspectiva, o módulo de E/S tem de ser capaz de efetuar comunicação com o dispositivo. Esta comunicação envolve comandos.
	
<h3>Dispositivos de E/S</h3>

Os dispositivos de E/S podem ser divididos, genericamente, em duas categorias: dispositivos de bloco e dispositivos de caractere.

<h3>Dispositivos de Bloco</h3>

Dispositivos de blocos, são todos os dispositivos que podem enviar/transmitir dados em blocos de tamanho fixo. Um exemplo de dispositivo de bloco, é o HD.

<h3>Dispositivos de caractere</h3>

O dispositivo de caractere não utiliza estrutura de blocos nem posicionamento. No dispositivo de caractere ele recebe um fluxo de caracteres, além de não ser endereçável.

<h3>Controladoras de Dispositivo</h3>

As unidades de E/S geralmente consistem em um componente mecânico e em outro eletrônico. É possível separar as duas partes para oferecer um projeto mais modular e genérico. Em computadores pessoais, esse frequentemente toma a forma de uma placa de circuito impresso.

![](https://deinfo.uepg.br/~alunoso/2019/SO/MINIX/DISPOSITIVOS/site%20rea/barramento.png)

O trabalho da controladora é converter o fluxo serial de bits em um bloco de bytes e executar qualquer correção de erro necessária. O bloco de bytes tipicamente é primeiro montado, bit por bit, em um buffer dentro da controladora. Depois que sua soma de verificação foi verificada e o bloco foi declarado livre de erros, ele pode, então, ser copiado para a memoria principal. Cada controladora tem alguns registradores que são utilizados para comunicar-se com a cpu. Em alguns computadores, esses registradores são parte do espaço normal de endereçamento de memoria. Esse esquema é chamado E/S mapeada em memoria.

<h3>Acesso direto a memória (DMA)</h3>

Não importa se a CPU tem ou não E/S mapeada na memoria, ela precisa endereçar os controladores dos dispositivos para poder trocar dados com eles. A CPU pode requisitar dados de um controlador de E/S, um byte de cada vez, mas desperdiça muito tempo, de modo que um esquema diferente (DMA) seja usado.

O controlador de DMA tem acesso ao barramento do sistema. Eles contem vários registradores que podem ser lidos ou escritos na CPU, os quais possuem registrador de endereço de memoria, registrador de controle e registrador de contador de bytes.

O controlador lê um bloco do dispositivo, bit a bit, até que todo bloco esteja no buffer do controlador. Em seguida, ele calcula a soma de verificação, para constatar de que não houve algum erro de leitura. Então, o controlador causa uma interrupção. Quando o S.O inicia o atendimento, ele pode ler o bloco do disco a partir do buffer do controlador. Um bloco de byte ou uma palavra é lida no registrador do controlador e armazenada na memoria principal.

<h3>Software de entrada e saída</h3>

Um conceito-chave no projeto de software de E/S é conhecido como independência de dispositivo. Isso significa que deve ser possível escrever programas que podem ler arquivos em um disquete, em um disco rígido ou em um CDROM, sem que seja necessário modificar os programas para cada tipo de dispositivo diferente. Outra questão importante para o software de E/S é o tratamento de erros. Em geral devem ser tratados o mais perto possível do hardware. Se a controladora descobrir um erro de leitura, ela devera tentar corrigir o erro se puder. Se não puder, então o driver de dispositivo devera tratá-lo, talvez tentando simplesmente ler o bloco novamente.

<h3>Manipulação de interrupções</h3>

Interrupções é uma realidade desagradável. Elas devem ser escondidas longe, no fundo das entranhas do sistema operacional, de modo que o mínimo possível do sistema saiba sobre elas. A melhor maneira de oculta-las é ter cada processo que inicia uma operação de E/S bloqueado ate que a E/S tenha-se completado e a interrupção tenha ocorrido. O processo pode bloquear-se fazendo um dowx em um semáforo. Um wait em uma variável de condição ou um receive em uma mensagem, por exemplo. Quando as interrupções acontecem, o procedimento de interrupção faz o que tem de fazer para desbloquear o processo que iniciou a E/S em alguns sistemas, ele fara um UP em um semáforo. Em outros, ele fara um sinal em uma variável de condição em um monitor. Em outros, ainda, ele enviara uma mensagem para o processo bloqueando. Em todos casos, o efeito geral da interrupção será que um processo que anteriormente estava bloqueado agora será capaz de executar.

<h3>Impasses</h3>

Os sistemas de computador estão repletos de recursos que podem ser utilizados apenas por um processo por vez. Ter dois processos simultaneamente gravando na impressora resulta em uma confusão. Portanto, todos os sistemas operacionais têm a capacidade de temporariamente conceder acesso exclusivo a certo recursos para um processo.

<h3>Recursos</h3>

dispositivos, a arquivos, etc. Os recursos dividem-se em dois tipos: preemptivel e não preempetivel. Um recurso preempetivel é aquele que pode ser tirado do processo que é proprietário dele sem nenhum problema. A memoria é um exemplo de um recurso preemptivel.

<h3>Entrada e saída MINIX</h3>

No Minix, drivers de entrada e saída são feitos com passagem de mensagens, de forma que rodem em modo usuário e se comuniquem com o kernel. Isso garante que um driver tenha limites quanto ao que pode fazer e aumente a estabilidade do sistema.


## Referências

* http://minix3.org/doc/ (documentação oficial) [1]
* https://deinfo.uepg.br/~alunoso/2019/SO/MINIX/DISPOSITIVOS/site%20rea/#:~:text=Entrada%20e%20saida%20minix%20No%20Minix%2C%20drivers%20de,pode%20fazer%20e%20aumente%20a%20estabilidade%20do%20sistema. [2]
	
#### Autores
	
- Gabriel Henrique Silva Gontijo
- Leonardo de Oliveira Campos
- Lucas Ribeiro Silva
- Thomás Teixeira Pereira

