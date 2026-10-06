# Impacto do Padrão Adapter na Portabilidade de Aplicações Web Integradas a Provedores de Inteligência Artificial Generativa: um Estudo de Caso Comparativo

**Resumo:** Aplicações web que integram serviços de Inteligência Artificial Generativa costumam depender diretamente da API de um provedor específico, o que expõe o código da aplicação a mudanças decididas pelo provedor, como a descontinuação de modelos e a alteração de formatos de comunicação. Este trabalho investiga, por meio de um estudo de caso comparativo, em que medida a adoção de uma arquitetura desacoplada, baseada no padrão de projeto *Adapter*, reduz o impacto dessas mudanças. Um protótipo de aplicação web para resumo de textos acadêmicos, desenvolvido em Java com Spring Boot, foi implementado em duas versões funcionalmente equivalentes: uma acoplada, em que a classe de serviço se comunica diretamente com a API do provedor, e uma desacoplada, em que essa comunicação é mediada por uma interface definida pela aplicação e por adaptadores específicos de cada provedor. Três cenários controlados de mudança — substituição do provedor, inclusão de um segundo provedor e alteração do formato de comunicação — foram aplicados às duas versões, e o impacto foi medido a partir do histórico do Git, em arquivos e linhas alterados, distinguindo o código dedicado ao provedor, os testes e os demais componentes. Somados os cenários, o volume total de alterações foi maior ou equivalente na versão desacoplada: 1.249 contra 1.106 linhas tocadas, ou 961 contra 956 com a detecção de renomeações do Git ativa. Em contrapartida, o código alterado fora do pacote do provedor caiu de 245 para 61 linhas: foi nulo nos cenários de substituição e de alteração de formato e cerca de 65% menor no cenário de inclusão. Conclui-se que, no caso estudado, o desacoplamento não reduziu o esforço bruto das mudanças, mas as confinou ao código específico do provedor, preservando a lógica de negócio, ao custo de mais estrutura por provedor. Por se tratar de um caso único, os resultados são descritivos e não permitem generalização estatística.

**Palavras-chave:** *Arquitetura de Software; Padrão Adapter; Portabilidade de Software.*

**Abstract:** Web applications that integrate Generative Artificial Intelligence services often depend directly on the API of a specific provider, which exposes the application code to changes decided by the provider, such as model deprecation and changes in communication formats. Through a comparative case study, this work investigates to what extent a decoupled architecture based on the Adapter design pattern reduces the impact of such changes. A web application prototype for summarizing academic texts, developed in Java with Spring Boot, was implemented in two functionally equivalent versions: a coupled one, in which the service class communicates directly with the provider's API, and a decoupled one, in which this communication is mediated by an interface defined by the application and by provider-specific adapters. Three controlled change scenarios — provider replacement, inclusion of a second provider, and a change in the communication format — were applied to both versions, and their impact was measured from the Git history, in changed files and lines, distinguishing provider-specific code, tests, and the remaining components. Across the three scenarios, the total volume of changes was larger or equivalent in the decoupled version: 1,249 versus 1,106 changed lines, or 961 versus 956 with Git rename detection enabled. In contrast, the code changed outside the provider package dropped from 245 to 61 lines: it was zero in the replacement and format-change scenarios and about 65% lower in the inclusion scenario. In the studied case, decoupling did not reduce the raw effort of the changes, but confined them to provider-specific code, preserving the business logic at the cost of additional structure per provider. As a single case, the results are descriptive and do not support statistical generalization.

**Keywords:** *Software Architecture; Adapter Pattern; Software Portability.*

## 1 Introdução

A adoção de serviços de Inteligência Artificial Generativa em aplicações web tem se consolidado como uma tendência expressiva no desenvolvimento de software contemporâneo, impulsionada pela disponibilização de modelos de linguagem de grande escala (*Large Language Models* — LLMs) por meio de interfaces de programação de aplicações (APIs) oferecidas por provedores como Google (Gemini), OpenAI (GPT) e outros. Essa disponibilização em formato de serviço tem permitido que aplicações web incorporem, com relativa agilidade, capacidades de geração de texto, compreensão de linguagem natural e automação de tarefas cognitivas, sem que seja necessário desenvolver ou treinar modelos próprios.

Entretanto, essa integração frequentemente é implementada por meio de dependências diretas entre a aplicação e a API específica de um provedor, o que, sob a perspectiva da Engenharia de Software, caracteriza acoplamento entre a aplicação e um componente externo, entendido como interdependência entre módulos (CHIDAMBER; KEMERER, 1994). Esse tipo de dependência expõe a aplicação a mudanças decididas pelo provedor e fora do controle da equipe de desenvolvimento: a descontinuação de modelos, o aumento de custos, a mudança na política de uso e a alteração da própria interface de comunicação da API. A descontinuação, em particular, faz parte do ciclo de vida desses serviços: o Google mantém uma página pública com o calendário de desligamento dos modelos da API do Gemini, que indica, para cada modelo, o modelo recomendado como substituto (GOOGLE, [2026]).

A comunidade de Arquitetura de Software já reconhece a relevância dessa preocupação, ainda que de forma recente e pouco consolidada: Bucaioni et al. (2025), na 22ª Conferência Internacional IEEE sobre Arquitetura de Software (ICSA), apontam privacidade, segurança, modularidade e interoperabilidade como preocupações críticas em sistemas integrados a LLMs, à medida que esses sistemas crescem em complexidade e impacto social. A proposta desses autores, porém, tem caráter preliminar e não mensura o impacto de decisões arquiteturais específicas sobre a manutenibilidade, o que caracteriza uma lacuna que o presente trabalho busca contribuir para preencher, em escala reduzida e delimitada.

Na Engenharia de Software, uma solução consolidada para mitigar os efeitos do acoplamento direto a componentes externos é a adoção de camadas de abstração, por meio das quais a aplicação se comunica com uma interface genérica, e não diretamente com a implementação específica de um fornecedor. Um dos mecanismos clássicos para viabilizar essa abstração é o padrão de projeto *Adapter*, formalizado por Gamma et al. (1994), que permite adaptar uma interface incompatível ao formato esperado pelo restante da aplicação, isolando as particularidades de um fornecedor externo em um componente específico e substituível.

A eficácia do padrão *Adapter* na melhoria da manutenibilidade já foi investigada empiricamente em contextos gerais da Engenharia de Software. Al-Obeidallah et al. (2021), por exemplo, refatoraram quatro sistemas para gerar versões com e sem a aplicação do padrão, compararam métricas de software entre as versões e constataram que as versões que utilizavam o *Adapter* apresentaram métricas de manutenibilidade superiores. Essa evidência, contudo, não foi obtida em um contexto de integração com serviços de Inteligência Artificial Generativa, domínio que apresenta características próprias, como a descontinuação periódica de modelos e a heterogeneidade dos formatos de requisição e resposta entre fornecedores.

Assim, o presente trabalho tem como **objeto de pesquisa** o impacto arquitetural do acoplamento entre aplicações web e provedores de Inteligência Artificial Generativa, investigado por meio da implementação do padrão de projeto *Adapter* como mecanismo de abstração entre a aplicação e o provedor externo. O **problema de pesquisa** que orienta este estudo pode ser formulado da seguinte maneira: *em que medida a adoção de uma arquitetura desacoplada, baseada no padrão Adapter, reduz o impacto de mudanças relacionadas a provedores de Inteligência Artificial Generativa em uma aplicação web?*

Parte-se da **hipótese** de que uma arquitetura baseada em abstrações e adaptadores reduz o impacto de mudanças de provedor sobre a lógica de negócio e os demais componentes da aplicação, em consonância com resultados empíricos obtidos para o padrão em outros contextos da Engenharia de Software (AL-OBEIDALLAH et al., 2021).

