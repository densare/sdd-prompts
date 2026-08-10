# Anti-Patterns de Codigo AI - Licoes Aprendidas

> **LEITURA OBRIGATORIA** antes de /plan, /implement e /check.
> Baseado na analise real do DenStudio (2025), onde codigo gerado por AI acumulou problemas sistematicos.

---

## Principio Fundamental

> **A complexidade do codigo deve ser proporcional a complexidade do problema.**
> Se o problema e simples, a solucao DEVE ser simples.

---

## AP-01: Interfaces 1:1 (Interface sem Polimorfismo)

### Problema

Criar uma interface para CADA classe/struct, mesmo quando so existe UMA implementacao.

### Exemplo Real (DenStudio)

```
28 interfaces, TODAS com exactamente 1 implementacao:
- ISettingsService -> SettingsService
- IThemeService -> ThemeService
- IMunicipalityService -> MunicipalityService
- INavigationService -> NavigationService
... (24 mais)
```

**Resultado**: 56 ficheiros em vez de 28. Zero beneficio. Duplicacao de assinaturas.

### Porque Acontece com AI

O agente segue "boas praticas" genericas (Dependency Injection, SOLID) sem avaliar se o contexto justifica. Em enterprise Java/C# e comum — mas num projecto novo sem necessidade de polimorfismo, e ruido.

### Regra de Prevencao

```
ANTES de criar uma interface, responder:
1. "Quantas implementacoes CONCRETAS existem ou estao PLANEADAS?"
2. Se a resposta e "1" -> NAO criar interface, usar struct/class directa
3. Excepcoes aceites:
   - Boundary de teste (ex: repository que acede a DB -> interface para mock em testes)
   - Boundary de sistema (ex: API externa -> interface para trocar provider)
4. Documentar a justificacao no plan.md
```

---

## AP-02: Seguranca Desproporcional (Sledgehammer)

### Problema

Aplicar medidas de seguranca enterprise a problemas triviais. Encriptacao militar para dados nao-sensiveis. Rate limiting para operacoes internas.

### Exemplo Real (DenStudio)

```
Para guardar POSICOES DE JANELA (top, left, width, height):
- AES-256-GCM encryption
- HMAC-SHA256 signing
- Key derivation com salt
- Rate limiting (max 10 saves/min)
- Authorization com [Authorize] attribute
- Audit logging de cada operacao

SecureViewStateService: 786 LOC
vs
O necessario (BasicViewStateService): 282 LOC
```

**Resultado**: 500+ LOC de seguranca para proteger dados sem valor. "Usar um canhao para matar uma mosca."

### Porque Acontece com AI

O agente trata TODOS os dados como igualmente sensiveis. Aplica seguranca "best practice" sem avaliar o threat model. Encripta porque "encriptar e seguro", sem perguntar "seguro contra que ameaca?"

### Regra de Prevencao

```
ANTES de adicionar seguranca, responder:
1. "Que dados estamos a proteger?"
2. "Qual e a ameaca concreta?" (quem atacaria, como, porque)
3. "Qual e o impacto se estes dados forem comprometidos?"

Classificacao:
- Dados CRITICOS (passwords, tokens, dados pessoais) -> Encriptacao, auth, audit
- Dados INTERNOS (configuracao, estado de UI) -> Permissoes basicas, sem encriptacao
- Dados PUBLICOS (temas, preferencias visuais) -> Nada

NUNCA aplicar seguranca "porque sim". Documentar threat model no plan.md.
```

---

## AP-03: Security Theater (Atributos sem Enforcement)

### Problema

Adicionar atributos/decorators de seguranca que nao sao aplicados pelo runtime. Codigo que PARECE seguro mas nao faz nada.

### Exemplo Real (DenStudio)

```csharp
// Atributos de seguranca em TODOS os services:
[Authorize(Role = "Admin")]
[AuditLog("UpdateSettings")]
public class SettingsService

// Realidade: ZERO middleware que verifica [Authorize]
// Realidade: ZERO handler que processa [AuditLog]
// Resultado: Falsa sensacao de seguranca
```

