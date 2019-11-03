# Gerenciamento de memória da JVM
A JVM é uma máquina virtual usada para executar aplicações Java e também outras linguagens que são compiladas para o bytecode do Java (Groovy, Jython, JRuby, Scala). Como uma máquina físíca, a JVM necessita de recursos de processamento e memória. No caso da CPU, o próprio sistema operacional pode conceder tempo de processamento para a JVM diretamente como um processo comum. Já quando falamos em memória, a JVM faz um gerenciamento interno para garantir o isolamento em relação a memória do sistema operacional.  

Na JVM, as variáveis locais e referências de vida curta são colocar na stack memory e os objetos e classes na heap memory.

## Stack Memory
A stack memory é uma estrutura de First in First Out(FIFO) e de curta duração. Variáveis locais são colocadas e retiradas durante a execução de um método.

## Heap Memory
A memória heap do Java pode ser dividida em três espaços: Old Generation(OG), New Genaration(NG) e Permanent Generation (PermGen) que foi substituida pelo Metaspace no Java 8.  

### New Generation
A New Generation é separada em 3 partes: Eden, Survivor 0 (S0), Survivor 1 (S1).   
Todos objetos que são criados dentro da aplicação são colocados no Eden. Depois de um determinado tempo, é feita a coleta dos objetos que ainda estão sendo usados e limpado o Eden junto com os objetos que não estavam sendo utilizados. Esse processo de coleta é chamado de Garbage Collection Minor (GC Minor). Os objetos que sobreviveram vão para o S0. Se o objeto já estava no S0 quando foi feita a coleta ele é passado para o S1. Depois de vários ciclos de GC Minor, os objetos que estão no S1 são passados para o espaço de Old Generation.  

### Old Generation
A Old Generation é a área de heap onde estão os objetos mais antigos, esse espaço é por padrão duas vezes maior que o NG, assim, a heap é separada em 1/3 para a NG e 2/3 para a OG. Objetos que estão na New Generation são enviados a Old Generation quando são executados vários ciclos de coleta GC Minor. Quando o espaço da OG é esgotado ou um determinado tempo é passado (tempo sempre muito maior que o GC Minor), ocorre o Garbage Collection Major (GC Major). No GC Major, similar ao GC Minor os objetos que estão sendo usados e estão na OG são mantidos e o resto removido. Como o espaço do OG é maior e faz a checagem de todos objetos que estão na memoria, esse evento exige uma parada completa da aplicação ("Stop the world") causando indisponibilidade da aplicação.  

### Permanent Generation/Metaspace
A PermGen é o espaço de memória onde estão os objetos imutáveis. Objetos imutáveis não podem ser modificados e permitem ser reusados várias vezes. Objetos na PermGen são limpos no GC Major. A partir do Java 8 o PermGen passou a se chamar Metaspace e tem por padrão uso direto da memória do sistema.  

## Configurações de Memória:
**-Xmx**: indica a quantidade de mémória maxima usada para o OG e o NG.  
**-Xms**: indica a quantidade de memória inicial usada para o OG e o NG.  
**-XX:MaxPermGen**:indica a quantidade de memória maxima usada para a PermGen. A partir do Java 8, a PermGen foi substituida pelo metaspace e o máximo de memória padrão do Metaspace é ilimitado, sendo assim, limitado pela memória do SO.  

## Configurações da Garbage Collection
Como foi visto, o GC Major causa uma indisponibilidade na aplicação e é necessária uma maior atenção nessa parte. 
Existem 5 estratégias para executar o GC  
**Serial GC (-XX:+UseSerialGC)**: GC é executado sequencialmente (uma Thread).  
**Parallel GC (-XX:+UseParallelGC)**: GC executado com paralelismo na GC Minor e sequenciamente na GC Major reduzindo o tempo da coleta.  
**Parallel Old GC (-XX:+UseParallelOldGC)**: GC executado com paralelismo na GC Minor e Major.  
**Concurrenct Mark Sweep GC(CMS) (-XX:+UseConcMarkSweepGC)**: Na GC Minor, ele se comporta como o Parallel GC. Na GC Major, parte do processo de marcar os objetos que ainda estão sendo utilizados é feito enquanto a aplicação está sendo executada, reduzindo o tempo da coleta.  
**G1 GC (-XX:+useG1GC)**: disponível a partir da versão 7 do Java. Não existe o conceito de OG ou de NG. É dividido o heap em espaços iguais e é limpo o espaço com menos objetos vivos. Espaços com menos objetos vivos são mais fáceis de serem limpos, objetos vivos necessitam de serem copiados para outros espaços e esse processo toma um tempo considerável. A tarefa de selecionar os objetos vivos é feita como no CMS, em paralelismo enquanto a aplicaço está sendo executada. No Java 9 e 10 é usada essa estratégia por padrão.  

## Possíveis problemas com memória
Problemas com memória são evidenciados quando a aplicação lança OutOfMemory Exception. Também ocorrem quando o sistema passa a usar o swap. O swap é um espaço de disco reservado para auxiliar a memória do sistema no caso dela se esgotar, como o disco é lento a aplicação que usa swap pode apresentar lentidão.

### java.lang.OutOfMemoryError: PermGen space
Esse erro ocorre quando a PermGen esgota. Aplicações que importam muitas APIs acabam consumindo bastante memória da Permgen e esgotando o recurso da máquina. Uma solução para esse problema é aumentar a PermGen ou migrar para o Java 8 e usar o metaspace.

### java.lang.OutOfMemoryError: Java heap space
Esse erro ocorre quando o OG e o NG esgotam. É possível que exista muitos objetos sendo usados na aplicação, ou um objeto qe usa muita memória sendo usado. Uma forma de entender melhor o problema é fazer um dump da memória e visualizar em alguma aplicação como o jvisualvm que já vem por padrão no JDK.

### swap
Quando a máquina começa a usar o swap ocorre uma possível lentidão na máquina. O swap pode ser evidenciado quando usamos o comando ```free -m``` e vemos que a quantidade de memória livre está esgotada e o swap está sendo usada.
