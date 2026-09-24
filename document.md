# Impacto do Padrão Adapter na Portabilidade de Aplicações Web Integradas a Provedores de Inteligência Artificial Generativa: um Estudo de Caso Comparativo

**Resumo:** Aplicações web que integram serviços de Inteligência Artificial Generativa frequentemente estabelecem dependências diretas com as APIs de provedores específicos, o que pode ampliar significativamente o esforço necessário para substituir um provedor, incorporar um novo serviço ou adaptar-se a mudanças na forma de comunicação com essas APIs. Este trabalho investiga, por meio de um estudo de caso comparativo, o impacto da adoção do padrão de projeto *Adapter* sobre a portabilidade e a manutenibilidade de aplicações web integradas a provedores de Inteligência Artificial Generativa. Para tanto, foi desenvolvido um protótipo de aplicação web em duas versões arquiteturais distintas: uma na qual a comunicação com o provedor de IA ocorre de forma acoplada e direta, e outra na qual essa comunicação é mediada por uma camada de abstração, implementada com base no padrão *Adapter*. Cenários controlados de mudança relacionados ao provedor de IA — como substituição de provedor, inclusão de um novo provedor e alteração na forma de comunicação com o serviço — foram aplicados a ambas as versões, permitindo a coleta e comparação de métricas objetivas de impacto, como número de arquivos e componentes modificados, linhas de código alteradas e testes afetados. Espera-se, com base na literatura consolidada sobre acoplamento e padrões de projeto, que a arquitetura desacoplada apresente impacto significativamente menor diante dos cenários de mudança avaliados, confirmando, em um contexto ainda pouco explorado empiricamente, benefícios já documentados do padrão *Adapter* em outros domínios da Engenharia de Software. *[Nota: esta última frase do resumo será revisada com os resultados reais após a execução do experimento.]*

**Palavras-chave:** *Arquitetura de Software; Padrão Adapter; Portabilidade de Software.*

**Abstract:** Web applications that integrate Generative Artificial Intelligence services often establish direct dependencies on specific provider APIs, which may significantly increase the effort required to replace a provider, incorporate a new service, or adapt to changes in how communication with these APIs occurs. This work investigates, through a comparative case study, the impact of adopting the Adapter design pattern on the portability and maintainability of web applications integrated with Generative AI providers. For this purpose, a web application prototype was developed in two distinct architectural versions: one in which communication with the AI provider occurs directly (coupled architecture), and another in which this communication is mediated by an abstraction layer implemented based on the Adapter pattern. Controlled change scenarios related to the AI provider — such as provider replacement, inclusion of a new provider, and changes in the communication format with the service — were applied to both versions, allowing the collection and comparison of objective impact metrics, such as the number of modified files and components, changed lines of code, and affected tests. *[Note: to be finalized with actual results.]*

**Keywords:** *Software Architecture; Adapter Pattern; Software Portability.*

## Introdução

A adoção de serviços de Inteligência Artificial Generativa em aplicações web tem se consolidado como uma tendência expressiva no desenvolvimento de software contemporâneo, impulsionada pela disponibilização de modelos de linguagem de grande escala (Large Language Models — LLMs) por meio de interfaces de programação de aplicações (APIs) oferecidas por provedores como Google (Gemini), OpenAI (GPT) e outros. Essa disponibilização em formato de serviço tem permitido que aplicações web incorporem, com relativa agilidade, capacidades de geração de texto, compreensão de linguagem natural e automação de tarefas cognitivas, sem que seja necessário o desenvolvimento ou o treinamento de modelos próprios.

Entretanto, essa forma de integração frequentemente é implementada por meio de dependências diretas entre a aplicação e a API específica de um provedor, uma prática que, sob a perspectiva da Engenharia de Software, caracteriza um alto grau de acoplamento entre a aplicação e um componente externo (CHIDAMBER; KEMERER, 1994). Esse tipo de dependência direta expõe a aplicação a riscos que já são conhecidos na literatura clássica de arquitetura de software havendo, contudo, ainda escassa investigação empírica sobre sua manifestação no contexto específico de provedores de Inteligência Artificial Generativa: entre esses riscos estão a dificuldade de substituição do provedor em caso de descontinuação do serviço, aumento de custos, mudanças na política de uso, ou mesmo a alteração da própria interface de comunicação da API, que pode ocorrer a qualquer momento por decisão do provedor e fora do controle da equipe de desenvolvimento.

A comunidade de Arquitetura de Software já reconhece a relevância dessa preocupação, ainda que de forma recente e ainda pouco consolidada: propostas de arquiteturas de referência para sistemas integrados a LLMs, como a apresentada por Bucaioni et al. (2025) na 22ª Conferência Internacional IEEE sobre Arquitetura de Software (ICSA), evidenciam que aspectos como modularidade e interoperabilidade se tornam centrais à medida que a complexidade e o impacto social desses sistemas aumentam. Ainda assim, tal proposta apresenta caráter preliminar e não se dedica a mensurar empiricamente o impacto de decisões arquiteturais específicas sobre a manutenibilidade desses sistemas, o que caracteriza uma lacuna que o presente trabalho busca contribuir para preencher, ainda que em escala reduzida e delimitada.