```csharp
// InputSanitizer que "protege contra SQL injection":
public static string Sanitize(string input)
{
    // Detecta patterns SQL com regex...
    // MAS todas as queries usam parametros ($1, $2)
    // E o sanitizer corrompe dados legitimos:
    // "Rua 1/2" -> "Rua 1&#47;2"  (HTML-encode de /)
}
```

**Resultado**: Seguranca ilusoria. Pior: dados corrompidos pelo sanitizer desnecessario.

### Porque Acontece com AI

O agente adiciona camadas de "proteccao" sem verificar se o runtime as aplica. Copia patterns de frameworks enterprise sem confirmar que a infraestrutura de enforcement existe.

### Regra de Prevencao

```
Regra: CADA medida de seguranca deve ter enforcement verificavel.

ANTES de adicionar seguranca, verificar:
1. "Existe middleware/handler que processa este atributo?"
2. "Consigo escrever um teste que falha se a seguranca for removida?"
3. Se NAO -> NAO adicionar. Seguranca sem enforcement e PIOR que nada.

NAO criar sanitizers redundantes:
- Se SQL usa parametros -> NAO precisa de sanitizer SQL
- Se output usa encoding automatico -> NAO precisa de sanitizer XSS
- Se input e validado com tipos -> NAO precisa de sanitizer generico
```

---

## AP-04: Duplicacao Sistematica

### Problema

Copiar codigo entre modulos/packages em vez de extrair para codigo partilhado. Reimplementar utilidades que ja existem.

### Exemplo Real (DenStudio)

```
Duplicacoes encontradas:
- RelayCommand: 4 implementacoes identicas em 4 modulos
- INotifyPropertyChanged: 6 implementacoes separadas
- SetProperty helper: 3 versoes
- Municipios/Freguesias/Regioes: mesma logica de lookup em 3 modules separados
- DTOs duplicados: EntityDTO e EntityModel com campos identicos
```

**Resultado**: Bug fix num sitio nao propaga para os outros. Inconsistencias entre versoes.

### Porque Acontece com AI

O agente trabalha modulo a modulo. Ao implementar o modulo B, nao verifica que o modulo A ja tem o mesmo codigo. Cria codigo "fresco" em vez de procurar existente.

### Regra de Prevencao

```
Regra: SEARCH BEFORE CREATE

ANTES de criar qualquer funcao/tipo/package:
1. Procurar no projecto: "existe algo que faz isto?"
2. Procurar em pkg/ (Go), lib/ (SvelteKit), templates/ (Templ), ou namespaces partilhados (C#)
3. Se existe -> REUTILIZAR
4. Se e similar mas nao exacto -> GENERALIZAR o existente
5. Se nao existe e sera usado em 2+ sitios -> Criar em pkg/ ou lib/

NUNCA copiar codigo entre packages/modulos. Extrair para partilhado.
```

---

## AP-05: God Objects (Classes/Packages Gigantes)

### Problema

Classes ou packages que fazem demasiadas coisas. Ficheiros com centenas ou milhares de linhas.

### Exemplo Real (DenStudio)

```
- DenStudioEntity (base class): 700+ LOC
  Responsabilidades: validacao, tracking, serialization, equality, cloning, metadata

- NavigationPanel: 1028 LOC
  Responsabilidades: navigation, tree building, search, filtering, expand/collapse, icons

- ModuleSettings: 686 LOC
  Responsabilidades: settings, validation, persistence, defaults, migration
```

**Resultado**: Impossivel testar isoladamente. Qualquer alteracao pode partir outra coisa.

### Porque Acontece com AI

O agente adiciona funcionalidade incrementalmente ao mesmo ficheiro/classe. "Ja que estou aqui, adiciono tambem..." Nao refactoriza proactivamente para manter ficheiros pequenos.

### Regra de Prevencao