Para investigar essa hipótese, define-se como **objetivo geral** investigar, por meio de um estudo de caso comparativo, o impacto da adoção do padrão de projeto *Adapter* sobre a portabilidade e a manutenibilidade de uma aplicação web integrada a provedores de Inteligência Artificial Generativa, comparando uma arquitetura acoplada e uma arquitetura desacoplada diante de cenários controlados de mudança de provedor. Como **objetivos específicos**, busca-se: (i) fundamentar teoricamente os conceitos de acoplamento, coesão, inversão de dependência e o padrão de projeto *Adapter*, à luz da literatura de Engenharia de Software e Arquitetura de Software; (ii) implementar um protótipo de aplicação web integrado a um provedor de Inteligência Artificial Generativa em duas versões arquiteturais distintas — uma acoplada diretamente ao provedor e outra desacoplada por meio de uma camada de abstração; (iii) definir e aplicar cenários controlados de mudança relacionados ao provedor de IA, como substituição de provedor ou alteração na forma de comunicação com o serviço, em ambas as versões do protótipo; (iv) coletar e comparar métricas objetivas de impacto entre as duas arquiteturas, como número de arquivos e componentes modificados, linhas de código alteradas e testes afetados; e (v) discutir os resultados obtidos à luz da literatura existente sobre acoplamento e padrões de projeto, avaliando em que medida a arquitetura desacoplada reduz o impacto de mudanças de provedor de IA sobre a aplicação.

A relevância deste trabalho se justifica em duas frentes complementares. Do ponto de vista prático, a crescente adoção de serviços de Inteligência Artificial Generativa por aplicações comerciais e institucionais torna a decisão sobre a forma de integração com esses provedores uma questão de impacto direto sobre a sustentabilidade técnica dos sistemas de software, sobretudo em um mercado em que a disponibilidade de cada modelo tem prazo definido pelo provedor (GOOGLE, [2026]). Do ponto de vista acadêmico, embora a literatura sobre o impacto de padrões de projeto na manutenibilidade de software seja extensa, sua aplicação ao contexto de integração com provedores de Inteligência Artificial Generativa ainda é incipiente, o que caracteriza uma oportunidade de contribuição empírica em um domínio emergente.

Este trabalho não se propõe a avaliar a qualidade ou o desempenho dos modelos de Inteligência Artificial Generativa utilizados, tampouco a desenvolver uma aplicação comercialmente completa. O protótipo tem finalidade instrumental, servindo como objeto de estudo controlado para a investigação da questão arquitetural proposta. Por se tratar de um estudo de caso com um único sistema, os resultados não possuem pretensão de generalização estatística e constituem evidência contextualizada sobre o fenômeno investigado, limitação discutida na subseção 3.6.

Os resultados indicam que, no caso estudado, a arquitetura desacoplada não reduziu o volume total das alterações exigidas pelas mudanças de provedor, mas as concentrou no código dedicado ao provedor, eliminando ou reduzindo as alterações na lógica de negócio. A contribuição do trabalho é, portanto, dupla: uma evidência empírica, em pequena escala, sobre o efeito do desacoplamento nesse domínio, e um procedimento reprodutível de medição do impacto de mudanças que distingue o código do provedor dos demais componentes.

Este trabalho está organizado da seguinte forma: a seção 2 apresenta a fundamentação teórica e os trabalhos relacionados, abordando arquitetura de software e atributos de qualidade, acoplamento e coesão, o padrão *Adapter* e a inversão de dependência, e a integração de aplicações com serviços de Inteligência Artificial Generativa; a seção 3 descreve os procedimentos metodológicos, incluindo o protótipo, o protocolo de cenários de mudança e as métricas; a seção 4 apresenta e discute os resultados da comparação entre as duas versões; e a seção 5 apresenta as considerações finais, retomando o problema de pesquisa e os objetivos.

## 2 Referencial Teórico

### 2.1 Arquitetura de Software e Atributos de Qualidade

A Arquitetura de Software pode ser compreendida como o conjunto de estruturas necessárias para o raciocínio sobre um sistema, compreendendo os elementos de software, as relações entre eles e as propriedades de ambos (BASS; CLEMENTS; KAZMAN, 2012). Decisões arquiteturais determinam, em grande medida, a capacidade de um sistema de atender a requisitos não funcionais — também denominados atributos de qualidade —, entre os quais se destacam, para os fins deste trabalho, a **manutenibilidade** e a **portabilidade**.

Este trabalho adota as definições do modelo de qualidade de produto da norma ISO/IEC 25010 (INTERNATIONAL ORGANIZATION FOR STANDARDIZATION, 2011). Nela, a manutenibilidade corresponde ao grau de eficácia e eficiência com que um produto pode ser modificado por seus mantenedores e se decompõe em subcaracterísticas, entre as quais a **modularidade**, grau em que um sistema é composto de componentes discretos, de modo que a mudança em um deles tenha impacto mínimo sobre os demais, e a **modificabilidade**, grau em que o produto pode ser modificado sem introduzir defeitos ou degradar a qualidade existente. A mesma norma define, no âmbito da portabilidade, a **substituibilidade** (*replaceability*): o grau em que um produto pode substituir outro, com o mesmo propósito, no mesmo ambiente.

No título e nos objetivos deste trabalho, o termo portabilidade é empregado em sentido operacional: a capacidade de substituir ou acrescentar o provedor de IA com impacto mínimo sobre o restante da aplicação. Em termos da ISO/IEC 25010, esse sentido está mais próximo da modularidade do que da portabilidade entre ambientes, e é a modularidade, observada por meio do alcance das mudanças, que as métricas da subseção 3.4 aproximam. Ambos os atributos dependem do grau de dependência que um sistema estabelece com componentes e serviços externos, o que remete aos conceitos de acoplamento e coesão.

### 2.2 Acoplamento e Coesão

O acoplamento refere-se ao grau de interdependência entre os módulos de um sistema. O baixo acoplamento é desejável porque reduz o risco de que uma alteração em um módulo produza efeitos não previstos em outros (CHIDAMBER; KEMERER, 1994). A coesão, por sua vez, refere-se ao grau em que os elementos internos de um módulo estão relacionados em torno de uma responsabilidade única e bem definida, sendo desejável que seja alta. Uma classe que reúne regras de negócio e detalhes de integração com um serviço externo tem, por essa definição, coesão reduzida, pois acumula responsabilidades de naturezas distintas.

Chidamber e Kemerer (1994) propuseram uma suíte de métricas para a avaliação de projetos orientados a objetos, entre as quais a métrica CBO (*Coupling Between Objects*), que quantifica o número de classes às quais uma classe está acoplada. Este trabalho não calcula o CBO: observa o acoplamento de forma indireta, pelo alcance das alterações exigidas por cada mudança e pelas referências ao provedor fora do código dedicado a ele (subseção 3.4). A expectativa que orienta essa escolha é a de que, quanto mais acoplada ao provedor estiver a aplicação, mais as mudanças originadas no provedor se propaguem para outras classes.

### 2.3 Padrões de Projeto, o Padrão Adapter e a Inversão de Dependência

Os padrões de projeto (*design patterns*) constituem soluções reutilizáveis para problemas recorrentes de projeto de software, catalogadas e sistematizadas por Gamma et al. (1994) em obra que se tornou referência fundamental da área. Entre os padrões estruturais catalogados pelos autores, o **Adapter** tem como propósito converter a interface de uma classe em outra interface esperada pelos clientes, permitindo que classes com interfaces incompatíveis colaborem sem que seja necessário modificar seu código-fonte original (GAMMA et al., 1994).

A estrutura clássica do padrão Adapter é composta por quatro elementos: (i) o **Target**, que define a interface específica utilizada pelo código cliente; (ii) o **Adaptee**, que representa a classe ou componente existente cuja interface é incompatível com a esperada pelo cliente; (iii) o **Adapter**, componente concreto que implementa a interface Target e internamente traduz as chamadas recebidas para o formato compreendido pelo Adaptee; e (iv) o **Client**, que consome exclusivamente a interface Target, sem conhecimento direto da implementação do Adaptee.