Na Engenharia de Software, uma solução amplamente consolidada para mitigar os efeitos do acoplamento direto a componentes externos é a adoção de camadas de abstração, por meio das quais a aplicação se comunica com uma interface genérica, e não diretamente com a implementação específica de um fornecedor. Um dos mecanismos clássicos para viabilizar essa abstração é o padrão de projeto *Adapter*, formalizado por Gamma et al. (1994), que permite que uma interface incompatível seja adaptada para o formato esperado pelo restante da aplicação, isolando as particularidades de implementação de um fornecedor externo em um componente específico e substituível.

A eficácia do padrão *Adapter* na melhoria da manutenibilidade de sistemas de software já foi investigada empiricamente em contextos gerais da Engenharia de Software. Al-Obeidallah et al. (2021), por exemplo, refatoraram quatro sistemas para gerar versões com e sem a aplicação do padrão, comparando métricas de software entre ambas as versões, e constataram que as versões que utilizavam o *Adapter* apresentaram métricas de manutenibilidade superiores às versões sem o padrão. Tal evidência, embora robusta, não foi obtida em um contexto que envolvesse a integração com serviços de Inteligência Artificial Generativa — um domínio de aplicação que apresenta características próprias, como a volatilidade acelerada das APIs dos provedores e a heterogeneidade dos formatos de requisição e resposta entre diferentes fornecedores.

Diante desse cenário, o presente trabalho tem como **objeto de pesquisa** o impacto arquitetural do acoplamento entre aplicações web e provedores de Inteligência Artificial Generativa, investigado por meio da implementação do padrão de projeto *Adapter* como mecanismo de abstração entre a aplicação e o provedor externo. O **problema de pesquisa** que orienta este estudo pode ser formulado da seguinte maneira: *em que medida a adoção de uma arquitetura desacoplada, baseada no padrão Adapter, reduz o impacto de mudanças relacionadas a provedores de Inteligência Artificial Generativa em uma aplicação web?*

Parte-se da **hipótese** de que uma arquitetura baseada em abstrações e adaptadores reduz significativamente o impacto de mudanças de provedor sobre a lógica de negócio e os demais componentes da aplicação, em consonância com o que já foi demonstrado empiricamente pela literatura de acoplamento e padrões de projeto em outros contextos da Engenharia de Software (AL-OBEIDALLAH et al., 2021).

Para investigar essa hipótese, define-se como **objetivo geral** deste trabalho investigar, por meio de um estudo de caso comparativo, o impacto da adoção do padrão de projeto *Adapter* sobre a portabilidade e a manutenibilidade de uma aplicação web integrada a provedores de Inteligência Artificial Generativa, comparando uma arquitetura acoplada e uma arquitetura desacoplada diante de cenários controlados de mudança de provedor. Como **objetivos específicos**, busca-se: (i) fundamentar teoricamente os conceitos de acoplamento, coesão, inversão de dependência e o padrão de projeto *Adapter*, à luz da literatura de Engenharia de Software e Arquitetura de Software; (ii) implementar um protótipo de aplicação web integrado a um provedor de Inteligência Artificial Generativa em duas versões arquiteturais distintas — uma acoplada diretamente ao provedor e outra desacoplada por meio de uma camada de abstração; (iii) definir e aplicar cenários controlados de mudança relacionados ao provedor de IA, como substituição de provedor ou alteração na forma de comunicação com o serviço, em ambas as versões do protótipo; (iv) coletar e comparar métricas objetivas de impacto entre as duas arquiteturas, como número de arquivos e componentes modificados, linhas de código alteradas e testes afetados; e (v) discutir os resultados obtidos à luz da literatura existente sobre acoplamento e padrões de projeto, avaliando em que medida a arquitetura desacoplada reduz o impacto de mudanças de provedor de IA sobre a aplicação.

A relevância deste trabalho se justifica em duas frentes complementares. Do ponto de vista prático, a crescente adoção de serviços de Inteligência Artificial Generativa por aplicações comerciais e institucionais torna a decisão sobre a forma de integração com esses provedores uma questão de impacto direto sobre a sustentabilidade técnica e financeira dos sistemas de software, especialmente diante de um mercado de provedores de IA ainda em rápida transformação, com mudanças frequentes de preços, políticas de uso e disponibilidade de modelos. Do ponto de vista acadêmico, embora a literatura sobre o impacto de padrões de projeto na manutenibilidade de software seja consolidada, sua aplicação específica ao contexto de integração com provedores de Inteligência Artificial Generativa ainda é incipiente, o que caracteriza uma oportunidade de contribuição empírica em um domínio de aplicação emergente e academicamente relevante.

