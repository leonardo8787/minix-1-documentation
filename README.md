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

## Gerenciamento de Memória

## Sistema de Arquivos

## Entrada e Saída (E/S)

## Referências

<h1></h1>
	
#### Autores
	
- Gabriel Henrique Silva Gontijo
- Leonardo de Oliveira Campos
- Lucas Ribeiro Silva
- Thomás Teixeira Pereira