Aplicado ao contexto deste trabalho, o Adaptee corresponde à API específica de um provedor de Inteligência Artificial Generativa (por exemplo, a API do Gemini), com seu formato particular de requisição e resposta; o Target corresponde a uma interface genérica definida pela própria aplicação, com um método que solicita a geração de um resumo; e o Adapter corresponde à implementação concreta que traduz as chamadas dessa interface para o formato exigido pela API do provedor em uso. Dessa forma, a lógica de negócio da aplicação, representada pelo Client, permanece isolada das particularidades de cada provedor, dependendo exclusivamente da interface Target.

Essa estrutura também aplica o princípio da inversão de dependência, formulado por Martin (1996): módulos de alto nível não devem depender de módulos de baixo nível, e ambos devem depender de abstrações; as abstrações, por sua vez, não devem depender de detalhes. No protótipo, a classe de serviço (alto nível) depende da interface definida pela aplicação, e cada adaptador (baixo nível) a implementa; a implementação concreta é fornecida à classe de serviço pelo mecanismo de injeção de dependência do Spring, por meio do construtor.

O padrão Adapter não é o único mecanismo presente nessa estrutura. Quando a aplicação passa a escolher entre provedores em tempo de execução, como no Cenário 2 deste trabalho, as implementações da interface funcionam como alternativas intercambiáveis selecionadas pelo cliente, papel que Gamma et al. (1994) descrevem no padrão *Strategy*. Por isso, os efeitos observados neste trabalho são atribuídos à camada de abstração como um todo — interface definida pela aplicação, adaptadores por provedor e inversão de dependência —, e não ao padrão Adapter isoladamente.

### 2.4 Evidências Empíricas e Trabalhos Relacionados

Embora a literatura reconheça, em nível teórico, os benefícios do uso de padrões de projeto para a redução do acoplamento e o aumento da manutenibilidade, não há consenso sobre esses benefícios quando mensurados empiricamente (ALI; ELISH, 2013), o que reforça a importância de estudos que produzam evidências objetivas sobre esse impacto.

Nesse sentido, destaca-se o estudo de Al-Obeidallah et al. (2021), que investigou especificamente o impacto do padrão Adapter sobre a manutenibilidade de software. Os autores aplicaram técnicas de refatoração para gerar, a partir de quatro sistemas distintos, versões equivalentes com e sem o padrão Adapter, e em seguida calcularam e compararam métricas de software entre as versões, apoiando-se na relação entre essas métricas e a manutenibilidade apontada por estudos anteriores. Os resultados indicaram que as versões com o padrão apresentaram métricas de software melhores do que as versões sem ele, o que os autores interpretam como impacto positivo do Adapter sobre a manutenibilidade.

Este trabalho adota desse estudo a lógica de comparar versões equivalentes de um mesmo sistema com e sem o padrão, mas difere dele em dois aspectos. Primeiro, as métricas de Al-Obeidallah et al. (2021) descrevem a estrutura de cada versão em um dado momento, enquanto este trabalho mede o custo de mudanças concretas aplicadas a cada versão. Segundo, o objeto de estudo é um protótipo construído para investigar a integração com provedores de IA Generativa, e não um conjunto de sistemas genéricos preexistentes.

A literatura, contudo, não é unânime quanto ao sentido nem à magnitude desses efeitos. Ali e Elish (2013), em levantamento da literatura empírica sobre os padrões GoF, concluíram que o impacto dos padrões havia sido investigado em apenas quatro atributos de qualidade e que não havia consenso sobre esse impacto. Qamar e Malik (2020), ao comparar pares de programas escritos sem e com padrões de projeto para os 23 padrões GoF, verificaram que a complexidade ciclomática foi menor, na maioria dos casos, nos programas com padrões, mas que as métricas CK, o número de classes e o tamanho em linhas de código aumentaram. Esses resultados indicam que a redução de um tipo de impacto pode vir acompanhada de aumento de estrutura e de volume de código, de modo que a análise não deve se limitar a um único indicador.

O Quadro 1 sintetiza os trabalhos relacionados e a posição deste estudo em relação a eles.

**Quadro 1 — Trabalhos relacionados e posição deste estudo**

| Trabalho | Objeto | Método | Resultado principal | Relação com este trabalho |
|---|---|---|---|---|
| Al-Obeidallah et al. (2021) | Padrão Adapter em quatro sistemas | Comparação de métricas de software entre versões com e sem o padrão, obtidas por refatoração | Versões com Adapter apresentaram métricas melhores | Origem do desenho comparativo; este trabalho mede o custo de mudanças, e não métricas estruturais |
| Qamar e Malik (2020) | Os 23 padrões GoF | Comparação de pares de programas sem e com padrões | Menor complexidade ciclomática na maioria dos casos; aumento das métricas CK, do número de classes e de linhas de código | Antecipa o custo estrutural observado na versão desacoplada |
| Ali e Elish (2013) | Evidências empíricas sobre padrões GoF | Levantamento da literatura | Impacto estudado em quatro atributos de qualidade, sem consenso | Justifica a medição de mais de um indicador |
| Bucaioni et al. (2025) | Sistemas integrados a LLMs | Arquitetura de referência funcional preliminar, avaliada frente a preocupações arquiteturais e aplicada a três sistemas de código aberto | Aponta privacidade, segurança, modularidade e interoperabilidade como preocupações críticas | Contextualiza o problema; não mede o impacto de decisões arquiteturais |
| Este trabalho | Integração de aplicação web com provedores de IA Generativa | Estudo de caso comparativo com duas versões de um protótipo e três cenários de mudança | Seção 4 | — |

Fonte: elaborado pelo autor.

### 2.5 Integração de Aplicações Web com Provedores de Inteligência Artificial Generativa

A disponibilização de modelos de Inteligência Artificial Generativa por meio de APIs comerciais tem favorecido sua incorporação em aplicações web. Diferentemente de uma biblioteca incorporada ao código, esses serviços evoluem por decisão do provedor: no caso da API do Gemini, a documentação oficial publica o calendário de desligamento dos modelos e indica o modelo recomendado para substituir cada um deles (GOOGLE, [2026]). Os formatos de comunicação também variam entre provedores. Na implementação deste trabalho, a API do Gemini recebeu o nome do modelo no caminho da URL e a chave de acesso em um cabeçalho próprio, ao passo que a API do Groq, compatível com o formato da OpenAI, recebeu o modelo no corpo da requisição e a chave como *token* de portador, com estruturas de requisição e de resposta distintas das do Gemini.

Do ponto de vista acadêmico, a discussão sobre arquitetura de software para sistemas integrados a modelos de linguagem de grande escala ainda é incipiente. Bucaioni et al. (2025) propuseram uma arquitetura de referência funcional preliminar para esses sistemas, apontando privacidade, segurança, modularidade e interoperabilidade como preocupações críticas à medida que eles crescem em complexidade e impacto social. A proposta foi avaliada frente a essas preocupações e aplicada à análise de três sistemas de código aberto; os autores não mediram o impacto de decisões arquiteturais específicas, como a adoção de camadas de abstração, sobre atributos de qualidade, lacuna para a qual o presente trabalho busca contribuir, em escopo delimitado e a partir de um único caso.

### 2.6 Síntese e Lacuna de Pesquisa

A revisão da literatura evidencia que o efeito do padrão Adapter sobre a manutenibilidade já foi investigado empiricamente em contextos gerais da Engenharia de Software, com resultados favoráveis em Al-Obeidallah et al. (2021), porém sem consenso na literatura sobre padrões de projeto (ALI; ELISH, 2013; QAMAR; MALIK, 2020), e que a comunidade de Arquitetura de Software reconhece a relevância de preocupações como a modularidade em sistemas integrados a modelos de Inteligência Artificial Generativa (BUCAIONI et al., 2025). Não foram identificados, contudo, estudos empíricos que mensurem o impacto da adoção do padrão Adapter sobre a manutenibilidade e a portabilidade de aplicações web diante de mudanças relacionadas a provedores de Inteligência Artificial Generativa. É essa lacuna que o presente trabalho investiga, aplicando um desenho comparativo já empregado na literatura de padrões de projeto a um contexto de aplicação emergente, com métricas centradas no custo das mudanças.