Importa destacar, desde já, que este trabalho não se propõe a avaliar a qualidade ou o desempenho dos modelos de Inteligência Artificial Generativa utilizados, tampouco a desenvolver uma aplicação comercialmente completa. O protótipo desenvolvido tem finalidade estritamente instrumental, servindo como objeto de estudo controlado para a investigação da questão arquitetural proposta. Da mesma forma, por se tratar de um estudo de caso com um único sistema, implementado e avaliado por um único pesquisador, reconhece-se, desde a introdução, que os resultados obtidos não possuem pretensão de generalização estatística, constituindo antes uma evidência qualificada e contextualizada sobre o fenômeno investigado — limitação que será retomada e discutida com maior profundidade na seção de procedimentos metodológicos.

Este trabalho está organizado da seguinte forma: a seção 2 apresenta a fundamentação teórica que sustenta a pesquisa, abordando os conceitos de arquitetura de software, acoplamento e coesão, o padrão de projeto *Adapter* e a integração de aplicações com serviços de Inteligência Artificial Generativa; a seção 3 descreve os procedimentos metodológicos adotados, incluindo o protocolo de cenários de mudança e as métricas utilizadas; a seção 4 apresenta e discute os resultados obtidos a partir da comparação entre as duas versões arquiteturais do protótipo; e, por fim, a seção 5 apresenta as considerações finais do trabalho, retomando o problema de pesquisa e os objetivos propostos.

## Desenvolvimento

### Referencial Teórico

#### Arquitetura de Software e Atributos de Qualidade

A Arquitetura de Software pode ser compreendida como o conjunto de estruturas necessárias para o raciocínio sobre um sistema, compreendendo os elementos de software, as relações entre eles e as propriedades de ambos (BASS; CLEMENTS; KAZMAN, 2012). Decisões arquiteturais determinam, em grande medida, a capacidade de um sistema de atender a requisitos não funcionais — também denominados atributos de qualidade — entre os quais se destacam, para os fins deste trabalho, a **manutenibilidade** e a **portabilidade**.

A manutenibilidade refere-se ao esforço necessário para realizar alterações em um sistema de software após sua implantação, seja para corrigir defeitos, adaptar-se a novos requisitos ou incorporar melhorias (BASS; CLEMENTS; KAZMAN, 2012). Já a portabilidade, segundo a norma ISO/IEC 20233:2019, refere-se à capacidade de um sistema, ou de um de seus componentes, ser transferido de um ambiente para outro — no caso específico de sistemas que dependem de serviços externos, essa noção pode ser estendida para a capacidade de substituição de um provedor de serviço por outro com o mínimo de impacto possível sobre o restante da aplicação. Ambos os atributos são diretamente influenciados pelo grau de dependência que um sistema estabelece com componentes e serviços externos, o que remete diretamente aos conceitos de acoplamento e coesão, tratados a seguir.

#### Acoplamento e Coesão

O acoplamento e a coesão figuram entre os princípios mais fundamentais da Engenharia de Software para a avaliação da qualidade de um projeto de software. O acoplamento refere-se ao grau de interdependência entre os módulos de um sistema, sendo consenso na literatura que um baixo acoplamento é desejável, pois reduz o risco de que uma alteração em um módulo produza efeitos colaterais não previstos em outros módulos do sistema (CHIDAMBER; KEMERER, 1994). A coesão, por sua vez, refere-se ao grau em que os elementos internos de um módulo estão relacionados entre si em torno de uma responsabilidade única e bem definida, sendo desejável que essa coesão seja alta.

Chidamber e Kemerer (1994) propuseram uma suíte de métricas para a avaliação objetiva desses atributos em sistemas orientados a objetos, entre as quais se destaca a métrica CBO (*Coupling Between Objects*), que quantifica o número de classes às quais uma determinada classe está acoplada. Tal métrica fornece uma base objetiva e replicável para a mensuração do acoplamento, sendo amplamente adotada em estudos empíricos de Engenharia de Software — inclusive no estudo que fundamenta a metodologia adotada neste trabalho, discutido na subseção 2.4. Sistemas com alto acoplamento a componentes externos, como é o caso de aplicações que dependem diretamente da API de um provedor específico de serviço, tendem a apresentar maior dificuldade de manutenção e maior propagação de efeitos colaterais diante de mudanças nesse componente externo — fenômeno que este trabalho busca investigar empiricamente no contexto de provedores de Inteligência Artificial Generativa.

#### Padrões de Projeto e o Padrão Adapter

Os padrões de projeto (*design patterns*) constituem soluções reutilizáveis para problemas recorrentes de design de software, catalogadas e sistematizadas por Gamma et al. (1994) em obra que se tornou referência fundamental da área. Entre os padrões estruturais catalogados pelos autores, o **Adapter** tem como propósito converter a interface de uma classe em outra interface esperada pelos clientes, permitindo que classes com interfaces incompatíveis colaborem entre si sem que seja necessário modificar seu código-fonte original (GAMMA et al., 1994).

