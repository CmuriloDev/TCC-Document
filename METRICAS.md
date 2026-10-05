# Registro de Métricas — Estudo de Caso Comparativo (versão corrigida)

> Substitui o METRICAS.md anterior, que tinha dois erros de transcrição (ver final do arquivo).

## Critério de medição (vale para TODAS as medições)

- Comando: `git diff --numstat --no-renames <baseline> <tag> -- prototype/`
- Linhas tocadas = adicionadas + removidas.
- Valores com detecção de renomeação do Git (padrão) ficam como análise de sensibilidade.
- Grupos (regra por arquivo, igual nas duas versões):
  - **P** = código dedicado ao provedor (nome `Gemini*`/`Groq*` ou pacote `provider/gemini|groq`)
  - **T** = testes
  - **O** = demais arquivos (separar código x configuração)
- Excluído da medição: `METRICAS.md` (commit 0be01e7, entre a baseline e o Cenário 3 da acoplada).
- Script de medição do Codador: `scratchpad/medir.sh` (somente leitura).

## Baselines e tags

| Tag | Commit | Observação |
|---|---|---|
| baseline-acoplada | 295d660 | Sprint 1 (só Gemini) |
| baseline-desacoplada | 497ed71 | Sprint 3.3 (Target + GeminiAdapter) |
| cenario-1-acoplada | bc0cf1c | Gemini -> Groq |
| cenario-2-acoplada | cf372e9 | Groq adicionado |
| cenario-3-acoplada | a648c39 | Formato simulado + teste |
| cenario-1-desacoplada | 86dc06d | GeminiAdapter -> GroqAdapter |
| cenario-2-desacoplada | d784cc8 | GroqAdapter adicionado; Service escolhe o Adapter por nome (Map) |
| cenario-3-desacoplada | ea76236 | GeminiAdapter e DTOs ajustados ao formato simulado + teste |

---

## Versão Acoplada

### Cenário 1 — Substituição (295d660 -> bc0cf1c)

| Arquivo | Status | + | − | Grupo |
|---|---|---|---|---|
| client/GeminiRequest | D | 0 | 63 | P |
| client/GeminiResponse | D | 0 | 65 | P |
| client/GroqRequest | A | 68 | 0 | P |
| client/GroqResponse | A | 52 | 0 | P |
| config/GeminiConfig | D | 0 | 41 | P |
| config/GeminiProperties | D | 0 | 48 | P |
| config/GroqConfig | A | 41 | 0 | P |
| config/GroqProperties | A | 48 | 0 | P |
| exception/ProvedorIndisponivelException | M | 1 | 1 | O (código) |
| service/ResumoService | M | 20 | 25 | O (código) |
| application.properties | M | 5 | 5 | O (config) |

Totais `--no-renames`: **11 arquivos, +235/−248 = 483** (P 8/426, O 3/57; O código 2/47, O config 1/10).
Sensibilidade (padrão): **9 arquivos, 333 linhas** (160/173; P 6/276, O 3/57).

### Cenário 2 — Inclusão de provedor (295d660 -> cf372e9)

| Arquivo | Status | + | − | Grupo |
|---|---|---|---|---|
| client/GroqRequest | A | 67 | 0 | P |
| client/GroqResponse | A | 51 | 0 | P |
| config/GroqConfig | A | 41 | 0 | P |
| config/GroqProperties | A | 48 | 0 | P |
| dto/ResumoRequest | M | 11 | 0 | O (código) |
| exception/GlobalExceptionHandler | M | 6 | 0 | O (código) |
| exception/ProvedorInvalidoException | A | 12 | 0 | O (código) |
| service/ResumoService | M | **120** | 24 | O (código) |
| application.properties | M | 8 | 0 | O (config) |

Totais (os dois critérios coincidem): **9 arquivos, +364/−24 = 388** (P 4/207, O 5/181; O código 4/173, O config 1/8).

### Cenário 3 — Alteração de formato, simulada (295d660 -> a648c39)