## 3 Metodologia

### 3.1 Classificação da Pesquisa

Do ponto de vista da natureza, esta pesquisa classifica-se como aplicada, uma vez que se propõe a gerar conhecimento com aplicação prática sobre um problema real de Engenharia de Software, sem a pretensão de desenvolver teoria original. Quanto à abordagem, é predominantemente quantitativa na coleta e comparação de métricas de software, complementada por análise qualitativa na discussão dos resultados à luz da literatura. Quanto ao procedimento técnico, adota-se o **estudo de caso comparativo**, seguindo as diretrizes de Runeson e Höst (2009) para a condução e o relato de estudos de caso em Engenharia de Software, sendo o caso constituído por um único protótipo de aplicação web, implementado em duas versões arquiteturais distintas.

Optou-se deliberadamente por não classificar este trabalho como um experimento controlado no sentido estatístico, uma vez que o estudo envolve uma única unidade de análise, sem grupo de controle independente, sem randomização e sem repetição amostral que permita inferência estatística. Trata-se de um estudo de caso comparativo de caráter exploratório, cujos resultados devem ser interpretados como evidência contextualizada.

Runeson e Höst (2009) caracterizam o estudo de caso como a investigação de um fenômeno contemporâneo em seu contexto real. Neste trabalho, o caso é um protótipo construído para a pesquisa e avaliado em ambiente controlado, o que o aproxima de um estudo de caso único conduzido em laboratório. Essa característica favorece o controle das variáveis, pois as duas versões diferem apenas na forma de integração com o provedor, e enfraquece a validade externa, como discutido na subseção 3.6.

### 3.2 Descrição do Protótipo e das Versões Arquiteturais

O objeto de estudo é um protótipo de assistente de resumo de textos acadêmicos, desenvolvido especificamente para esta pesquisa: o usuário envia um texto e recebe um resumo gerado por um provedor de Inteligência Artificial Generativa. O escopo funcional foi mantido reduzido de forma deliberada, pois o objeto de investigação não é a aplicação em si, mas o impacto arquitetural das decisões de projeto adotadas para sua integração com o provedor.

O protótipo foi desenvolvido em Java 17 com o *framework* Spring Boot (módulo Spring Web) e o gerenciador de *build* Maven. A interface de usuário é uma página estática em HTML, CSS e JavaScript, servida pela própria aplicação, que envia o texto ao *endpoint* `POST /api/resumo`. A classe de serviço aplica as regras de negócio — texto não vazio, com no mínimo 50 e no máximo 5.000 caracteres —, acrescenta ao texto do usuário a instrução enviada ao modelo e devolve o resumo. Falhas de validação são respondidas com o status HTTP 400 e falhas do provedor (indisponibilidade, tempo esgotado ou limite de requisições excedido) com o status 502, ambas com uma mensagem padronizada. As chamadas ao provedor usam o cliente HTTP `RestTemplate`, com tempos limite de 10 segundos para conexão e de 15 segundos para leitura. A escolha da plataforma não interfere na comparação, pois é comum às duas versões; o mecanismo de injeção de dependência por construtor do Spring permitiu implementar a inversão de dependência da Versão B sem código adicional de montagem de objetos.

O provedor integrado na linha de base das duas versões é a API do Gemini (Google); nos cenários de mudança, foi utilizada também a API do Groq (subseção 3.3). O protótipo foi implementado em duas versões funcionalmente equivalentes, porém arquiteturalmente distintas:

- **Versão A — Arquitetura Acoplada:** a comunicação com a API do provedor é realizada diretamente pela classe de serviço que contém as regras de negócio (`ResumoService`), sem camada de abstração intermediária. As particularidades do formato de requisição e resposta do provedor ficam, portanto, expostas a essa classe.

- **Versão B — Arquitetura Desacoplada:** a comunicação com o provedor é mediada por uma interface definida pela aplicação (`AiSummarizerClient`, o Target) e por uma implementação concreta do padrão Adapter por provedor (`GeminiAdapter`), responsável por traduzir as chamadas dessa interface para o formato exigido pela API do provedor (o Adaptee), conforme a estrutura apresentada na subseção 2.3. A classe de serviço (o Client) depende exclusivamente da interface.

O Quadro 2 detalha onde cada responsabilidade reside em cada versão, na linha de base.

**Quadro 2 — Distribuição das responsabilidades nas linhas de base das duas versões**

| Responsabilidade | Versão A (acoplada) | Versão B (desacoplada) |
|---|---|---|
| Validação do texto e instrução enviada ao modelo | `ResumoService` | `ResumoService` |
| Montagem da requisição, chamada HTTP, mapeamento de erros e extração da resposta | `ResumoService` | `GeminiAdapter` (pacote `provider/gemini`) |
| Classes de requisição e resposta do provedor | Pacote `client`, visíveis a toda a aplicação | Pacote `provider/gemini`, visíveis apenas dentro do pacote |
| Configuração do provedor (URL, chave, modelo e tempos limite) | Pacote `config` | Pacote `provider/gemini` |
| Dependência da classe de serviço | Classes do Gemini e cliente HTTP | Interface `AiSummarizerClient`, recebida por injeção no construtor |
| Exceção de falha do provedor | Pacote `exception` | Pacote `provider` |
| Conversão de exceções em respostas HTTP (400 e 502) | `GlobalExceptionHandler` | `GlobalExceptionHandler` |

Fonte: elaborado pelo autor.

Ambas as versões compartilham a mesma lógica de negócio e a mesma interface de usuário, diferenciando-se exclusivamente na forma de comunicação com o provedor de IA, condição necessária para que as diferenças de impacto observadas possam ser atribuídas à decisão arquitetural investigada, e não a diferenças funcionais.

### 3.3 Protocolo de Cenários de Mudança

Em conformidade com a recomendação de definir o protocolo de coleta de dados antes de sua execução — o que mitiga o risco de viés do pesquisador, discutido na subseção 3.6 —, foram definidos três cenários controlados de mudança, aplicados de forma idêntica às duas versões do protótipo:

1. **Cenário 1 — Substituição de provedor:** substituir integralmente o provedor de IA utilizado, mantendo a funcionalidade inalterada do ponto de vista do usuário.

2. **Cenário 2 — Inclusão de um novo provedor:** incorporar um segundo provedor, mantendo o original em funcionamento e permitindo que a aplicação alterne entre ambos.

3. **Cenário 3 — Alteração na forma de comunicação com o provedor:** adaptar a aplicação a uma mudança no formato de requisição e de resposta da API do provedor em uso, do tipo que um provedor pode introduzir em uma nova versão de sua API.

O provedor utilizado nos Cenários 1 e 2 foi o Groq, escolhido por oferecer acesso gratuito à API no período da pesquisa e por adotar um formato de comunicação compatível com o da OpenAI, distinto do formato do Gemini (subseção 2.5). Essa diferença de formato torna a troca de provedor um teste exigente para as duas versões. No Cenário 1, todo o código específico do Gemini foi removido e substituído pelo equivalente do Groq, sem arquivos ou nomes remanescentes. No Cenário 2, a requisição passou a aceitar um campo opcional `provedor` (`gemini` ou `groq`), normalizado antes da comparação; na ausência do campo, utiliza-se o Gemini, e valores fora do conjunto aceito são rejeitados com o status 400. No Cenário 3, simulou-se uma nova versão do contrato do Gemini: a requisição passou a exigir um campo `generationConfig`, e a resposta ganhou um nível adicional (`response`), com o texto fragmentado em várias partes a serem concatenadas. Como a API real não adota esse formato, o Cenário 3 foi validado exclusivamente por testes automatizados com respostas simuladas, ao passo que os Cenários 1 e 2 foram validados por chamadas às APIs reais.