A estrutura clássica do padrão Adapter é composta por quatro elementos: (i) o **Target**, que define a interface específica utilizada pelo código cliente; (ii) o **Adaptee**, que representa a classe ou componente existente cuja interface é incompatível com a esperada pelo cliente; (iii) o **Adapter**, componente concreto que implementa a interface Target e internamente traduz as chamadas recebidas para o formato compreendido pelo Adaptee; e (iv) o **Client**, que consome exclusivamente a interface Target, sem conhecimento direto da implementação do Adaptee.

Aplicado ao contexto deste trabalho, o Adaptee corresponde à API específica de um provedor de Inteligência Artificial Generativa (por exemplo, a API do Gemini), com seu formato particular de requisição e resposta; o Target corresponde a uma interface genérica definida pela própria aplicação (por exemplo, um método abstrato responsável por solicitar a geração de uma resposta textual); e o Adapter corresponde à implementação concreta responsável por traduzir as chamadas dessa interface genérica para o formato específico exigido pela API do provedor em uso. Dessa forma, a lógica de negócio da aplicação — representada pelo Client — permanece isolada das particularidades de implementação de qualquer provedor específico, dependendo exclusivamente da interface Target.

#### Evidências Empíricas do Impacto do Adapter Pattern na Manutenibilidade

Embora a literatura reconheça, em nível teórico, os benefícios do uso de padrões de projeto para a redução do acoplamento e o aumento da manutenibilidade, a comunidade de Engenharia de Software não possui consenso unânime sobre a magnitude real desses benefícios quando mensurados empiricamente (AL-OBEIDALLAH et al., 2021), o que reforça a importância de estudos que produzam evidências objetivas sobre esse impacto.

Nesse sentido, destaca-se o estudo conduzido por Al-Obeidallah et al. (2021), que investigou especificamente o impacto do padrão Adapter sobre a manutenibilidade de software. Os autores aplicaram técnicas de refatoração para gerar, a partir de quatro sistemas de software distintos, versões equivalentes com e sem a aplicação do padrão Adapter, e em seguida calcularam e compararam métricas de software entre ambas as versões, adotando correlações entre essas métricas e a manutenibilidade já estabelecidas por pesquisas anteriores. Os resultados obtidos indicaram que as versões que incorporavam o padrão Adapter apresentaram métricas de software superiores às versões sem o padrão, evidenciando um impacto positivo do Adapter sobre a manutenibilidade dos sistemas avaliados.

Esse desenho metodológico — a comparação de métricas objetivas entre uma versão de um sistema com um determinado padrão arquitetural e uma versão equivalente sem esse padrão, diante de cenários de mudança controlados — constitui a principal referência metodológica adotada neste trabalho, com a distinção de que, em vez de comparar a presença ou ausência do padrão Adapter em sistemas genéricos já existentes, o presente estudo aplica esse mesmo raciocínio comparativo a um protótipo desenvolvido especificamente para investigar o contexto de integração com provedores de Inteligência Artificial Generativa, conforme detalhado na seção de procedimentos metodológicos.

Estudos correlatos reforçam a mesma direção de evidências: Qasim et al. (2021) igualmente constataram, por meio de avaliação empírica comparando uma solução sem padrões de projeto com sua versão refinada, ganhos significativos de manutenibilidade após a introdução de padrões de projeto adequados. De maneira convergente, revisões sistemáticas da literatura, como a conduzida por Wedyan e Abufakher (2019), apontam que, apesar de eventuais divergências pontuais entre estudos, a tendência predominante na literatura empírica é de associação positiva entre o uso adequado de padrões de projeto e a melhoria de atributos de qualidade como a manutenibilidade.

#### Integração de Aplicações Web com Provedores de Inteligência Artificial Generativa

A disponibilização de modelos de Inteligência Artificial Generativa por meio de APIs comerciais tem impulsionado sua rápida incorporação em aplicações web das mais diversas naturezas. Contudo, diferentemente de componentes de software tradicionais, os provedores de IA Generativa operam em um mercado caracterizado por rápida evolução técnica e comercial, com atualizações frequentes de modelos, alterações nos formatos de requisição e resposta das APIs, e mudanças nas políticas de preço e disponibilidade dos serviços — fatores que ampliam a relevância prática da discussão sobre acoplamento nesse domínio específico de aplicação.

Do ponto de vista acadêmico, a discussão sobre arquitetura de software para sistemas integrados a modelos de linguagem de grande escala ainda é incipiente, mas já apresenta sinais de consolidação. Bucaioni et al. (2025), em trabalho apresentado na 22ª Conferência Internacional IEEE sobre Arquitetura de Software (ICSA), propuseram uma arquitetura de referência funcional preliminar para sistemas integrados a LLMs, destacando a modularidade e a interoperabilidade como preocupações arquiteturais centrais diante da crescente complexidade desses sistemas. Os autores, no entanto, não se dedicaram à mensuração empírica do impacto de decisões arquiteturais específicas — como a adoção de camadas de abstração — sobre atributos de qualidade mensuráveis, o que representa uma lacuna à qual o presente trabalho busca contribuir, ainda que em escopo delimitado e a partir de um único caso de estudo.

