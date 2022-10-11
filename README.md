<h3 align="center">	
Escalonamento | MINIX 3 <p>

</h3>

<h1></h1> 

Documentação sobre o sistema operacional Minix 3 [Wiki :scroll:](https://github.com/leonardo8787/minix-1-documentation/wiki/MINIX-3-%7C-Guia) 

Neste repositório iremos dicertar sobre organização estrutural, escalonamento e política nos processos do sistema operacional MINIX 3, desenvolvido pelo professor Tanembaum.
Também iremos falar sobre o mudança na política do sistema além de abrir "issues" falando mais detalhadamente sobre cada processo ao qual fomos submetidos a erros
incluindo a compilação do kernel e a virtualização da ISO gerada no processo. Este repositório tem fins acadêmicos e foi produzido para comunidade do CEFET/MG.

<h2>Processos</h2>

É inivitável falar sobre processos quando estamos tratando sobre escalonamento, há uma convergência nos assuntos, visto que os processos são uma sombra dos escalonadores. Tendo isso em vista é hora de discertar sobre o que é um processo e como o sistema operacional MINIX 3 gerencia os seus processos, para que possamos 
adentrar nos escalonadores. 

<h2>Organização Estrutural</h2>

<h3> Design Inteligente </h3> 
 
O MINIX utiliza um design inteligente, dentro do nicho de Sistemas Operacionais, que utiliza como base um Microkernel com 15.000 linhas de código (para comparação, o Kernel do LINUX possui mais de 15 milhões de linhas). Existe uma estimativa de que a cada 1000 linhas de código um a dez bugs podem aparecer, alguns sendo menos sérios como erros de gramática em mensagens, mas drivers possuem a estimativa de terem entre 3 a 7 vezes mais bugs do que o resto kernel, e 70% do código são drivers. 
 
O MINIX também é altamente modular, o que significa que algumas partes do núcleo do sistema estão em pastas diferentes e independentes, que podem ser chamadas e adicionadas ao sistema durante a execução do mesmo. 
 
Para atingir esse tipo de design, algumas partes foram removidas do kernel e transformadas em módulos. Primeiramente, todos os módulos que precisam ser carregados foram isolados do kernel e passaram para o espaço de usuário, incluindo todos os drivers e sistemas de arquivo. Assim, durante a execução, cada módulo é tratado como um processo diferente e executado utilizando o princípio de menor autoridade (POLA), garantindo que nenhum processo tenha mais poder do que necessário para cumprir sua tarefa, assim tendo também menos poder para causar algum dano ao processo como um todo. Por exemplo, algum port de entrada e saída poderia ter acesso ao disco, talvez causando um bug, mas nesse modelo este estaria em módulo no espaço de usuário, logo não poderia causar tão problema por falta de acesso. 
 
<h3> Arquitetura em Camadas do MINIX </h3> 
 
O MINIX é composto de quatro camadas principais que formam uma hierarquia de acesso. A primeira camada é o Kernel, que trata serviços de baixo nível necessários para a execução do sistema, como escalonamento de processos, gerenciamento de interrupções, traps, e comunicação entre processos. As camadas acima desta estão no espaço de usuario. 
 
A segunda camada contém os drivers, que são responsáveis pelo funcionamento de dispositivos como disco, terminais, placas de rede, relógio, além de dispositivos E/S. 
 
A terceira é a dos servidores, e contém processos que fornecem serviços úteis para os processos de usuário como, por exemplo, gerenciamento de memória (MM), sistema de arquivos (FS), rede, etc. 
 
Por fim, a quarta camada diz respeito aos processos de usuário como shells, editores e compiladores. 


<h2>Escalonamento</h2>

<h2>Política</h2>

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