```
Limites CONCRETOS (sistema de zonas):

FICHEIROS (C#, Go, qualquer linguagem):
- 🟢 VERDE (< 500 LOC): Livre para desenvolver
- 🟡 AMARELA (500-600 LOC): Congelado - só bug fixes, sem novas funções
- 🔴 VERMELHA (> 600 LOC): Obrigatório refatorar antes de merge

MÉTODOS/FUNÇÕES (C#, Go, qualquer linguagem):
- 🟢 VERDE (< 45 LOC): Livre para desenvolver
- 🟡 AMARELA (45-55 LOC): Congelado - só bug fixes, sem nova lógica
- 🔴 VERMELHA (> 55 LOC): Obrigatório refatorar antes de merge

TEMPLATES (Templ, SvelteKit, Avalonia AXAML):
- Template/Componente > 200 LOC -> Extrair sub-componentes/templates

PACKAGES/NAMESPACES (funcoes exportadas / metodos publicos):
- 🟢 VERDE (< 30): Livre para desenvolver
- 🟡 AMARELA (30-45): Considerar split se continuar a crescer
- 🔴 VERMELHA (> 45): Split obrigatorio antes de merge

Nota: Na zona amarela, se precisares de adicionar funcionalidade,
primeiro extrai código para baixar para zona verde.
```
---

## AP-06: Codigo na Camada Errada

### Problema

Logica de negocio em camadas de infraestrutura. Dependencias de UI em camadas de dominio. Mistura de responsabilidades.

### Exemplo Real (DenStudio)

```
No Domain layer (que deveria ser puro):
- Sentry SDK (monitoring/infraestrutura)
- MessageBox (UI)
- File I/O (infraestrutura)
- HTTP calls (infraestrutura)

Resultado: O "dominio" depende de tudo. Impossivel testar sem mock de UI, network, filesystem.
```

### Porque Acontece com AI

O agente coloca codigo onde e mais "conveniente" no momento, ignorando boundaries entre camadas. Segue a path of least resistance.

### Regra de Prevencao

```
Go:
  handler.go  -> SO: HTTP (parse request, call service, write response)
  service.go  -> SO: logica de negocio (decisoes, validacao de regras, orquestracao)
  repository.go -> SO: dados (SQL, Redis)
  model.go    -> SO: tipos e validacao de dominio

SvelteKit / Templ + HTMX:
  +page.svelte   -> SO: UI (SvelteKit) / templates/ -> SO: UI (Templ)
  stores/        -> SO: estado (SvelteKit) / handler.go -> SO: HTTP (Templ)
  lib/api/       -> SO: chamadas HTTP (SvelteKit) / service.go -> SO: logica (Templ)
  lib/utils/     -> SO: funcoes puras (SvelteKit) / static/ -> SO: CSS, JS (Templ)

C#:
  DensareClient  -> SO: API publica (fachada)
  Internal/      -> SO: implementacao (auth, license, http)
  Models/        -> SO: DTOs

TESTE: Se tirar o handler/UI, o service continua a funcionar? Se nao -> violacao.
```

---

## AP-07: Dead Code e Features Fantasma

### Problema

Codigo que foi criado mas nunca ligado. Dialogs stubbed. Features parcialmente implementadas. Interfaces sem implementacao.

### Exemplo Real (DenStudio)

```
- BackupDialog: stub vazio, nunca chamado
- ExportService: metodos que retornam NotImplementedException
- 4 interfaces definidas mas nunca implementadas
- Event handlers registados para eventos que nunca sao emitidos
```

**Resultado**: Codigo que ocupa espaco, confunde, e cria falsa expectativa de funcionalidade.

### Porque Acontece com AI

O agente planeia features completas mas implementa parcialmente. Cria "esqueletos" para completar depois, mas o "depois" nunca chega. Stubs ficam para sempre.

### Regra de Prevencao

```
Regra: NAO criar stubs, scaffolds ou "para implementar depois"

1. Se a feature nao vai ser implementada AGORA -> NAO criar o ficheiro
2. Se a feature vai ser implementada parcialmente -> spec deve dizer exactamente o que
3. Nao criar interfaces "para o futuro"
4. Nao criar metodos que lancam NotImplementedException
5. Nao registar event handlers para eventos que nao existem

Tudo o que e criado deve funcionar. Zero dead code.
```

---

## AP-09: Infrastructure-only sem Entry-Point

### Problema