#### Síntese e Lacuna de Pesquisa

A revisão da literatura evidencia que, embora o impacto positivo do padrão Adapter sobre a manutenibilidade de software já tenha sido demonstrado empiricamente em contextos gerais da Engenharia de Software (AL-OBEIDALLAH et al., 2021; QASIM et al., 2021), e embora a comunidade de Arquitetura de Software já reconheça a relevância de preocupações arquiteturais em sistemas integrados a modelos de Inteligência Artificial Generativa (BUCAIONI et al., 2025), não foram identificados, até o presente momento, estudos empíricos que mensurem especificamente o impacto da adoção do padrão Adapter sobre a portabilidade e a manutenibilidade de aplicações web diante de cenários de mudança relacionados a provedores de Inteligência Artificial Generativa. É exatamente essa lacuna que o presente trabalho se propõe a investigar, aplicando um desenho metodológico consolidado na literatura de padrões de projeto a um contexto de aplicação emergente e ainda pouco explorado empiricamente.

### Metodologia

#### Classificação da Pesquisa

Do ponto de vista da natureza, esta pesquisa classifica-se como aplicada, uma vez que se propõe a gerar conhecimento com aplicação prática direta sobre um problema real de Engenharia de Software, sem a pretensão de desenvolver teoria original. Quanto à abordagem, a pesquisa é predominantemente quantitativa na etapa de coleta e comparação de métricas de software, complementada por uma análise qualitativa na discussão dos resultados à luz da literatura. Quanto ao procedimento técnico, adota-se o **estudo de caso comparativo**, seguindo as diretrizes propostas por Runeson e Höst (2009) para a condução e o relato de estudos de caso em Engenharia de Software, sendo o caso investigado constituído por um único protótipo de aplicação web, implementado em duas versões arquiteturais distintas.

Optou-se deliberadamente por não classificar este trabalho como um experimento controlado no sentido estatístico do termo, uma vez que o estudo envolve uma única unidade de análise (um protótipo), sem grupo de controle independente, sem randomização e sem repetição amostral que permita inferência estatística generalizável. Trata-se, portanto, de um estudo de caso comparativo de caráter exploratório, cujos resultados devem ser interpretados como evidência contextualizada e não como generalização estatística — limitação retomada com maior profundidade na subseção de ameaças à validade.

#### Descrição do Protótipo e das Versões Arquiteturais

O objeto de estudo consiste em um protótipo de aplicação web simples, desenvolvido especificamente para os fins desta pesquisa, cuja funcionalidade central é receber uma entrada textual do usuário e retornar uma resposta gerada por um provedor de Inteligência Artificial Generativa. Optou-se deliberadamente por manter o escopo funcional do protótipo reduzido, uma vez que o objeto de investigação deste trabalho não é a aplicação em si, mas sim o impacto arquitetural das decisões de design adotadas para sua integração com o provedor de IA.

O provedor de IA Generativa inicialmente integrado ao protótipo é a API do Gemini (Google), podendo ser complementado por um segundo provedor ou por um provedor simulado (*mock*) durante a execução dos cenários de mudança descritos na subseção 3.3. O protótipo é implementado em duas versões funcionalmente equivalentes, porém arquiteturalmente distintas:

- **Versão A — Arquitetura Acoplada:** nesta versão, a comunicação com a API do provedor de IA é realizada de forma direta a partir dos componentes da aplicação que dependem dessa funcionalidade, sem qualquer camada de abstração intermediária. As particularidades do formato de requisição e resposta da API do provedor ficam, portanto, diretamente expostas aos componentes consumidores.

- **Versão B — Arquitetura Desacoplada:** nesta versão, a comunicação com o provedor de IA é mediada por uma interface de abstração (Target), definida pela própria aplicação, e por uma implementação concreta do padrão *Adapter*, responsável por traduzir as chamadas dessa interface genérica para o formato específico exigido pela API do provedor em uso (Adaptee), conforme a estrutura teórica apresentada na subseção 2.3. Os componentes consumidores da aplicação (Client) dependem exclusivamente da interface Target, sem conhecimento direto da implementação do provedor.

Ambas as versões compartilham a mesma lógica de negócio e a mesma interface de usuário, diferenciando-se exclusivamente na forma de comunicação com o provedor de IA — condição necessária para assegurar que as diferenças de impacto observadas entre as duas versões possam ser atribuídas à decisão arquitetural investigada, e não a diferenças funcionais entre elas.

#### Protocolo de Cenários de Mudança

Em conformidade com a recomendação metodológica de definir o protocolo de coleta de dados previamente à sua execução — mitigando o risco de viés do pesquisador, discutido na subseção 3.6 — foram definidos três cenários controlados de mudança, aplicados de forma idêntica a ambas as versões do protótipo:

