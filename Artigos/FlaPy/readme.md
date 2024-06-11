# FlaPy: Mining Flaky Python Tests at Scale

## o que é flaky?

`Flaky` é um termo muito utilizado na área de desenvolvimento de software para descrever testes automatizados que têm comportamento não determinístico. Ou seja, esses testes podem apresentar resultados diferentes em diferentes momentos, mesmo quando executados no mesmo código fonte e ambiente.

Os testes flaky podem tanto passar quanto falhar em uma execução, sem que tenha havido mudança no código do software testado. Isso pode ocorrer devido a diversos fatores, como dependência de condições de tempo de execução, falhas de rede, concorrência de processos, entre outros.

A presença de testes flaky pode ser prejudicial para o processo de desenvolvimento de software, pois pode levar a falsos positivos ou falsos negativos na detecção de bugs, além de causar desperdício de tempo na depuração de falhas falsas. Por isso, é importante identificar e corrigir esses testes para garantir a confiabilidade dos resultados dos testes automatizados.

## Utilizando o FlaPy para Identificar Testes Flaky

O FlaPy é uma ferramenta de fácil utilização que permite aos desenvolvedores e pesquisadores identificar testes flaky em conjuntos de projetos Python e suas suítes de teste. Escrito em Python, o FlaPy oferece uma interface de linha de comando para executar testes e analisar os resultados. Aqui está um resumo das etapas envolvidas no uso do FlaPy:

1. **Instalação (Seção A):** O FlaPy utiliza o Docker para isolar as execuções dos testes e proporcionar uma instalação rápida. Os scripts de suporte estão disponíveis no repositório do [Flapy](https://github.com/se2p/FlaPy) no GitHub.

2. **Entrada (Seção B):** Após a instalação, é possível definir os projetos de estudo, fornecendo seus nomes, URLs Git e outras informações relevantes em um arquivo CSV de entrada.

3. **Execução de Testes Localmente (Seção C):** Utilizando o comando `run` no `flapy.sh`, os testes dos projetos definidos são executados um número especificado de vezes. Opções adicionais permitem executar testes em ordem aleatória e salvar os resultados em um diretório específico.

4. **Execução de Testes no SLURM (Seção D):** Para execução em um cluster SLURM, o FlaPy oferece a opção de executar testes em diferentes nós, utilizando o Podman em vez do Docker para melhor desempenho e escalabilidade.

5. **Análise de Resultados (Seção E):** O comando `parse` do FlaPy permite analisar os resultados dos testes executados. Ele fornece uma visão geral dos testes executados, identificando aqueles que são considerados flaky e fornecendo informações detalhadas sobre sua execução.

### Demonstração da Eficácia do FlaPy

Para demonstrar as capacidades do FlaPy, replicamos nosso estudo sobre testes flaky em Python. Inicialmente, realizamos uma amostragem aleatória de 30.000 projetos do PyPI usando o comando de amostragem do FlaPy. Em seguida, executamos os testes desses projetos 100 vezes em ordem padrão e 100 vezes em ordem aleatória. Utilizamos um cluster SLURM composto por 58 máquinas para executar até 700 iterações em paralelo, o que levou cerca de 3 dias e gerou 5 GB de dados comprimidos. Isso contrasta com um tempo de execução total sequencial de aproximadamente 20.500 horas (854 dias). Descobrimos que entre os 10.924 projetos que continham pelo menos um teste, 396 continham pelo menos um teste flaky. No total, encontramos 2.460 testes flaky nos aproximadamente 1,5 milhão de testes executados. Esses números confirmam resultados anteriores, exceto por menos flakiness de infraestrutura.

### Extensibilidade do FlaPy

O FlaPy pode ser facilmente estendido para implementar novos recursos, como medição de métricas de tempo de execução adicionais ou manipulação da execução de teste para provocar flakiness. Isso pode ser feito utilizando o argumento opcional --core-args do comando run para encaminhar argumentos diretamente para o procedimento central de execução de teste.

### Trabalhos Relacionados

Existem outras ferramentas de detecção de flakiness baseadas em rerun, como o framework iDFlakies, que se assemelha ao FlaPy, mas se destina a projetos Java construídos com Maven ou Gradle. No entanto, o FlaPy é destinado a pesquisadores que desejam conduzir estudos e avaliações em grandes conjuntos de projetos de amostra diversificados.

### Conclusões

O FlaPy é uma ferramenta fácil de usar para identificar testes flaky, repetindo a execução das suítes de teste de um conjunto dado de projetos Python. Ele aborda os desafios dessa tarefa garantindo que as execuções individuais dos testes sejam isoladas, simulando condições de integração contínua do mundo real e garantindo resultados precisos. Além disso, promove a diversidade entre os projetos de amostra e oferece meios para paralelizar as execuções de teste em um cluster SLURM. Por fim, oferece uma interface fácil de usar para consolidar os resultados dos testes em uma tabela concisa. O código-fonte do FlaPy está disponível no GitHub sob a licença GNU LGPL. Contribuições, feedback e perguntas são bem-vindos.