Componentes existem no codigo (services, ViewModels, dialogs, handlers, endpoints) mas **nenhum caminho do utilizador chega la**: nao ha menu, navigation node, keybinding, painel que invoque a feature; ou o handler existe mas o route nunca foi montado no router. A feature parece "implementada" mas e invisivel/inacessivel em runtime.

### Exemplo Real (DenTherm 2026-05)

Cadeia de 6+ issues descobertas em smoke E2E **depois** do merge porque a feature ficou inacessivel:

```
- DT-549: PaineisStandalonePontesTermicas — painel implementado, zero entrada de menu/nav
- DT-550: Edit button SolutionLibraryPanel — handler implementado, botao nunca renderizado
- DT-554: Entry-points Caderno + Medidas — TabDefinition existe, zero nav node
- DT-555: ViewKeys orfaos — 4 views registadas, zero rotas
- DT-565: SolarProtectionEditorDialog — evento SolutionEditorRequested disparado mas
          zero subscriber em producao (so em testes)
- DT-591: Pattern repetido — feature G2 fase 1 (15 pts em DT-402..406) invisivel ao
          utilizador porque nunca foi adicionada seccao "Relatorios" em Navigation.cs
```

**Exemplo Real (Cloud — variante backend)**: PLT-281 — TOTP handler + service implementados mas nunca instanciados em main.go; 6 rotas devolviam 404 em runtime apesar de existirem no codigo.

**Resultado**: feature aparece como "Done" no Linear, mas user testa e ve nada. Vira issue nova (eg `entry-point-X-feature-Y`), as vezes meses depois. Custo: cada uma destas requer nova spec + plan + impl + check + end-issue.

### Porque Acontece com AI

O agente foca-se em "implementar a logica/UI da feature" e termina quando os ficheiros estao escritos e os testes passam. Nao verifica que **existe um caminho navegavel do estado inicial da app ate esta feature**. Os unit tests passam porque invocam o componente directamente; nada testa "user abre app -> chega aqui".

### Regra de Prevencao

```
Regra: TODA spec que adiciona uma feature observavel pelo utilizador DEVE
declarar explicitamente os entry-points UI/runtime.

DURANTE /sdd-specify:
1. Listar na seccao "Entry-Points" da spec.md TODOS os caminhos que o user/cliente
   usa para chegar a feature. Exemplos por stack:
   - Avalonia/UI: menu (qual?), nav node (qual painel?), keybinding (qual atalho?),
     evento (qual emitter?), context menu (qual item?)
   - Go/HTTP: route registada em qual mux, em qual main.go, com que middleware
   - HTMX/Templ: hx-get/post para que endpoint, swap em que target

2. Se a feature e puramente interna (helper, refactor, package partilhado) que NAO
   tem observabilidade UI/runtime directa: marcar "Entry-Points: N/A (nao observavel
   externamente)" e justificar.

DURANTE /sdd-plan:
3. O ultimo passo da ordem de implementacao DEVE ser "wire entry-points +
   smoke navegavel". Nao "wire entry-points" suficiente — tem de incluir o smoke
   manual que valida que o user chega la a partir de estado vazio.

4. Cada entry-point declarado na spec deve aparecer como ficheiro modificado no plan
   (ex: Navigation.cs, cmd/<service>/main.go, layout templ). Se nao aparece -> spec
   declara entry-point mas plan nao o wire -> BLOQUEAR.

DURANTE /sdd-check:
5. Smoke manual obrigatorio comeca em "abrir app vazia" / "deploy fresco" e
   tem de chegar a feature SEM ler codigo. Se reviewer precisa de ler codigo para
   saber onde clicar -> entry-point e invisivel -> REJECT.
```

### Sinais de Detecao

- Plan.md lista "Files to create" mas zero ficheiros sao Navigation/Router/main.go
- Spec.md tem RFs mas seccao "Entry-Points" vazia ou ausente
- Unit tests cobrem o componente directamente mas zero teste cobre "navegacao + abertura"
- Reviewer pergunta "onde clico para ver isto?" e implementer responde via path do ficheiro

---

## AP-10: Storage Semantics Inconsistency

### Problema