1. **Cenário 1 — Substituição de provedor:** simula a necessidade de substituir integralmente o provedor de IA Generativa utilizado (por exemplo, migrar da API do Gemini para a API de outro provedor), mantendo a funcionalidade da aplicação inalterada do ponto de vista do usuário final.

2. **Cenário 2 — Inclusão de um novo provedor:** simula a necessidade de incorporar um segundo provedor de IA Generativa à aplicação, mantendo o provedor original em funcionamento e permitindo que a aplicação alterne entre ambos.

3. **Cenário 3 — Alteração na forma de comunicação com o provedor:** simula uma mudança no formato de requisição ou resposta da API do provedor atualmente em uso — situação que reflete uma ocorrência real e recorrente no mercado de provedores de IA Generativa, no qual atualizações de versão de API, descontinuação de modelos e alterações de esquema de dados ocorrem com frequência e fora do controle da equipe de desenvolvimento.

Cada cenário é implementado separadamente em ambas as versões do protótipo, a partir de um estado inicial idêntico (mesmo *commit* de referência), permitindo que as alterações realizadas em cada versão sejam isoladas e comparadas de forma independente.

#### Métricas de Avaliação

Para cada cenário aplicado a cada versão do protótipo, são coletadas as seguintes métricas objetivas, com base no controle de versão do código-fonte (Git):

- **Número de arquivos modificados:** quantidade de arquivos alterados para a implementação do cenário, obtida por meio do comando `git diff --stat` entre o estado inicial e o estado final da alteração.
- **Número de linhas de código alteradas:** soma de linhas adicionadas e removidas para a implementação do cenário, também obtida via `git diff --stat`.
- **Número de pontos de acoplamento direto ao provedor:** contagem manual dos pontos do código-fonte em que há referência direta à biblioteca ou à API específica do provedor de IA fora da camada de abstração (aplicável apenas à Versão A, servindo como métrica de caracterização do grau de acoplamento inicial).
- **Número de testes automatizados afetados:** quantidade de testes que precisaram ser criados, removidos ou modificados para que a suíte de testes voltasse a passar após a implementação do cenário.

Optou-se por não adotar métricas subjetivas de esforço (como percepção de dificuldade) como critério central de avaliação, justamente por se tratar de métricas sujeitas a viés de quem implementa e mede simultaneamente — decisão diretamente relacionada às ameaças à validade discutidas a seguir.

#### Procedimento de Execução

A execução da coleta de dados segue a seguinte sequência: (i) implementação completa da Versão A (arquitetura acoplada) até um estado funcional estável, correspondendo ao estado inicial de referência; (ii) aplicação sequencial dos três cenários de mudança sobre a Versão A, com coleta das métricas descritas para cada cenário, restaurando o estado de referência entre um cenário e outro; (iii) implementação completa da Versão B (arquitetura desacoplada), com escopo funcional equivalente à Versão A; (iv) aplicação sequencial dos mesmos três cenários de mudança sobre a Versão B, com coleta das mesmas métricas; e (v) consolidação e comparação dos resultados obtidos entre as duas versões, para cada cenário, conforme apresentado na seção de Resultados.

#### Ameaças à Validade

Em conformidade com o modelo de discussão de ameaças à validade proposto por Runeson e Höst (2009) para estudos de caso em Engenharia de Software, reconhecem-se as seguintes limitações metodológicas deste trabalho:

- **Validade de construto:** as métricas adotadas (número de arquivos, linhas de código e testes afetados) constituem indicadores indiretos do esforço de manutenção, e não uma medida direta de "impacto" — decisão adotada por serem métricas objetivas e reproduzíveis, em detrimento de métricas subjetivas de esforço, mais sujeitas a viés.
- **Validade interna:** o autor deste trabalho foi responsável tanto pela implementação de ambas as versões do protótipo quanto pela coleta das métricas, o que constitui uma ameaça relacionada ao viés do pesquisador-implementador. Essa ameaça é parcialmente mitigada pela definição prévia e documentada do protocolo de cenários de mudança, anterior à implementação de qualquer uma das versões, reduzindo a possibilidade de ajuste inconsciente da implementação em função do resultado esperado.
- **Validade externa:** por se tratar de um estudo de caso com uma única unidade de análise, implementado por um único desenvolvedor e restrito a um domínio de aplicação específico (integração com provedores de IA Generativa em aplicações web simples), os resultados obtidos não podem ser generalizados estatisticamente para outros sistemas, domínios de aplicação ou desenvolvedores, constituindo evidência contextualizada e não uma conclusão universal.
- **Confiabilidade:** buscou-se mitigar essa ameaça por meio da documentação detalhada do protocolo experimental apresentado nesta seção, de forma a permitir que outros pesquisadores possam reproduzir o procedimento descrito, ainda que com resultados numéricos potencialmente distintos em função de decisões de implementação específicas.