Cada cenário é implementado separadamente em cada versão, sempre a partir da linha de base daquela versão, de modo que as alterações de um cenário não se acumulam com as de outro.

### 3.4 Métricas de Avaliação

Para cada cenário aplicado a cada versão do protótipo, são coletadas as seguintes métricas objetivas, com base no controle de versão do código-fonte (Git):

- **Número de arquivos modificados:** quantidade de arquivos alterados para a implementação do cenário, obtida por meio do comando `git diff --numstat --no-renames` entre a linha de base e o estado final do cenário.
- **Número de linhas de código alteradas:** soma das linhas adicionadas e removidas para a implementação do cenário, obtida pelo mesmo comando.
- **Testes automatizados afetados:** arquivos e linhas de teste criados, removidos ou modificados no cenário.
- **Referências textuais ao provedor fora do código do provedor:** contagem de ocorrências, sem distinção entre maiúsculas e minúsculas, dos nomes dos provedores em arquivos `.java` do código-fonte principal que não pertencem ao grupo P (definido abaixo). Diferentemente das demais métricas, ela caracteriza o grau de acoplamento de um estado do sistema (as duas linhas de base e os seis estados resultantes dos cenários), e não o custo de uma mudança. Por ser uma contagem textual, soma importações, nomes de tipos, literais e comentários, e deve ser lida como indicador, não como contagem de dependências.

Optou-se por não adotar métricas subjetivas de esforço, como a percepção de dificuldade, por estarem sujeitas ao viés de quem implementa e mede simultaneamente, decisão relacionada às ameaças à validade discutidas adiante.

As métricas de arquivos e linhas são obtidas com a opção `--no-renames`, que desabilita a detecção heurística de renomeações do Git (limiar padrão de 50% de similaridade) e contabiliza integralmente cada arquivo removido e cada arquivo criado. A opção torna a contagem determinística e reprodutível, pois, com a detecção ativa, o enquadramento de um mesmo par de arquivos como "renomeado" ou como "removido e criado" depende de um limiar de similaridade, e não do conteúdo da mudança. O critério é aplicado de forma idêntica às duas versões; os valores com a detecção de renomeações ativa são reportados como análise de sensibilidade.

Além dos totais por cenário, que constituem a métrica primária, os arquivos alterados são classificados em três grupos, com a mesma regra nas duas versões: (P) código dedicado ao provedor, isto é, arquivos cujo nome começa com o nome de um provedor (`Gemini*`, `Groq*`) ou que residem em pacote de provedor; (T) testes automatizados, regra aplicada antes das demais; e (O) demais arquivos do protótipo, subdivididos em código e configuração. A classificação é feita por arquivo, e não por linha: em arquivos mistos, como o `ResumoService` da Versão A, que reúne regras de negócio e lógica de integração, todas as linhas alteradas são contabilizadas no grupo O. O grupo O aproxima a modularidade definida na subseção 2.1: quanto menos uma mudança no provedor exige alterações fora do código dedicado a ele, menor é seu impacto sobre os demais componentes. Essa decomposição responde ao enunciado da hipótese, que se refere ao impacto sobre "a lógica de negócio e os demais componentes da aplicação". Declara-se que o agrupamento foi formalizado após a observação dos totais do Cenário 1 em ambas as versões, e que sua regra foi fixada por escrito antes da medição dos Cenários 2 e 3 da Versão B. Por isso, os totais e a decomposição são reportados em conjunto, inclusive quando divergem.

### 3.5 Procedimento de Execução

A coleta de dados seguiu a sequência: (i) implementação da Versão A até um estado funcional estável, correspondente à sua linha de base; (ii) aplicação dos três cenários de mudança sobre a Versão A, com coleta das métricas de cada cenário e restauração da linha de base entre um cenário e outro; (iii) implementação da Versão B, com escopo funcional equivalente ao da Versão A; (iv) aplicação dos mesmos três cenários sobre a Versão B, com coleta das mesmas métricas; e (v) consolidação e comparação dos resultados das duas versões, cenário a cenário, apresentadas na seção 4. Cada etapa de implementação partiu de uma especificação escrita definida antes da codificação.