O mesmo campo (ou valor) tem **forma canonica divergente entre paths**: o codigo escreve em formato A num path (ex: import), formato B noutro (ex: edit), e o read path assume formato C. Resultado: dados ficam corruptos consoante o caminho usado para alterar; bugs aparecem em cadeia ao tocar em qualquer ponto.

### Exemplo Real (DenTherm 2026-05)

Cadeia DT-572 → DT-568 → DT-558 → DT-573 → DT-576 → DT-578 (6 issues, ~13 pts), todos a corrigir a mesma raiz: **U-value de elemento opaco tem semantics divergentes**:

```
Path 1 — Import:           grava U *aggravated* (× 1.35 ja aplicado)
Path 2 — Mass-update:      grava U *aggravated* via OpaqueElementUValueService
Path 3 — Dialog Edit Save: grava U *base* (sem aggrav)
Path 4 — No-op edit:       copia raw 9 fields do item VM -> stored data
                          (ApplyEnvelopeItemToOpaqueElement de DT-573)
                          -> sobrescreve aggravated com base; perde × 1.35
                          -> Name "PA35 - Parede" -> "NENHUMA SOLUCAO DEFINIDA..."

Read path:                 assume aggravated (e mostra como tal)

Resultado: User abre dialog de elemento, carrega OK SEM mudar nada,
e o stored U passa de 1.30 para 0.96 (perde × 1.35) + Name e sobrescrito.
```

**Exemplo Real (Cloud)**: PLT-205 — `apiMRRResponse` tinha campos `mrr`/`mrr_eur` em dois packages diferentes (admin vs payment); JSON contract divergia entre o que o admin esperava ler e o que o payment escrevia. Bug pre-existente apanhado em E2E pos-mock. Mesma classe expos PLT-207 (`apiUser` admin vs `UserDTO` platform: `Status:string` vs `Active:bool`, `CreatedAt:string` vs `CreatedAt:time.Time`).

**Resultado**: cada bug "corrige um path" mas reabre outro. Ate alguem desenhar a tabela completa de paths × forma canonica, o codigo permanece numa instabilidade silenciosa.

### Porque Acontece com AI

O agente implementa cada path isolado ("agora vou fazer o import", "agora vou fazer o edit") sem mapear o invariante. Cada implementacao escolhe a forma "que da menos trabalho neste contexto" — frequentemente a forma raw do input, em vez de canonizar antes de gravar. Sem uma declaracao explicita na spec de "o stored canonical de X e Y", cada path diverge.

### Regra de Prevencao

```
Regra: TODA spec que persiste estado DEVE declarar explicitamente a forma canonica
de cada campo e onde se faz a conversao entre formatos.

DURANTE /sdd-specify:
1. Seccao "Storage Semantics" obrigatoria com tabela:
   | Campo | Forma canonica armazenada | Conversao no write path | Conversao no read path |
   |-------|---------------------------|-------------------------|------------------------|
   | U-value opaco | aggravated (× 1.35 aplicado) | service.AggravateU() antes de gravar | nenhuma — ler como esta |
   | Name | display synthesized | display = template.Resolve(code) | nenhuma |
   | (...) | (...) | (...) | (...) |

2. Se a feature toca um campo ja existente: ler stored canonical actual do AGENTS.md
   ou da spec original; se conflituar com o que esta feature precisa, ALERTA
   "muda semantics canonical" + plano de migracao.

3. Edge cases obrigatorios em features com persistencia:
   EC-XX: no-op edit (abrir dialog + OK sem alterar nada) preserva TODOS os campos
   EC-XX: round-trip save->reopen->save e idempotente (mesmo output)
   EC-XX: import existente vs criar novo produzem mesma estrutura stored
   EC-XX: mass-update sobre item importado preserva semantics do import path

DURANTE /sdd-plan:
4. Para cada novo path (handler/service/UI) que escreve no campo afectado:
   listar explicitamente que conversao aplica. Se algum path nao tem conversao
   documentada e o stored canonical exige uma -> BLOQUEAR.

5. Helper central para conversoes (ex: ApplyEnvelopeItemToOpaqueElement) deve
   tornar a forma canonical aparente no nome ou comentario unico:
   "// Stored canonical: aggravated U. Caller passa base, helper aggrava."

DURANTE /sdd-check:
6. Smoke manual obrigatorio inclui:
   - "No-op edit" (abrir + OK sem alterar) -> ZERO mudancas observaveis
   - Round-trip save/reload via import + edit + mass-update -> mesmo valor final
7. Test obrigatorio para cada path de escrita: assert que stored value respeita
   forma canonical declarada na spec.
```