### Resultados Parciais — Versão Acoplada

> **Nota sobre o estado da coleta:** até o presente momento, a coleta de dados foi concluída integralmente para a Versão A (arquitetura acoplada), contemplando os três cenários de mudança definidos na subseção 3.3. A implementação e a coleta de métricas da Versão B (arquitetura desacoplada) encontram-se em andamento. Por esse motivo, esta seção apresenta os resultados obtidos exclusivamente para a Versão A, com caráter preliminar. A análise comparativa entre as duas arquiteturas — núcleo do problema de pesquisa deste trabalho — será apresentada na versão final desta seção, tão logo a coleta referente à Versão B seja concluída, momento em que esta subseção será integrada a uma seção unificada de Resultados, conforme a estrutura anunciada na Introdução.

#### Consolidação das Métricas — Versão Acoplada

A Tabela 1 apresenta a consolidação das métricas objetivas coletadas para os três cenários de mudança aplicados à Versão A, obtidas por meio do comando `git diff --stat` entre o estado de referência da versão e o estado resultante da aplicação de cada cenário, conforme os critérios descritos na subseção 3.4.

**Tabela 1 — Métricas consolidadas por cenário (Versão Acoplada)**

| Cenário | Arquivos alterados | Linhas adicionadas | Linhas removidas | Total de linhas tocadas | Testes automatizados afetados |
|---|---|---|---|---|---|
| 1 — Substituição de provedor | 9 | 160 | 173 | 333 | 0 |
| 2 — Inclusão de novo provedor | 9 | 364 | 24 | 388 | 0 |
| 3 — Alteração de formato de comunicação | 4 | 226 | 9 | 235 | 5 (1 arquivo novo) |

As Tabelas 2, 3 e 4 detalham o impacto por arquivo em cada cenário, permitindo identificar não apenas a magnitude, mas também a distribuição da mudança ao longo dos componentes do sistema.

**Tabela 2 — Detalhamento por arquivo, Cenário 1 (Substituição de provedor)**

| Arquivo | Tipo de alteração | Linhas tocadas |
|---|---|---|
| `GeminiRequest.java` | Deletado | 63 |
| `GeminiResponse.java` | Deletado | 65 |
| `GroqRequest.java` | Criado | 68 |
| `GroqResponse.java` | Criado | 52 |
| `GeminiConfig.java` → `GroqConfig.java` | Renomeado/modificado | 12 |
| `GeminiProperties.java` → `GroqProperties.java` | Renomeado/modificado | 16 |
| `ProvedorIndisponivelException.java` | Modificado | 1 |
| `ResumoService.java` | Modificado | 45 |
| `application.properties` | Modificado | 10 |

**Tabela 3 — Detalhamento por arquivo, Cenário 2 (Inclusão de novo provedor)**

| Arquivo | Tipo de alteração | Linhas adicionadas | Linhas removidas |
|---|---|---|---|
| `GroqRequest.java` | Criado | 67 | 0 |
| `GroqResponse.java` | Criado | 51 | 0 |
| `GroqConfig.java` | Criado | 41 | 0 |
| `GroqProperties.java` | Criado | 48 | 0 |
| `ResumoRequest.java` | Modificado | 11 | 0 |
| `GlobalExceptionHandler.java` | Modificado | 6 | 0 |
| `ProvedorInvalidoException.java` | Criado | 12 | 0 |
| `ResumoService.java` | Modificado | 144 | 24 |
| `application.properties` | Modificado | 8 | 0 |

**Tabela 4 — Detalhamento por arquivo, Cenário 3 (Alteração de formato de comunicação)**

| Arquivo | Tipo de alteração | Linhas adicionadas | Linhas removidas |
|---|---|---|---|
| `GeminiRequest.java` | Modificado | 39 | 0 |
| `GeminiResponse.java` | Modificado | 19 | 5 |
| `ResumoService.java` | Modificado | 21 | 4 |
| `ResumoServiceExtracaoGeminiTest.java` | Criado (teste) | 147 | 0 |

#### Discussão Preliminar — Padrões Observados dentro da Versão Acoplada

Ainda que a comparação central deste trabalho (Versão Acoplada versus Versão Desacoplada) permaneça pendente, os dados já coletados permitem observações preliminares relevantes sobre o comportamento da arquitetura acoplada diante de diferentes naturezas de mudança relacionada a provedor de IA Generativa.