| Arquivo | Status | + | − | Grupo |
|---|---|---|---|---|
| client/GeminiRequest | M | 39 | 0 | P |
| client/GeminiResponse | M | 19 | 5 | P |
| service/ResumoService | M | 21 | 4 | O (código) |
| test/.../ResumoServiceExtracaoGeminiTest | A | 147 | 0 | T |

Totais (os dois critérios coincidem): **4 arquivos, +226/−9 = 235** (P 2/63, T 1/147, O 1/25).
Produção sem teste: 88 linhas (79 + 9). O teste (5 casos) = 63% do total.
Validação: teste automatizado com JSON simulado (a API real não devolve esse formato).

---

## Versão Desacoplada

### Cenário 1 — Substituição (497ed71 -> 86dc06d)

| Arquivo | Status | + | − | Grupo |
|---|---|---|---|---|
| provider/gemini/GeminiAdapter | D | 0 | 96 | P |
| provider/gemini/GeminiConfig | D | 0 | 41 | P |
| provider/gemini/GeminiProperties | D | 0 | 48 | P |
| provider/gemini/GeminiRequest | D | 0 | 66 | P |
| provider/gemini/GeminiResponse | D | 0 | 68 | P |
| provider/groq/GroqAdapter | A | 91 | 0 | P |
| provider/groq/GroqConfig | A | 41 | 0 | P |
| provider/groq/GroqProperties | A | 48 | 0 | P |
| provider/groq/GroqRequest | A | 71 | 0 | P |
| provider/groq/GroqResponse | A | 55 | 0 | P |
| application.properties | M | 5 | 5 | O (config) |

Totais `--no-renames`: **11 arquivos, +311/−324 = 635** (P 10/625, O 1/10; O código 0/0).
Sensibilidade (padrão): **8 arquivos, 347 linhas** (P 7/337, O 1/10).
Intocados (0 linhas): ResumoService, ResumoController, GlobalExceptionHandler, AiSummarizerClient, ProvedorIndisponivelException, DTOs, index.html.

### Cenário 2 — Inclusão de provedor (497ed71 -> d784cc8)

| Grupo | Arquivos | Linhas tocadas |
|---|---|---|
| P | 6 | 308 (provider/groq ×5 = 306, copiados da tag cenario-1-desacoplada; GeminiAdapter = 1/1, só o `@Component("gemini")`) |
| T | 0 | 0 |
| O código | 4 | 61 |
| O configuração | 1 | 11 |
| **Total** | **11** | **380** |

O código por arquivo (linhas tocadas): `ResumoService` 32 (+28/−4), `ProvedorInvalidoException` 12 (novo), `ResumoRequest` 11, `GlobalExceptionHandler` 6. Configuração: `application.properties` 11.
Os dois critérios coincidem (sem renomeações). `AiSummarizerClient`: 0 linhas. `git grep -i "gemini|groq"` em `service/`, `controller/`, `dto/` e `exception/`: vazio.
Saída bruta (`git diff --numstat --no-renames 497ed71 cenario-2-desacoplada`), +/− por arquivo:

| Arquivo | + | − | Grupo |
|---|---|---|---|
| dto/ResumoRequest | 11 | 0 | O (código) |
| exception/GlobalExceptionHandler | 6 | 0 | O (código) |
| exception/ProvedorInvalidoException | 12 | 0 | O (código) |
| provider/gemini/GeminiAdapter | 1 | 1 | P |
| provider/groq/GroqAdapter | 91 | 0 | P |
| provider/groq/GroqConfig | 41 | 0 | P |
| provider/groq/GroqProperties | 48 | 0 | P |
| provider/groq/GroqRequest | 71 | 0 | P |
| provider/groq/GroqResponse | 55 | 0 | P |
| service/ResumoService | 28 | 4 | O (código) |
| application.properties | 11 | 0 | O (config) |

Soma: +375/−5 = 380.

Validação por curl: sem provedor 200; `"  GROQ "` 200; `"gemini"` 200; `"openai"` 400 com a mensagem "Provedor inválido. Valores aceitos: gemini, groq."; texto curto 400; chave inválida (groq e gemini) 502. Um caso extra (provedor em branco) deu 502 uma vez e 200 em seguida, atribuído à instabilidade do Gemini (inferência do Codador, não verificada).

