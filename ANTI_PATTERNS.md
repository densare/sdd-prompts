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

---

*Baseado na analise de CODE_REVIEW_ANALYSIS.md do DenStudio (2025) — 200+ ficheiros, 42.000+ LOC revistos.*
*Densare SDD - Fevereiro 2026*