Em primeiro lugar, observa-se que os Cenários 1 e 2 — ambos envolvendo a incorporação de um novo provedor à aplicação, seja em substituição (Cenário 1), seja em adição (Cenário 2) — apresentaram impacto de magnitude semelhante em número de arquivos (9 em ambos), porém com perfis distintos de composição: o Cenário 1 apresentou volume expressivo de remoção de código (173 linhas), decorrente da exclusão das classes específicas do provedor substituído, enquanto o Cenário 2 apresentou remoção mínima (24 linhas), concentrada exclusivamente na reescrita do ponto de decisão de roteamento entre provedores no `ResumoService`. Tal diferença evidencia que, mesmo dentro da arquitetura acoplada, tarefas de manutenção nominalmente similares ("lidar com um novo provedor") podem apresentar assinaturas de impacto substancialmente diferentes a depender da natureza exata da mudança solicitada — achado que reforça a importância de definir cenários de mudança específicos e diversificados, em vez de tratar "mudança de provedor" como uma categoria única e homogênea.

Em segundo lugar, o Cenário 3 apresentou o menor impacto entre os três em termos de código de produção (79 linhas somando as três classes efetivamente modificadas: `GeminiRequest`, `GeminiResponse` e `ResumoService`), o que é consistente com a expectativa teórica de que uma mudança de contrato dentro do mesmo provedor — sem troca ou adição de fornecedor — deveria demandar menos esforço do que os Cenários 1 e 2. Entretanto, esse cenário foi o único, entre os três, a exigir a criação de testes automatizados (147 linhas em um arquivo de teste novo), o que elevou seu impacto total a 235 linhas — segundo colocado entre os três cenários. Esse resultado ilustra uma nuance metodologicamente relevante: o Cenário 3 representa uma mudança de contrato hipotética e prospectiva, para a qual não existe, no momento da coleta, uma API real correspondente contra a qual validar a implementação; a ausência dessa validação natural deslocou o custo de garantia de corretude do processo de teste manual (possível nos Cenários 1 e 2, validados diretamente contra as APIs reais do Gemini e do Groq) para a criação de infraestrutura de teste automatizado. Esse achado sugere que o custo de manutenção de uma arquitetura acoplada, diante de mudanças de contrato ainda não materializadas pelo provedor, pode se manifestar não apenas em código de produção, mas também no esforço adicional de construção de mecanismos de validação — uma dimensão de impacto não capturada pelas métricas de código de produção isoladamente, e que será retomada na discussão comparativa final deste trabalho.

Essas observações preliminares serão reexaminadas e contrastadas com os resultados equivalentes da Versão Desacoplada na versão final desta seção, de modo a responder de forma completa ao problema de pesquisa proposto.
## Considerações Finais

> **Nota sobre o estado desta seção:** esta seção será redigida após a conclusão da coleta de dados da Versão Desacoplada e da consequente análise comparativa entre as duas arquiteturas, retomando o problema de pesquisa, confirmando ou refutando a hipótese formulada na Introdução, e sintetizando as contribuições e limitações do trabalho.

## Referências

AL-OBEIDALLAH, M. G.; AL-FRAIHAT, D. G.; KHASAWNEH, A. M.; SALEH, A. M.; ADDOUS, H. Empirical Investigation of the Impact of the Adapter Design Pattern on Software Maintainability. In: INTERNATIONAL CONFERENCE ON INFORMATION TECHNOLOGY (ICIT), 2021, Amman. **Anais [...]**. IEEE, 2021. p. 206-211.

BASS, L.; CLEMENTS, P.; KAZMAN, R. **Software Architecture in Practice**. 3. ed. Boston: Addison-Wesley, 2012.

BUCAIONI, A.; WEYSSOW, M. et al. A Functional Software Reference Architecture for LLM-Integrated Systems. In: IEEE INTERNATIONAL CONFERENCE ON SOFTWARE ARCHITECTURE COMPANION (ICSA-C), 22., 2025, Odense. **Anais [...]**. IEEE, 2025. p. 1-5.

CHIDAMBER, S. R.; KEMERER, C. F. A Metrics Suite for Object-Oriented Design. **IEEE Transactions on Software Engineering**, v. 20, n. 6, p. 476-493, 1994.

GAMMA, E.; HELM, R.; JOHNSON, R.; VLISSIDES, J. **Design Patterns: Elements of Reusable Object-Oriented Software**. Boston: Addison-Wesley, 1994.

INTERNATIONAL ORGANIZATION FOR STANDARDIZATION. **ISO/IEC 20233:2019 — Information technology — Cloud computing — Interoperability and portability**. Geneva: ISO, 2019.

QASIM, A.; MUNAWAR, A.; HASSAN, J.; KHALID, A. Evaluating the Impact of Design Patterns on Software Maintainability: An Empirical Evaluation. In: INTERNATIONAL SUSTAINABILITY AND RESILIENCE CONFERENCE: CLIMATE CHANGE, 3., 2021, Sakheer. **Anais [...]**. IEEE, 2021.

RUNESON, P.; HÖST, M. Guidelines for Conducting and Reporting Case Study Research in Software Engineering. **Empirical Software Engineering**, v. 14, n. 2, p. 131-164, 2009.

WEDYAN, F.; ABUFAKHER, S. Impact of Design Patterns on Software Quality: A Systematic Literature Review. **IET Software**, v. 14, n. 1, p. 1-17, 2019.