### Sinais de Detecao

- Spec descreve campo persistente mas nao declara forma canonical
- 2+ paths gravam o mesmo campo com nenhum helper partilhado
- Bug "X corrige path A mas reabre path B" — sinal classico de semantics divergentes
- Pull request adiciona um novo path de escrita sem teste de no-op edit / round-trip
- Comentarios contraditorios em diferentes ficheiros sobre "o que esta gravado"

---

## AP-08: Multiplos Padroes para o Mesmo Problema

### Problema

Usar varias formas diferentes de resolver o mesmo problema dentro do mesmo projecto.

### Exemplo Real (DenStudio)

```
5 formas diferentes de gerir propriedades reactivas:
1. ObservableObject (CommunityToolkit)
2. INotifyPropertyChanged manual
3. SetProperty helper
4. DependencyProperty (WPF)
5. ReactiveUI

3 formas de fazer navigation:
1. NavigationService
2. Direct ViewModel switching
3. Frame navigation
```

**Resultado**: Cada developer/modulo usa um padrao diferente. Impossivel saber qual e o "correcto".

### Porque Acontece com AI

Em sessoes diferentes, o agente pode escolher patterns diferentes. Sem memoria de sessoes anteriores, nao sabe que padrao ja foi escolhido.

### Regra de Prevencao

```
Regra: UM padrao por problema. Documentado. Sempre.

1. Verificar CLAUDE.md e plan.md para padroes ja definidos
2. Se o problema ja tem padrao no projecto -> USAR esse padrao
3. Se e problema novo -> Escolher UM padrao, documentar no plan.md
4. NUNCA introduzir padrao alternativo sem remover o anterior
5. Em caso de duvida -> Procurar no codigo existente e seguir

Padroes a definir no arranque do projecto:
- Error handling
- Configuration
- Logging
- HTTP client
- Validation
- State management (SvelteKit) / Template components (Templ)
```

---

## AP-11: Codigo Nomeado por ID de Issue/Ticket

### Problema

Nomear um ficheiro, classe, metodo ou teste com o ID do issue/ticket que motivou a mudanca (`Dt695ProcessEngineVsLegacyTests.cs`, `DT846Handler`, `PLT-123Migration.go`) em vez de um nome que descreve o que o codigo FAZ.

### Exemplo Real (DenTherm)

```
Dt966AssumptionNeutralitySmokeTests.cs
Dt604NanInfinityGuardsTests.cs
Dt989GlazedSolutionReferenceNormalizationTests.cs
Dt989NoPerFractionSolutionCatalogArchTests.cs
```

**Resultado**: o nome nao diz nada sobre o comportamento testado/implementado a quem le o codigo sem o contexto do ticket. O sistema de tracking (Linear, GitHub Issues, ou outro) e uma ferramenta externa e mutavel — o codigo sobrevive-lhe. Um leitor futuro (humano ou agente) sem acesso ao ticket, ou depois de o ticket mudar de sistema, fica sem pista nenhuma do que `Dt966...` significa.

### Porque Acontece com AI

O agente le o `[ISSUE-ID]` no inicio do prompt/tarefa e usa-o como prefixo natural e unico para o novo ficheiro/classe — parece organizado e rastreavel. Sem uma regra explicita a proibir, e o padrao "obvio" a seguir. Piora em efeito de bola de neve: assim que **um** ficheiro no repo segue este padrao, um agente a explorar o codigo para decidir a convencao de nomes copia-o como se fosse o estilo da casa — mesmo que a regra o proiba, a ausencia de reforco no prompt deixa o exemplo errado falar mais alto que a regra escrita noutro sitio.

### Regra de Prevencao