**Comparação com a acoplada (Cenário 2, `--no-renames`)**

| | Acoplada | Desacoplada |
|---|---|---|
| Total (arquivos / linhas) | 9 / 388 | 11 / 380 |
| P | 4 / 207 | 6 / 308 |
| O código | 4 / 173 | 4 / 61 |
| `ResumoService` | 144 (+120/−24) | 32 (+28/−4) |
| O configuração | 1 / 8 | 1 / 11 |

Leitura: `ResumoRequest` (11), `GlobalExceptionHandler` (6) e `ProvedorInvalidoException` (12) têm o mesmo tamanho nas duas versões (29 linhas: o custo da funcionalidade em si). Toda a diferença do grupo O está no `ResumoService` (144 → 32).

**Expectativas registradas antes de medir: todas confirmadas.** `AiSummarizerClient` 0 linhas; O não zerou (mesmos 4 arquivos de código da acoplada); O menor na B; sem expectativa de total menor (380 contra 388, diferença desprezível); testes 0 nas duas.

### Cenário 3 — Alteração de formato, simulada (497ed71 -> ea76236)

| Grupo | Arquivos | Linhas tocadas |
|---|---|---|
| P | 3 | 89 (GeminiAdapter +22/−4, GeminiRequest +39/−0, GeminiResponse +19/−5) |
| T | 1 | 145 (GeminiAdapterExtracaoTest, 5 casos, mesmo pacote do Adapter) |
| O (código e configuração) | 0 | 0 |
| **Total** | **4** | **234** (+225/−9) |

Os dois critérios coincidem. Nenhum arquivo fora de `provider/` e fora do teste foi tocado. `AiSummarizerClient`: 0 linhas. Testes: `mvnw test` com 5 novos + `contextLoads`, 0 falhas. Não validado contra a API real (formato simulado).

**Comparação com a acoplada (Cenário 3)**

| | Acoplada | Desacoplada |
|---|---|---|
| Total (arquivos / linhas) | 4 / 235 | 4 / 234 |
| P | 2 / 63 | 3 / 89 |
| O | 1 / 25 (`ResumoService`) | 0 / 0 |
| T | 1 / 147 | 1 / 145 |
| Produção (P + O) | 88 | 89 |

Leitura: a lógica de extração mudou o mesmo tanto (88 contra 89 linhas de produção); o que muda é onde ela mora (no `ResumoService` na A, no `GeminiAdapter` na B). Total praticamente igual. Expectativas registradas antes de medir: todas confirmadas.

---

## Totais lado a lado (arquivos / linhas)

| Cenário | Versão | P | T | O | Geral (`--no-renames`) | Geral (padrão) |
|---|---|---|---|---|---|---|
| 1 | Acoplada | 8 / 426 | 0 | 3 / 57 | 11 / 483 | 9 / 333 |
| 1 | Desacoplada | 10 / 625 | 0 | 1 / 10 | 11 / 635 | 8 / 347 |
| 2 | Acoplada | 4 / 207 | 0 | 5 / 181 | 9 / 388 | 9 / 388 |
| 2 | Desacoplada | 6 / 308 | 0 | 5 / 72 (código 4 / 61 + config 1 / 11) | 11 / 380 | 11 / 380 |
| 3 | Acoplada | 2 / 63 | 1 / 147 | 1 / 25 | 4 / 235 | 4 / 235 |
| 3 | Desacoplada | 3 / 89 | 1 / 145 | 0 | 4 / 234 | 4 / 234 |

## Resumo agregado (descritivo; soma dos três cenários, sem ponderar)

| | Acoplada | Desacoplada |
|---|---|---|
| Total de linhas, `--no-renames` | 1106 (483 + 388 + 235) | 1249 (635 + 380 + 234) |
| Total de linhas, critério padrão | 956 (333 + 388 + 235) | 961 (347 + 380 + 234) |
| Linhas em O, só código | 245 (47 + 173 + 25) | 61 (0 + 61 + 0) |

Reportar os dois totais: no critério `--no-renames` a B soma mais (o Cenário 1 conta o pacote antigo apagado e o novo criado por inteiro); no critério padrão os totais são praticamente iguais. O efeito consistente é o grupo O: o código fora do pacote do provedor.

