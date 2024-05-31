# NodeRacer: Event Racer Detection

## Navegação

1. [Introdução](#introducao)
2. [Motivação](#motivacao)
3. [Background](#background)
4. [Abordagem](#abordagem)

## <a name="introducao"></a>1. Introdução

JavaScript é uma linguagem de programação dinâmica multiparadigma (permitir que o programador implemente o código através do uso dos mais adequados paradigmas ao problema em questão) que foi inicialmente projetada para programar aplicações web do lado do cliente.
Entretanto viu-se uma necessidade de executar aplicações do lado so servidor e como consequencia, em 2009, foi lançado a plataforma Node.js que ajudou a espalhar o Javascript para outros domínios, incluindo aplicativos de servidor, ferramentas de linha de comando e até mesmo aplicativos de desktop.

## <a name="motivacao"></a>2. Motivação

---

## <a name="background"></a>3. Background

A arquitetura do Node.js é baseada em eventos onde `callbacks` (também conhecidos como manipuladores de eventos) são registrados para serem executados quando eventos relevantes ocorrem. Os `callbacks` são agendados para serem executados por um loop de eventos single-threaded. Dessa forma, podemos descrever o nodeJs através de evento e single-thread, o que significa que todas as operações que demandam tempo de execução (como operações de entrada e saída, I/O) são executadas de maneira assíncrona. Isso funciona da seguinte forma: uma função é chamada e associada a um callback. Por exemplo, ao ler dados de um arquivo, a execução principal não é interrompida; em vez disso, o Node.js continua executando outras tarefas e chama o callback quando a leitura do arquivo é concluída.

`Single-threaded` refere-se a um ambiente de execução onde apenas uma thread está ativa e executando tarefas de cada vez. Em um sistema single-threaded, todas as tarefas são processadas sequencialmente pela única thread disponível. Isso significa que, enquanto uma tarefa está sendo executada, nenhuma outra tarefa pode ser iniciada até que a primeira seja concluída.

No Node.js, quando se executa operações assíncronas como leitura de arquivos, solicitações HTTP ou outras operações de entrada/saída (I/O), essas operações são tratadas em segundo plano para que o programa não fique parado esperando a operação ser concluída. Isso é crucial para que o Node.js seja eficiente em lidar com muitas conexões simultâneas sem bloquear a thread principal.

Para lidar com essas operações assíncronas, o Node.js possui um mecanismo interno de `thread pool`. Este pool de threads é um conjunto de threads separadas, independentes da thread principal do Node.js, que são usadas para executar essas operações de I/O assíncronas em paralelo.

Por exemplo, se ocorrecem várias solicitações de leitura de arquivos simultaneamente em um código Node.js, o Node.js pode atribuir cada uma dessas operações de leitura de arquivo a uma thread diferente no pool de threads. Isso permite que várias operações de leitura de arquivo ocorram em paralelo, enquanto a thread principal do Node.js continua a executar outras tarefas.

Portanto, o pool de threads do Node.js é usado para executar operações de I/O assíncronas em segundo plano, enquanto a thread principal do Node.js continua a executar outras tarefas e a lidar com o loop de eventos. Isso ajuda o Node.js a ser eficiente e escalável em ambientes com muitas solicitações concorrentes.

Alem disso cada thread em um pool de threads tem seu próprio espaço de memória separado. Isso significa que as variáveis e os recursos alocados em uma thread não são compartilhados com outras threads no pool.

Por exemplo, se está sendo realizado várias operações de leitura de arquivos em paralelo em diferentes threads do pool, cada thread tem sua própria área de memória para armazenar os dados lidos do arquivo e para realizar qualquer processamento adicional necessário. Isso garante que os dados e recursos em uma thread não interfiram ou sejam afetados pelas operações em outras threads.

Essa separação de memória entre as threads é fundamental para garantir a integridade e a segurança dos dados, bem como para evitar condições de corrida e outros problemas de concorrência que podem ocorrer em sistemas multithreaded se não forem tratados corretamente.

Quando uma operação de I/O é concluída, o resultado é enviado de volta para a thread principal do Node.js. O callback associado à operação é então colocado na fila do loop de eventos para ser executado quando a thread principal estiver livre para processá-lo. Entretanto, a ordem em que os `callbacks` são chamados é não determinística, ou seja, não podemos prever com certeza em qual ordem os `callbacks` serão executados. Isso ocorre especialmente em um ambiente assíncrono e concorrente, onde várias operações podem ocorrer simultaneamente e seus `callbacks` podem ser chamados em momentos diferentes.

Além disso, quando múltiplas operações assíncronas estão ocorrendo simultaneamente e seus `callbacks` são chamados em momentos diferentes, seus períodos de execução podem se sobrepor no tempo. Esse fenômeno é chamado de interleaving.

Devido a essas características, aplicações Node.js são suscetíveis a `condições de corrida (event races)`, que podem causar bugs de corrida (race bugs). Esses bugs são difíceis de detectar e podem levar a crashes, travamentos do sistema e persistência de dados inconsistentes.

## <a name="abordagem"></a>4. abordagem

A abordagem `NODERACER` é composta por três fases principais.

Fase 1, ela instrumenta a aplicação e coleta informações relevantes para a fase seguinte a partir de uma execução, que pode ser conduzida por um teste ou por uma interação manual com a aplicação.

Fase 2 utiliza as informações coletadas para inferir uma relação de precedência ("happens-before") entre os callbacks.

Fase 3, ela executa a aplicação várias vezes em um modo especial que adia seletivamente os callbacks enquanto respeita a relação de precedência, para expor erros de corrida de eventos.

### Fase de observação

Na fase de observação, o `NODERACER` instrumenta o código da aplicação para rastrear _comportamentos assíncronos_, _chamadas de funções_ e _certas funções nativas_. Em seguida, ele executa a aplicação e coleta informações sobre a execução em um arquivo de `log`.

Primeiro, o `NODERACER` rastreia a vida útil do comportamento assíncrono usando as funções `Async Hooks`: `init`, `before` e `promiseResolve`.

A função `init` é chamada toda vez que um `callback assíncrono` é registrado, e o `NODERACER` adiciona uma entrada no log com um ID de callback único, o ID do callback em execução e o tipo de callback (Immediate, Timeout, TickObject, Promise ou Outro). Para timeouts, ele também registra o atraso especificado no registro.

A função `before` é chamada imediatamente antes da execução de um callback, e, nesse caso, uma entrada é adicionada ao log com o ID do callback. Essas entradas são usadas principalmente para distinguir diferentes ocorrências de um callback; por exemplo, setInterval define um callback que pode ser chamado várias vezes.

A função `promiseResolve` é chamada quando uma promise é resolvida, e, nesse caso, uma entrada é adicionada ao log com o ID do callback e o ID do callback em execução, permitindo a identificação da causalidade entre promises.

A `API Async Hooks` não fornece diretamente informações sobre quais funções são usadas como callbacks. Por essa razão, também usamos a biblioteca `njsTrace`, que fornece duas funções de hook, `onEntry` e `onExit`, permitindo ao `NODERACER` adicionar uma entrada cada vez que uma função é chamada ou retornada, respectivamente, com informações sobre a função (seu nome, se disponível, caminho do arquivo e número da linha) e o ID do callback em execução.

Finalmente, fazemos uma modificação ou extensão (monkey patch) nas funções `Promise.all` e` Promise.race` para rastrear de maneira semelhante quando são chamadas ou retornadas (com entradas init do Async Hooks no meio).

Todas as informações registradas são usadas na próxima fase para identificar os callbacks e as restrições de ordenação entre eles.
