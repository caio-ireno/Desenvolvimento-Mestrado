# FlaPy: Mining Flaky Python Tests at Scale

## Navegação

1. [Introdução](#introducao)
2. [Motivação](#motivacao)
3. [Background](#background)
4. [Abordagem](#abordagem)

## <a name="introducao"></a>1. Introdução

### Flaky Tests

**Flaky tests** são testes que se comportam de maneira não determinística, ou seja, eles podem tanto passar quanto falhar na mesma versão do software. Isso significa que o mesmo código pode, às vezes, passar no teste e, em outras, falhar, sem que nenhuma mudança tenha sido feita no código entre essas execuções.

### Problemas com Flaky Tests

Os flaky tests são problemáticos por várias razões:

1. **Atrasam o desenvolvimento:** Como esses testes podem falhar sem motivo claro, os desenvolvedores precisam gastar tempo analisando manualmente as falhas, o que retarda o processo de desenvolvimento.
2. **Atrasam o feedback dos testes:** Quando um teste falha de forma inconsistente, ele precisa ser reexecutado várias vezes para confirmar se a falha é real ou apenas um comportamento esporádico. Isso atrasa o feedback que os testes devem fornecer.
3. **Reduzem a confiança nos testes:** Com o tempo, se os desenvolvedores percebem que os testes frequentemente falham sem razão aparente, eles começam a confiar menos na eficácia e precisão do processo de teste.

### Pesquisas Anteriores

Para entender melhor os flaky tests, pesquisadores têm conduzido estudos empíricos (baseados em observação e experiência) investigando flaky tests em projetos de código aberto escritos em Java. Esses estudos ajudam a compreender onde e por que os flaky tests ocorrem, mas a maioria das ferramentas existentes se concentra apenas em Java.

### Necessidade de Ferramentas para Outras Linguagens

A comunidade carece de ferramentas de mineração para outras linguagens de programação que incluam recursos importantes para a pesquisa, como a paralelização das execuções dos testes. Sem essas ferramentas, é difícil conduzir estudos semelhantes em outras linguagens.

### Introdução ao FlaPy

Para promover e possibilitar a pesquisa sobre flakiness em outras linguagens de programação, os autores desenvolveram o **FlaPy**, uma ferramenta para minerar testes flaky em Python em grande escala, executando repetidamente os conjuntos de testes de um conjunto de projetos Python. As principais características do FlaPy são:

1. **Amostragem automática de projetos do PyPI:** PyPI (Python Package Index) é um repositório de software para a linguagem Python. O FlaPy pode automaticamente selecionar projetos deste repositório.
2. **Isolamento das execuções dos testes:** Isso garante que os resultados sejam realistas e precisos, sem interferência de outros fatores.
3. **Suporte a diferentes estratégias de instalação de dependências:** Para promover a diversidade entre os projetos amostrados, a ferramenta pode instalar dependências de maneiras diferentes.
4. **Paralelização das execuções dos testes via SLURM:** SLURM é um sistema de gerenciamento de cluster que permite executar vários testes em paralelo, economizando tempo e recursos.
5. **Análise e apresentação dos resultados de forma concisa:** Os resultados são apresentados de uma maneira que é fácil de entender e analisar.

Vamos explicar a seção "USANDO FLAPY" em formato markdown.

---

## II. USANDO FLAPY

FlaPy é uma ferramenta fácil de usar que permite aos pesquisadores identificar testes flaky dentro de um conjunto específico de projetos Python e seus conjuntos de testes. O próprio FlaPy é escrito em Python e fornece uma interface de linha de comando para executar testes e analisar os resultados. Quando fornecido com um conjunto de projetos Python como entrada (Seção II-B), ele configura um ambiente de execução isolado e executa os conjuntos de testes dos projetos um número especificado de vezes (Seções II-C e II-D). FlaPy analisa os resultados dos testes e os apresenta em uma tabela concisa (Seção II-E).

### A. Instalação

FlaPy utiliza Docker para isolar as execuções dos testes e proporcionar uma experiência de instalação rápida. Para automatizar a criação de contêineres, ele vem com scripts de suporte que estão localizados no repositório FlaPy e podem ser clonados via:

```sh
git clone https://github.com/se2p/flapy
```

O ponto de entrada principal do FlaPy é o script `flapy.sh`. A imagem Docker do FlaPy será baixada automaticamente na primeira utilização. Note que FlaPy requer que o Docker esteja instalado e executável sem privilégios de root. Além disso, não possui outros requisitos.

### B. Entrada

Após a instalação do FlaPy, podemos definir nossos objetos de estudo, ou seja, o conjunto de projetos Python cujos conjuntos de testes queremos executar. Como exemplo ao longo deste artigo, utilizamos os seguintes projetos, nos quais anteriormente encontramos flakiness:

- **avwx-engine**: um motor global de busca e análise de dados meteorológicos de aviação.
- **jgutils**: um módulo de utilidades Python.
- **flapy example**: um projeto de exemplo criado pelos autores deste artigo, localizado no repositório FlaPy em `minimal flaky python tests/`.

Para cada projeto, devemos fornecer um nome e uma URL do Git de onde seus fontes podem ser clonados. Alternativamente, também podemos fornecer um caminho de onde os fontes serão copiados. Opcionalmente, podemos especificar uma revisão ou tag do Git a ser verificada antes da execução dos testes, uma versão do PyPI usada para instalar o próprio projeto no ambiente de teste para recuperar suas dependências em vez de procurar dependências em seu código-fonte, as funções que queremos rastrear (veja a documentação para mais informações), bem como um subconjunto de testes que queremos executar em vez de todo o conjunto de testes. Essas informações devem ser fornecidas via um arquivo CSV, que chamamos de `input-csv`. Um possível `input-csv` para nosso exemplo contínuo é mostrado na Tabela I e também pode ser encontrado no repositório FlaPy com o nome `flapy input example.csv`. O FlaPy também pode gerar arquivos `input-csv` automaticamente amostrando projetos do PyPI. Para fazer isso, use:

```sh
./flapy.sh sample
```

Referimo-nos a uma linha no `input-csv` como uma iteração. Como mostrado na Tabela I, em nosso exemplo não usamos uma, mas quatro iterações por projeto. Dividir as execuções dos testes dessa forma tem várias vantagens, especialmente ao executar os testes em um cluster SLURM (Seção II-D): Como cada linha no `input-csv` resulta em um trabalho SLURM, a divisão em pedaços menores ajuda a evitar timeouts (muitos clusters têm tempos máximos de execução por trabalho, por exemplo, 24h) e permite mais paralelização e melhor agendamento, acelerando o processo. Além disso, como cada trabalho SLURM pode ser agendado em uma máquina diferente, isso ajuda a detectar flakiness de infraestrutura, ou seja, problemas globais que levam a falhas em massa nos testes.

#### Tabela I: Input-CSV para nosso exemplo contínuo

| PROJECT NAME  | PROJECT URL                                                                          | PROJECT HASH | PYPI TAG | FUNCS TO TRACE | TESTS TO RUN  |
| ------------- | ------------------------------------------------------------------------------------ | ------------ | -------- | -------------- | ------------- |
| avwx-engine   | [https://github.com/avwx-rest/avwx-engine](https://github.com/avwx-rest/avwx-engine) | ed1d3e       |          |                |               |
| avwx-engine   | [https://github.com/avwx-rest/avwx-engine](https://github.com/avwx-rest/avwx-engine) | ed1d3e       |          |                |               |
| avwx-engine   | [https://github.com/avwx-rest/avwx-engine](https://github.com/avwx-rest/avwx-engine) | ed1d3e       |          |                |               |
| avwx-engine   | [https://github.com/avwx-rest/avwx-engine](https://github.com/avwx-rest/avwx-engine) | ed1d3e       |          |                |               |
| jgutils       | [https://github.com/jerodg/jgutils](https://github.com/jerodg/jgutils)               | 6c8ed6       |          |                |               |
| jgutils       | [https://github.com/jerodg/jgutils](https://github.com/jerodg/jgutils)               | 6c8ed6       |          |                |               |
| jgutils       | [https://github.com/jerodg/jgutils](https://github.com/jerodg/jgutils)               | 6c8ed6       |          |                |               |
| jgutils       | [https://github.com/jerodg/jgutils](https://github.com/jerodg/jgutils)               | 6c8ed6       |          |                |               |
| flapy example | ./minimal flaky python tests                                                         |              |          |                | test flaky.py |
| flapy example | ./minimal flaky python tests                                                         |              |          |                | test flaky.py |
| flapy example | ./minimal flaky python tests                                                         |              |          |                | test flaky.py |
| flapy example | ./minimal flaky python tests                                                         |              |          |                | test flaky.py |