## Status de verificação

- **Verificado por reexecução.** Os seis pares foram remedidos com o `medir.sh` final (`--no-renames` e critério padrão) e coincidem em todos os valores com as tabelas deste arquivo. Classificação usada: T = qualquer arquivo em `src/test/` (regra aplicada primeiro); P = caminho em `provider/gemini|groq/` ou nome `Gemini*`/`Groq*`; O configuração = `src/main/resources/*`; O código = o restante. Só o `METRICAS.md` do Cenário 3 da acoplada (status A) está fora de `prototype/`, e não entra em nenhum total.
- **Baselines equivalentes no tratamento de erro.** `295d660:ResumoService` e `497ed71:GeminiAdapter` têm 2 categorias em `mapearErroHttp`, as mesmas mensagens e o mesmo número de pontos de lançamento de `ProvedorIndisponivelException` (9 ocorrências em cada). Diferem o pacote da exceção (`exception/` na A, `provider/` na B) e o `throws` na assinatura da interface (B).
- Ressalva: quem manteve este arquivo não executa Git; a verificação vem da reexecução feita pelo Codador.
- **Pendente:** copiar `medir.sh` e `medicao-final.txt` do scratchpad (pasta temporária da sessão) para o repositório do TCC, por exemplo `metricas/`.
- **Pendente:** conferir `git rev-parse versao-acoplada` (deve ser `295d660` ao final) e levar o remoto de `versao-desacoplada` de volta a `497ed71` (hoje em `6a436b5`), só depois de conferir os hashes e com `--force-with-lease`. A checagem anterior da ponta da `versao-acoplada` mostrou código anterior ao tratamento de erro e nunca foi explicada; as medições não são afetadas, porque partem do hash `295d660`.

## Decisões de equivalência entre as versões (guardar para a Metodologia e a defesa)

1. Acoplada bem organizada (métodos privados por provedor), para evitar comparação com "espantalho".
2. Duplicação mantida na acoplada (`extrairTextoGerado` por provedor), sem abstração disfarçada.
3. `mapearErroHttp` com 2 categorias (429 específico, resto genérico) nas duas versões.
4. Handler 502 de `ProvedorIndisponivelException` presente nas duas.
5. `PROMPT_BASE` no `ResumoService` nas duas.
6. Cenário 1: renomear/apagar tudo do provedor antigo (substituição limpa).
7. Cenário 3: mesmo teste (mesmos 5 casos) nas duas.
8. Critério `--no-renames` nas duas; padrão como sensibilidade.
9. Agrupamento P/O/T formalizado após ver o Cenário 1; regra fixada antes dos Cenários 2 e 3 da B.
10. Cenário 2 da B: o provedor padrão vem de `application.properties` (`ai.provider.default`) e o comentário do campo `provedor` no DTO é genérico; na A o padrão é literal no código e o comentário cita os provedores. Motivo: nenhum nome de provedor fora de `provider/` na B. Custo: +3 linhas de configuração na B.
11. Cenário 2 da B: os arquivos de `provider/groq/` foram copiados da tag `cenario-1-desacoplada` (mesmo contrato validado do Groq).
12. Cenário 3: o teste da B fica em `provider/gemini` (mesmo pacote do Adapter) e cobre os mesmos 5 casos da A; a extração passou de private a package-private no Adapter (na A, no Service).
13. Script de medição: `grupo()` passou a testar `src/test` antes de `provider/gemini`; não altera nenhuma medição anterior.

## Correções em relação ao registro anterior

- Cenário 2 acoplada: `ResumoService` é +120/−24 (o registro anterior dizia +144/−24). O total 388 já estava certo.
- Cenário 1 acoplada: a Tabela de arquivos somava 332 por causa de `ProvedorIndisponivelException` (+1/−1, não 1 linha).
- Cenário 3 acoplada: produção = 88 linhas (79 + 9), não "79"; o total 235 é o menor entre os três cenários.
- Cenário 1 acoplada: 483 linhas em `--no-renames` (333 era o critério padrão, que misturava renomeado e removido/criado).