```
Regra: NUNCA nomear codigo por um ID de tracking (issue/ticket/PR)

1. Nome do ficheiro/classe/metodo/teste = o que o codigo FAZ, nunca "em que ticket foi motivado"
   Exemplo: Dt695ProcessEngineVsLegacyTests.cs -> ProcessEngineVsLegacyGoldenTests.cs
2. O ID do ticket e legitimo SO na mensagem de commit (`git log`) -- e historico, nao codigo
3. Comentarios explicam a logica/regra de negocio, nunca "fix para o issue X"
4. Se o repo ja tem ficheiros com este padrao, NAO copiar o exemplo -- a presenca de
   violacoes existentes nao e defesa para criar mais uma
```

---

## Resumo: Perguntas de Auto-Verificacao

Antes de QUALQUER implementacao, o agente deve responder:

```
[ ] SEARCH: "Procurei no projecto codigo que ja faz isto?"
[ ] PROPORCIONALIDADE: "A complexidade da solucao e proporcional ao problema?"
[ ] INTERFACE: "Esta interface tem 2+ implementacoes concretas?"
[ ] CAMADA: "Este codigo esta na camada correcta?"
[ ] DUPLICACAO: "Estou a copiar codigo que ja existe noutro sitio?"
[ ] LOC: "Este ficheiro (< 500 LOC), funcao (< 45 LOC) e package (< 30 exports) esta na zona verde? Template < 200 LOC?"
[ ] PADRAO: "Estou a usar o mesmo padrao que o resto do projecto?"
[ ] SEGURANCA: "Esta medida de seguranca tem enforcement real?"
[ ] DEAD CODE: "Tudo o que criei e usado e funciona?"
[ ] ENTRY-POINT: "O utilizador chega a esta feature sem ler codigo? Menu/nav/route declarado e wired?" (AP-09)
[ ] STORAGE SEMANTICS: "Cada campo persistido tem forma canonica declarada e todos os write paths a respeitam?" (AP-10)
[ ] NAMING: "O nome deste ficheiro/classe/teste descreve o que ele faz, ou e um ID de ticket?" (AP-11)
```

---

## Referencia Rapida: Do vs Don't

| Situacao | DO (Correcto) | DON'T (Erro) |
|----------|---------------|---------------|
| Novo tipo | Struct/class directa | Interface + implementacao 1:1 |
| Dados nao-sensiveis | Guardar em plaintext | Encriptar com AES-256-GCM |
| Seguranca | Middleware que enforce | Atributo decorativo sem handler |
| Codigo util em 2+ sitios | Extrair para `pkg/` ou `lib/` | Copiar para cada modulo |
| Ficheiro com 550 LOC | Extrair antes de adicionar (zona amarela) | Continuar a adicionar |
| Feature para o futuro | Nao criar nada | Stub com NotImplemented |
| Pattern no projecto | Usar o existente | Introduzir alternativa |
| Input validation | Validar no handler/boundary | Criar sanitizer generico |
| SQL parametrizado | Confiar nos parametros | Adicionar sanitizer SQL redundante |
| Feature observavel pelo user | Declarar entry-point na spec + wire no plan + smoke navegavel no check | Implementar componente + assumir que "alguem ira ligar depois" (AP-09) |
| Campo persistido | Declarar forma canonica na spec + helper centralizado + teste round-trip | Cada path escolhe formato no momento (AP-10) |
| Nome de ficheiro/classe/teste | Nome que descreve o comportamento | Prefixo com o ID do issue/ticket (AP-11) |

---

*Baseado na analise de CODE_REVIEW_ANALYSIS.md do DenStudio (2025) — 200+ ficheiros, 42.000+ LOC revistos.*
*AP-09 e AP-10 adicionados em 2026-05 com base na cadeia DT-547..589 (DenTherm) e PLT-205/207/281 (Cloud).*
*AP-11 adicionado em 2026-08-10: dezenas de ficheiros no DenTherm (Dt604, Dt966, Dt989...) ja violavam a regra de nomenclatura do DESENVOLVIMENTO.md sem que nenhum prompt SDD a reforcasse.*
*Densare SDD - Fevereiro 2026 (rev. Agosto 2026)*