As duas versões partem de linhas de base fixadas por etiquetas (*tags*) no repositório do protótipo, e cada cenário resulta em uma etiqueta própria, de modo que todas as medições podem ser reproduzidas por meio do comando `git diff --numstat --no-renames <linha de base> <etiqueta>`. O código e as etiquetas estão disponíveis em repositório público (https://github.com/CmuriloDev/TCC-Prototype). O procedimento de medição foi automatizado em um *script* e reexecutado sobre os seis pares de estados ao final da coleta, com resultados idênticos aos registrados durante a execução.

Para que as diferenças observadas decorram da arquitetura, e não de diferenças funcionais entre as versões, foram adotadas decisões de equivalência: (i) a Versão A foi mantida organizada, com métodos privados por provedor, sem abstrações disfarçadas, e a duplicação específica de cada provedor foi preservada; (ii) o tratamento de erros é o mesmo nas duas versões, com duas categorias (limite de requisições excedido e demais falhas) e as mesmas mensagens, o que foi verificado nas duas linhas de base; (iii) a instrução enviada ao modelo reside no `ResumoService` nas duas versões; (iv) no Cenário 1, o provedor substituído foi integralmente removido; e (v) no Cenário 3, a mesma suíte de cinco casos de teste foi escrita nas duas versões, no mesmo pacote da classe testada. Duas diferenças de projeto são declaradas. Na Versão B, o provedor padrão do Cenário 2 é definido em arquivo de configuração e o comentário do campo `provedor` é genérico, de modo que nenhum nome de provedor aparece fora do pacote dos provedores; na Versão A, o padrão é um valor literal no código e o comentário cita os provedores. Essa diferença acrescenta linhas de configuração à Versão B (11 contra 8) e, portanto, não favorece a hipótese. Além disso, os arquivos do Groq incluídos no Cenário 2 da Versão B foram copiados do Cenário 1 da mesma versão, por se tratar do mesmo contrato de integração.

### 3.6 Ameaças à Validade

Em conformidade com o modelo de discussão de ameaças à validade proposto por Runeson e Höst (2009), reconhecem-se as seguintes limitações:

- **Validade de construto:** as métricas adotadas constituem indicadores indiretos do esforço de manutenção, e não uma medida direta dele. O número de linhas depende do critério de contagem, razão pela qual se reporta a análise de sensibilidade, e do estilo de implementação. A contagem de referências ao provedor é textual e inclui comentários.
- **Validade interna:** o autor foi responsável pela especificação e condução da implementação das duas versões e pela coleta das métricas, o que configura risco de viés do pesquisador-implementador. A ameaça é mitigada pela definição prévia do protocolo de cenários, pelas decisões de equivalência descritas na subseção 3.5, pela automação e reexecução da medição e pela declaração de que o agrupamento P, T e O foi formalizado após a observação do Cenário 1. O valor nulo de referências ao provedor fora do grupo P na Versão B decorre, em parte, de uma restrição imposta ao projeto e, por isso, não constitui evidência independente.
- **Validade externa:** trata-se de uma única unidade de análise, de pequeno porte, construída para a pesquisa, com uma única funcionalidade e dois provedores. Os resultados não podem ser generalizados para outros sistemas, domínios ou equipes, constituindo evidência contextualizada. O Cenário 3 é uma simulação, e não uma mudança real de contrato.
- **Confiabilidade:** o protocolo, as etiquetas, o *script* de medição e o repositório público permitem que outros pesquisadores reproduzam as medições. Uma nova implementação das versões, contudo, poderia produzir números diferentes, em função de decisões de implementação específicas.

## 4 Resultados e Discussão

Esta seção apresenta os resultados da aplicação dos três cenários de mudança às duas versões do protótipo e os discute à luz da literatura. As métricas foram obtidas conforme a subseção 3.4, entre a linha de base de cada versão e a etiqueta de cada cenário, e verificadas pela reexecução do procedimento automatizado sobre os seis pares de estados. As subseções 4.1 a 4.3 relatam o que foi observado; a subseção 4.4 interpreta essas observações. O detalhamento por arquivo consta do Apêndice A.

### 4.1 Totais por Cenário e Versão

A Tabela 1 apresenta o total de arquivos e de linhas tocadas em cada cenário, por versão.

**Tabela 1 — Totais por cenário e versão (`--no-renames`)**

| Cenário | Versão | Arquivos alterados | Linhas adicionadas | Linhas removidas | Linhas tocadas |
|---|---|---|---|---|---|
| 1 — Substituição de provedor | A (acoplada) | 11 | 235 | 248 | 483 |
| 1 — Substituição de provedor | B (desacoplada) | 11 | 311 | 324 | 635 |
| 2 — Inclusão de novo provedor | A (acoplada) | 9 | 364 | 24 | 388 |
| 2 — Inclusão de novo provedor | B (desacoplada) | 11 | 375 | 5 | 380 |
| 3 — Alteração de formato de comunicação | A (acoplada) | 4 | 226 | 9 | 235 |
| 3 — Alteração de formato de comunicação | B (desacoplada) | 4 | 225 | 9 | 234 |

Fonte: elaborado pelo autor.

Com a detecção de renomeações do Git ativa (análise de sensibilidade), apenas o Cenário 1 se altera: 9 arquivos e 333 linhas na Versão A, e 8 arquivos e 347 linhas na Versão B. Somados os três cenários, a Versão A totaliza 1.106 linhas tocadas e a Versão B, 1.249 no critério `--no-renames`; no critério padrão, os totais são 956 e 961, respectivamente. A diferença entre os critérios concentra-se no Cenário 1, em que o critério `--no-renames` contabiliza integralmente o pacote do provedor removido e o pacote do provedor criado.

### 4.2 Distribuição das Alterações por Grupo

A Tabela 2 separa as alterações segundo o grupo do arquivo, no formato arquivos / linhas tocadas.

**Tabela 2 — Distribuição por grupo, por cenário e versão (arquivos / linhas tocadas)**

| Cenário | Versão | P (provedor) | T (testes) | O — código | O — configuração |
|---|---|---|---|---|---|
| 1 | A | 8 / 426 | 0 / 0 | 2 / 47 | 1 / 10 |
| 1 | B | 10 / 625 | 0 / 0 | 0 / 0 | 1 / 10 |
| 2 | A | 4 / 207 | 0 / 0 | 4 / 173 | 1 / 8 |
| 2 | B | 6 / 308 | 0 / 0 | 4 / 61 | 1 / 11 |
| 3 | A | 2 / 63 | 1 / 147 | 1 / 25 | 0 / 0 |
| 3 | B | 3 / 89 | 1 / 145 | 0 / 0 | 0 / 0 |

Fonte: elaborado pelo autor.

**Cenário 1 — substituição de provedor.** Na Versão A, a troca alterou dois arquivos de código fora do grupo P: o `ResumoService` (45 linhas tocadas) e a `ProvedorIndisponivelException` (2 linhas). Na Versão B, nenhum arquivo de código fora do grupo P foi alterado: `ResumoService`, `ResumoController`, `GlobalExceptionHandler`, a interface `AiSummarizerClient`, a exceção e os DTOs tiveram zero linhas alteradas. O único arquivo fora de P alterado nas duas versões foi o `application.properties` (10 linhas). O grupo P da Versão B foi maior (625 contra 426 linhas): nela, a lógica de chamada, de mapeamento de erros e de extração da resposta está em uma classe própria (`GeminiAdapter`, 96 linhas), que foi removida e substituída por outra (`GroqAdapter`, 91 linhas); na Versão A, essa lógica está no `ResumoService`, contabilizado em O.

**Cenário 2 — inclusão de um segundo provedor.** As duas versões alteraram os mesmos quatro arquivos de código fora do grupo P. Três deles tiveram o mesmo número de linhas tocadas nas duas versões (`ResumoRequest`, 11; `GlobalExceptionHandler`, 6; `ProvedorInvalidoException`, 12), num total de 29 linhas, correspondentes à escolha do provedor por requisição. A diferença está no `ResumoService`: 144 linhas tocadas (120 adicionadas e 24 removidas) na Versão A, que passou a conter a lógica de chamada dos dois provedores, contra 32 (28 adicionadas e 4 removidas) na Versão B, que apenas seleciona a implementação pelo nome. O código fora de P somou 173 linhas na Versão A e 61 na Versão B, uma redução de cerca de 65%, enquanto o total do cenário foi praticamente igual (388 e 380). Na Versão B, a interface `AiSummarizerClient` não foi alterada, e o `GeminiAdapter` teve uma linha trocada, para receber o nome pelo qual é selecionado.

**Cenário 3 — alteração do formato de comunicação.** O total foi praticamente igual (235 e 234 linhas), assim como o código de produção alterado (88 e 89 linhas). Na Versão A, a lógica de extração da resposta está no `ResumoService`, e o cenário alterou 25 linhas desse arquivo (grupo O); na Versão B, ela está no `GeminiAdapter` (grupo P), e nenhum arquivo fora do pacote do provedor e do teste foi alterado. Nas duas versões foi escrita a mesma suíte de cinco casos de teste (147 e 145 linhas, 63% e 62% do total do cenário). Como o formato simulado não é o da API real, o código resultante, nas duas versões, não gera resumos a partir da API real, e a validação foi feita exclusivamente pelos testes, conforme a subseção 3.3.

### 4.3 Referências Textuais ao Provedor

A Tabela 3 apresenta, para cada estado, o número de arquivos fora do grupo P que citam o nome de algum provedor e o total de ocorrências.

**Tabela 3 — Referências textuais ao provedor fora do grupo P (arquivos / ocorrências)**

| Estado | Versão A (acoplada) | Versão B (desacoplada) |
|---|---|---|
| Linha de base | 2 / 26 | 0 / 0 |
| Após o Cenário 1 | 2 / 25 | 0 / 0 |
| Após o Cenário 2 | 3 / 78 | 0 / 0 |
| Após o Cenário 3 | 2 / 29 | 0 / 0 |

Fonte: elaborado pelo autor.

Na Versão A, quase todas as ocorrências estão no `ResumoService` (25, 24, 74 e 28, nos quatro estados), e o total passa de 26 para 78 quando o segundo provedor é incorporado. Nesse arquivo, as primeiras ocorrências estão na região de importações, em que são importados os tipos específicos de cada provedor. Na Versão B, o valor é zero em todos os estados. Há, adicionalmente, uma evidência estrutural: na Versão B, as classes de requisição e de resposta de cada provedor são visíveis apenas dentro do próprio pacote, de modo que o compilador impede que outras camadas as utilizem.

### 4.4 Discussão

Os resultados sustentam a hipótese de forma parcial. Quanto ao código fora do pacote do provedor, a Versão B apresentou menor impacto nos três cenários: 0 contra 47 linhas no Cenário 1, 61 contra 173 no Cenário 2 e 0 contra 25 no Cenário 3, o que, somado, corresponde a 61 contra 245 linhas, uma redução de cerca de 75%. Quanto ao volume total das alterações, a hipótese não se confirma: a Versão B somou cerca de 13% mais linhas no critério `--no-renames` e praticamente o mesmo (0,5% a mais) no critério padrão. Como o desenho é de caso único (subseção 3.1), a hipótese foi avaliada de forma descritiva, e esses percentuais não sustentam inferência estatística.

Em outras palavras, o desacoplamento não eliminou o trabalho de escrever o código de integração com um novo provedor; alterou o lugar em que esse trabalho se concentra. Esse resultado decorre da própria estrutura descrita na subseção 2.3: como a classe de serviço depende apenas da interface, uma mudança originada no provedor só alcança o adaptador correspondente, ao passo que, na Versão A, a mesma mudança alcança a classe que contém as regras de negócio. A Tabela 3 mostra o mesmo fenômeno sob outro ângulo: na Versão A, o número de referências ao provedor no `ResumoService` triplica quando um segundo provedor é incluído, o que indica que, nessa arquitetura, a classe de negócio acumula responsabilidades de integração a cada novo provedor. Pela definição de coesão adotada na subseção 2.2, essa acumulação reduz a coesão da classe, efeito que a Versão B evita ao mantê-la restrita às regras de negócio. Em termos da ISO/IEC 25010, o comportamento da Versão B corresponde a maior modularidade em relação às mudanças de provedor: a alteração em um componente teve impacto mínimo sobre os demais.

O volume total maior ou equivalente na Versão B tem duas origens identificáveis. A primeira é o critério de contagem: no Cenário 1, o critério `--no-renames` conta por inteiro o pacote removido e o pacote criado, ainda que o novo adaptador seja estruturalmente semelhante ao anterior; com a detecção de renomeações ativa, a diferença entre as versões cai de 152 para 14 linhas. A segunda é estrutural: na Versão B, cada provedor tem uma classe a mais (o adaptador, além das classes de requisição, resposta, configuração e propriedades que a Versão A também possui), e partes do código de integração, como o mapeamento de erros HTTP, repetem-se em cada adaptador, enquanto na Versão A o mesmo método é compartilhado pelos provedores dentro do `ResumoService`. A independência entre adaptadores tem, portanto, o custo de alguma duplicação.

Esse padrão é coerente com a literatura. Chidamber e Kemerer (1994) tratam o acoplamento entre classes como indicador de interdependência, e a Versão A exibe o efeito esperado: uma mudança originada no provedor atinge o `ResumoService`, a exceção e, no Cenário 2, o DTO de entrada e o tratamento de exceções, enquanto na Versão B a mesma mudança permanece contida no pacote do provedor, exceto pela parte que corresponde à nova funcionalidade. Em Al-Obeidallah et al. (2021), as versões com o padrão Adapter apresentaram métricas de software melhores; os resultados deste trabalho convergem com esse estudo quanto à direção do efeito sobre a propagação das mudanças, mas o sistema, as métricas e o desenho são distintos, o que impede a comparação de magnitudes. O aumento do número de classes e do volume de código na versão com o padrão corresponde ao que Qamar e Malik (2020) observaram para os padrões GoF em geral. A dependência do resultado em relação ao indicador escolhido — favorável na propagação, desfavorável ou neutra no volume — ajuda a explicar a ausência de consenso apontada por Ali e Elish (2013): estudos que medem atributos diferentes podem chegar a conclusões diferentes sobre o mesmo padrão.

Dois resultados merecem ressalva. No Cenário 2, o grupo O não foi nulo em nenhuma das versões, porque a escolha do provedor por requisição é uma funcionalidade nova da camada web e exige alterar o DTO de entrada, o tratamento de exceções e a classe de serviço em qualquer arquitetura; o desacoplamento reduziu essa alteração, mas não a eliminou. No Cenário 3, a suíte de testes teve tamanho semelhante nas duas versões, de modo que o deslocamento do custo de validação para o código de teste, observado quando a mudança de contrato é simulada e não pode ser verificada contra a API real, decorre da natureza do cenário, e não de um efeito diferencial da arquitetura. Por fim, como discutido na subseção 2.3, a Versão B combina o padrão Adapter, a inversão de dependência e, no Cenário 2, a seleção de implementações à maneira do padrão Strategy, razão pela qual o efeito observado é atribuído a essa camada de abstração como um todo.

Como implicação prática, restrita ao caso estudado, os resultados indicam que o desacoplamento compensa quando se espera trocar ou acrescentar provedores, ou quando a estabilidade das classes de negócio é prioritária, e que ele tem um custo estrutural que pode não se justificar em uma aplicação que dependa de um único provedor sem expectativa de mudança. As ameaças à validade da subseção 3.6 aplicam-se integralmente a essas conclusões. Em particular, a classificação é feita por arquivo e não por linha; o agrupamento em P, T e O foi formalizado após a observação do Cenário 1; a contagem de referências ao provedor é textual; e o número de linhas é uma aproximação do esforço de manutenção, e não uma medida direta dele.

## 5 Considerações Finais

Este trabalho investigou em que medida a adoção de uma arquitetura desacoplada, baseada no padrão Adapter, reduz o impacto de mudanças relacionadas a provedores de Inteligência Artificial Generativa em uma aplicação web. No caso estudado, a resposta depende do que se entende por impacto. A arquitetura desacoplada não reduziu o volume total das alterações exigidas pelas mudanças: somados os três cenários, ele foi maior ou equivalente ao da versão acoplada, conforme o critério de contagem. Ela reduziu, porém, as alterações fora do código dedicado ao provedor, que foram nulas nos cenários de substituição e de alteração de formato e menores no cenário de inclusão de um segundo provedor. A hipótese de que o desacoplamento reduz o impacto das mudanças sobre a lógica de negócio e os demais componentes da aplicação foi, portanto, sustentada para essa dimensão, de forma descritiva, e não se confirmou quanto ao esforço total.

Os objetivos específicos foram atendidos da seguinte forma. A fundamentação sobre acoplamento, coesão, inversão de dependência e o padrão Adapter (objetivo i) foi apresentada na seção 2 e utilizada na interpretação dos resultados. O protótipo foi implementado em duas versões funcionalmente equivalentes (objetivo ii), e os três cenários de mudança foram aplicados às duas (objetivo iii), conforme a seção 3. As métricas foram coletadas e comparadas (objetivo iv) e discutidas à luz da literatura (objetivo v) na seção 4.

A contribuição do trabalho tem duas partes. A primeira é uma evidência empírica, em pequena escala, de que, na integração com provedores de IA Generativa, o benefício de uma camada de abstração está na localização das mudanças, e não na redução de seu volume, e de que esse benefício tem um custo estrutural mensurável. A segunda é um procedimento reprodutível de medição, baseado em etiquetas do Git e em um *script* público, que separa o código dedicado ao provedor, os testes e os demais componentes, e que pode ser aplicado a outros sistemas.

As conclusões estão sujeitas às limitações discutidas na subseção 3.6: um único protótipo de pequeno porte, construído para a pesquisa; dois provedores e uma única funcionalidade; um cenário de alteração de formato simulado; métricas que aproximam, mas não medem diretamente, o esforço de manutenção; e um agrupamento dos arquivos formalizado após a observação do primeiro cenário.

Como trabalhos futuros, propõe-se replicar o procedimento em sistemas maiores e com mais provedores; utilizar mudanças reais de contrato, como as descontinuações de modelos registradas pelos provedores, em vez de mudanças simuladas; medir o esforço de desenvolvedores distintos do pesquisador, por exemplo em tempo de execução das tarefas; complementar as métricas de alteração com métricas estruturais, como o CBO; e comparar a camada de abstração própria com o uso de bibliotecas ou *gateways* de terceiros que oferecem uma interface comum para vários provedores.

## Referências

ALI, M.; ELISH, M. O. A comparative literature survey of design patterns impact on software quality. In: INTERNATIONAL CONFERENCE ON INFORMATION SCIENCE AND APPLICATIONS (ICISA), 4., 2013, Pattaya. **Anais [...]**. 2013.

AL-OBEIDALLAH, M. G.; AL-FRAIHAT, D. G.; KHASAWNEH, A. M.; SALEH, A. M.; ADDOUS, H. Empirical Investigation of the Impact of the Adapter Design Pattern on Software Maintainability. In: INTERNATIONAL CONFERENCE ON INFORMATION TECHNOLOGY (ICIT), 2021. **Anais [...]**. IEEE, 2021. p. 206-211. DOI: 10.1109/ICIT52682.2021.9491719.

BASS, L.; CLEMENTS, P.; KAZMAN, R. **Software Architecture in Practice**. 3. ed. Boston: Addison-Wesley, 2012.

BUCAIONI, A.; WEYSSOW, M.; HE, J.; LYU, Y.; LO, D. A Functional Software Reference Architecture for LLM-Integrated Systems. In: IEEE INTERNATIONAL CONFERENCE ON SOFTWARE ARCHITECTURE COMPANION (ICSA-C), 22., 2025, Odense. **Anais [...]**. IEEE, 2025. p. 1-5. DOI: 10.1109/ICSA-C65153.2025.00006.

CHIDAMBER, S. R.; KEMERER, C. F. A Metrics Suite for Object-Oriented Design. **IEEE Transactions on Software Engineering**, v. 20, n. 6, p. 476-493, 1994.

GAMMA, E.; HELM, R.; JOHNSON, R.; VLISSIDES, J. **Design Patterns: Elements of Reusable Object-Oriented Software**. Reading: Addison-Wesley, 1994.

GOOGLE. **Gemini deprecations**. [S. l.]: Google, [2026]. Disponível em: https://ai.google.dev/gemini-api/docs/deprecations. Acesso em: 6 out. 2026.

INTERNATIONAL ORGANIZATION FOR STANDARDIZATION. **ISO/IEC 25010:2011**: Systems and software engineering — Systems and software Quality Requirements and Evaluation (SQuaRE) — System and software quality models. Geneva: ISO, 2011.

MARTIN, R. C. The Dependency Inversion Principle. **C++ Report**, 1996.

QAMAR, N.; MALIK, A. A. Impact of Design Patterns on Software Complexity and Size. **Mehran University Research Journal of Engineering and Technology**, v. 39, n. 2, p. 342-352, 2020. DOI: 10.22581/muet1982.2002.10.

RUNESON, P.; HÖST, M. Guidelines for Conducting and Reporting Case Study Research in Software Engineering. **Empirical Software Engineering**, v. 14, n. 2, p. 131-164, 2009.

## Apêndice A — Detalhamento por arquivo

As tabelas a seguir correspondem às saídas de `git diff --numstat --no-renames` entre a linha de base de cada versão e a etiqueta do cenário (Versão A: `295d660`; Versão B: `497ed71`), restritas ao diretório `prototype/`.

**Tabela A1 — Versão A, Cenário 1 (substituição de provedor)**

| Arquivo | Tipo de alteração | Linhas adicionadas | Linhas removidas | Grupo |
|---|---|---|---|---|
| `GeminiRequest.java` | Removido | 0 | 63 | P |
| `GeminiResponse.java` | Removido | 0 | 65 | P |
| `GroqRequest.java` | Criado | 68 | 0 | P |
| `GroqResponse.java` | Criado | 52 | 0 | P |
| `GeminiConfig.java` | Removido | 0 | 41 | P |
| `GeminiProperties.java` | Removido | 0 | 48 | P |
| `GroqConfig.java` | Criado | 41 | 0 | P |
| `GroqProperties.java` | Criado | 48 | 0 | P |
| `ProvedorIndisponivelException.java` | Modificado | 1 | 1 | O (código) |
| `ResumoService.java` | Modificado | 20 | 25 | O (código) |
| `application.properties` | Modificado | 5 | 5 | O (configuração) |

Fonte: elaborado pelo autor.

**Tabela A2 — Versão A, Cenário 2 (inclusão de novo provedor)**

| Arquivo | Tipo de alteração | Linhas adicionadas | Linhas removidas | Grupo |
|---|---|---|---|---|
| `GroqRequest.java` | Criado | 67 | 0 | P |
| `GroqResponse.java` | Criado | 51 | 0 | P |
| `GroqConfig.java` | Criado | 41 | 0 | P |
| `GroqProperties.java` | Criado | 48 | 0 | P |
| `ResumoRequest.java` | Modificado | 11 | 0 | O (código) |
| `GlobalExceptionHandler.java` | Modificado | 6 | 0 | O (código) |
| `ProvedorInvalidoException.java` | Criado | 12 | 0 | O (código) |
| `ResumoService.java` | Modificado | 120 | 24 | O (código) |
| `application.properties` | Modificado | 8 | 0 | O (configuração) |

Fonte: elaborado pelo autor.

**Tabela A3 — Versão A, Cenário 3 (alteração de formato de comunicação)**

| Arquivo | Tipo de alteração | Linhas adicionadas | Linhas removidas | Grupo |
|---|---|---|---|---|
| `GeminiRequest.java` | Modificado | 39 | 0 | P |
| `GeminiResponse.java` | Modificado | 19 | 5 | P |
| `ResumoService.java` | Modificado | 21 | 4 | O (código) |
| `ResumoServiceExtracaoGeminiTest.java` | Criado | 147 | 0 | T |

Fonte: elaborado pelo autor.

**Tabela A4 — Versão B, Cenário 1 (substituição de provedor)**

| Arquivo | Tipo de alteração | Linhas adicionadas | Linhas removidas | Grupo |
|---|---|---|---|---|
| `provider/gemini/GeminiAdapter.java` | Removido | 0 | 96 | P |
| `provider/gemini/GeminiConfig.java` | Removido | 0 | 41 | P |
| `provider/gemini/GeminiProperties.java` | Removido | 0 | 48 | P |
| `provider/gemini/GeminiRequest.java` | Removido | 0 | 66 | P |
| `provider/gemini/GeminiResponse.java` | Removido | 0 | 68 | P |
| `provider/groq/GroqAdapter.java` | Criado | 91 | 0 | P |
| `provider/groq/GroqConfig.java` | Criado | 41 | 0 | P |
| `provider/groq/GroqProperties.java` | Criado | 48 | 0 | P |
| `provider/groq/GroqRequest.java` | Criado | 71 | 0 | P |
| `provider/groq/GroqResponse.java` | Criado | 55 | 0 | P |
| `application.properties` | Modificado | 5 | 5 | O (configuração) |

Fonte: elaborado pelo autor.

**Tabela A5 — Versão B, Cenário 2 (inclusão de novo provedor)**

| Arquivo | Tipo de alteração | Linhas adicionadas | Linhas removidas | Grupo |
|---|---|---|---|---|
| `dto/ResumoRequest.java` | Modificado | 11 | 0 | O (código) |
| `exception/GlobalExceptionHandler.java` | Modificado | 6 | 0 | O (código) |
| `exception/ProvedorInvalidoException.java` | Criado | 12 | 0 | O (código) |
| `provider/gemini/GeminiAdapter.java` | Modificado | 1 | 1 | P |
| `provider/groq/GroqAdapter.java` | Criado | 91 | 0 | P |
| `provider/groq/GroqConfig.java` | Criado | 41 | 0 | P |
| `provider/groq/GroqProperties.java` | Criado | 48 | 0 | P |
| `provider/groq/GroqRequest.java` | Criado | 71 | 0 | P |
| `provider/groq/GroqResponse.java` | Criado | 55 | 0 | P |
| `service/ResumoService.java` | Modificado | 28 | 4 | O (código) |
| `application.properties` | Modificado | 11 | 0 | O (configuração) |

Fonte: elaborado pelo autor.

**Tabela A6 — Versão B, Cenário 3 (alteração de formato de comunicação)**

| Arquivo | Tipo de alteração | Linhas adicionadas | Linhas removidas | Grupo |
|---|---|---|---|---|
| `provider/gemini/GeminiAdapter.java` | Modificado | 22 | 4 | P |
| `provider/gemini/GeminiRequest.java` | Modificado | 39 | 0 | P |
| `provider/gemini/GeminiResponse.java` | Modificado | 19 | 5 | P |
| `GeminiAdapterExtracaoTest.java` | Criado | 145 | 0 | T |

Fonte: elaborado pelo autor.
