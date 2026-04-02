---
name: openspec-generator
description: Analyze an existing project and generate or update OpenSpec spec artifacts. Use this skill whenever the user wants to create or update specs for a project using OpenSpec, initialize an openspec/ folder, document system behavior, generate spec.md files from existing code, map features to requirements and scenarios, audit existing specs for gaps, or identify new domains not yet covered. Trigger this skill when the user mentions "openspec", "create specs", "generate specifications", "update specs", "audit specs", "document system behavior", "initialize openspec", "specs missing", or asks to analyze a codebase and produce structured requirements. Also trigger for brownfield projects where the user wants to capture existing behavior formally.
---

# OpenSpec Generator

Skill for analyzing an existing project and generating or updating specification artifacts following the OpenSpec structure and philosophy.

## OpenSpec Philosophy (Summary)

```
fluid not rigid       — no phase gates, work on what makes sense
iterative not waterfall — learn while building
easy not complex      — light setup, minimal ceremony
brownfield-first      — works with existing codebases
```

**Specs** describe how the system **currently behaves** (source of truth).  
**Changes** are proposed modifications, living in separate folders until merged.

---

## Processo de Geração

### Passo 0 — Verificar Pré-condições e Determinar Modo (OBRIGATÓRIO)

**Antes de qualquer coisa**, verifique o estado do projeto para determinar o **modo de operação**:

```bash
# 1. Verificar se a pasta openspec/ existe
[ ! -d "openspec" ] && echo "ERRO: openspec/ não encontrada" && exit 1

# 2. Verificar se openspec/specs/ existe
[ ! -d "openspec/specs" ] && echo "ERRO: openspec/specs/ não encontrada" && exit 1

# 3. Contar specs existentes e listar domínios cobertos
SPECS_COUNT=$(find openspec/specs -name "spec.md" | wc -l)
echo "Specs encontradas: $SPECS_COUNT"
find openspec/specs -name "spec.md" | sort
```

**Resultado da verificação determina o modo:**

| Situação | Modo | Ação |
|----------|------|------|
| `openspec/` não existe | ❌ Bloqueado | Informar: "Inicialize o OpenSpec primeiro criando `openspec/` e `openspec/specs/`." |
| `openspec/specs/` não existe | ❌ Bloqueado | Informar: "Crie a pasta `openspec/specs/` antes de continuar." |
| `openspec/specs/` **vazia** | 🟢 **Modo Inicial** | Prosseguir para Passo 1 — gerar specs do zero |
| `openspec/specs/` **com specs** | 🔵 **Modo Auditoria** | Prosseguir para Passo 1-A — auditar e expandir |

---

### Passo 1 — Explorar o Projeto (Modo Inicial)

> **Pule para o Passo 1-A se estiver no Modo Auditoria.**

Explore o projeto para identificar todos os domínios:

```bash
# Estrutura de arquivos relevantes
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" \) \
  | grep -v node_modules | grep -v dist | grep -v .next | head -80

# Estrutura de pastas
ls -la src/ 2>/dev/null || ls -la app/ 2>/dev/null || ls -la

# Contexto do projeto
cat package.json 2>/dev/null | head -40
cat README.md 2>/dev/null | head -50
```

**O que procurar:**
- Pastas de features/domínios (`auth/`, `payments/`, `users/`, `orders/`)
- Rotas de API (`routes.ts`, `controller.ts`, `api/`)
- Modelos/entidades (`schema`, `model`, `entity`, `dto`)
- Configurações (`.env.example`)
- Testes existentes — revelam comportamento esperado implícito

---

### Passo 1-A — Auditar Specs Existentes (Modo Auditoria)

Quando `openspec/specs/` já tem conteúdo, o objetivo é **identificar lacunas** — domínios não cobertos e requirements faltando nas specs existentes.

#### 1-A.1 — Ler todas as specs existentes

```bash
# Listar domínios já cobertos
find openspec/specs -name "spec.md" | sort

# Ler cada spec existente
for spec in $(find openspec/specs -name "spec.md"); do
  echo "=== $spec ==="
  cat "$spec"
  echo ""
done
```

#### 1-A.2 — Mapear domínios do código

```bash
# Explorar estrutura do projeto para encontrar domínios não documentados
find . -type f \( -name "*.ts" -o -name "*.tsx" \) \
  | grep -v node_modules | grep -v dist | grep -v ".spec.ts" \
  | grep -E "(controller|service|route|module)" | sort

ls src/ 2>/dev/null || ls app/ 2>/dev/null
```

#### 1-A.3 — Identificar lacunas

Construa dois mapas:

**Domínios no código mas SEM spec:**
- Liste pastas/features no código que não têm pasta correspondente em `openspec/specs/`
- Esses domínios precisam de specs novas

**Domínios COM spec mas com possíveis gaps:**
- Compare controllers/routes com requirements existentes
- Identifique endpoints ou comportamentos sem cenário documentado
- Verifique se edge cases críticos (erros, permissões, limites) estão cobertos

#### 1-A.4 — Apresentar diagnóstico ao usuário

Antes de escrever qualquer coisa, apresente o resultado da análise:

```
📋 Diagnóstico OpenSpec

✅ Domínios já cobertos (X specs):
  - auth/        → Y requirements, Z scenarios
  - payments/    → Y requirements, Z scenarios

🆕 Domínios sem spec (encontrados no código):
  - notifications/  → detectado em src/notifications/
  - reports/        → detectado em src/reports/

⚠️  Gaps nas specs existentes:
  - auth/: endpoint POST /auth/refresh não documentado
  - payments/: cenário de estorno (refund) ausente

Deseja que eu:
[1] Crie specs para os domínios novos
[2] Preencha os gaps nas specs existentes
[3] Faça os dois
```

Aguarde confirmação ou instrução do usuário antes de prosseguir.

---

### Passo 2 — Identificar Domínios (Modo Inicial)

Mapeie as áreas funcionais do sistema:

| Tipo de organização | Exemplos de domínios |
|---------------------|----------------------|
| Por feature         | `auth`, `payments`, `search`, `notifications` |
| Por componente      | `api`, `frontend`, `workers`, `database` |
| Por bounded context | `ordering`, `fulfillment`, `inventory` |

Para cada domínio, registre:
- **Nome** — pasta em `openspec/specs/`
- **Responsabilidade** — o que faz
- **Fontes de evidência** — quais arquivos revelam o comportamento

---

### Passo 3 — Gerar ou Atualizar os spec.md

#### Para domínios novos — criar `openspec/specs/{domínio}/spec.md`:

```markdown
# {Domínio} Specification

## Purpose
{Descrição de alto nível do que esse domínio faz no sistema.}

## Scope
O que está **incluído** nessa spec:
- {comportamento 1}
- {comportamento 2}

O que está **fora do escopo**:
- {o que não está aqui e por quê}

## Requirements

### Requirement: {Nome do Comportamento}
The system SHALL/MUST/SHOULD {descrição do comportamento observável}.

#### Scenario: {Caso feliz}
- GIVEN {pré-condição}
- WHEN {ação ou evento}
- THEN {resultado esperado}
- AND {resultado adicional, se houver}

#### Scenario: {Caso de erro ou edge case}
- GIVEN {pré-condição}
- WHEN {ação inválida ou situação excepcional}
- THEN {como o sistema responde}
```

#### Para gaps em specs existentes — adicionar na spec correspondente:

Inserir os requirements ou scenarios faltantes na seção `## Requirements` existente, mantendo o estilo e formato da spec original. **Não reescrever o que já existe.**

**Regras para o conteúdo:**

| ✅ Incluir | ❌ Evitar |
|------------|-----------|
| Comportamento observável externamente | Nomes de classes/funções internas |
| Entradas, saídas e condições de erro | Escolhas de biblioteca ou framework |
| Restrições externas (segurança, privacidade) | Detalhes de implementação passo a passo |
| Cenários testáveis (Given/When/Then) | Planos de execução detalhados |

**Keywords RFC 2119:**
- **MUST / SHALL** — requisito absoluto
- **SHOULD** — recomendado, mas exceções existem
- **MAY** — opcional

---

### Passo 4 — Atualizar o README do OpenSpec

Se `openspec/README.md` não existe, crie. Se já existe, **atualize apenas a tabela de domínios** para incluir os novos:

```markdown
# OpenSpec — {Nome do Projeto}

Especificações comportamentais do sistema, organizadas por domínio.

## Estrutura

### specs/
Source of truth — como o sistema atualmente se comporta.

| Domínio | Descrição |
|---------|-----------|
| [auth](specs/auth/spec.md) | Autenticação e gerenciamento de sessão |
| [payments](specs/payments/spec.md) | Processamento de pagamentos |
| ... | ... |

### changes/
Modificações propostas. Cada change vive em sua própria pasta até ser mergeada.

## Convenções

- Requisitos usam keywords RFC 2119 (SHALL, MUST, SHOULD, MAY)
- Cenários seguem formato Given/When/Then
- Specs descrevem **comportamento**, não implementação
```

---

### Passo 5 — Validar Qualidade

**Checklist por spec gerada ou modificada:**
- [ ] Tem `## Purpose` claro e conciso?
- [ ] Cada `### Requirement:` usa keyword RFC 2119?
- [ ] Cada requirement tem pelo menos 1 scenario?
- [ ] Os scenarios cobrem happy path E pelo menos 1 edge case?
- [ ] Não menciona classes, métodos ou libs internas?
- [ ] O comportamento descrito é **verificável/testável**?
- [ ] No Modo Auditoria: o conteúdo existente foi preservado?

---

## Estratégia por Tipo de Projeto

### NestJS / Express (Backend API)
Analisar: `controller`, `routes`, `dto`, `guard`, `middleware`, testes `*.spec.ts`  
Domínios típicos: `auth`, `users`, `{entidade-principal}`, `notifications`

### React / Next.js (Frontend)
Analisar: `pages/`, `app/`, formulários, hooks de data fetching, `store/`, `context/`  
Domínios típicos: `ui`, `navigation`, `forms`, `{feature-principal}`

### Full-stack (ex: Quantix — React + NestJS)
Dividir por feature end-to-end:
- `openspec/specs/auth/` — autenticação
- `openspec/specs/transactions/` — transações
- `openspec/specs/accounts/` — contas e saldos
- `openspec/specs/reports/` — relatórios e exportações

---

## Output Esperado

Ao concluir, reporte ao usuário:

**Modo Inicial:**
- Domínios identificados e specs criadas
- Requirements por domínio
- Áreas com evidência fraca (precisam de revisão humana)

**Modo Auditoria:**
- Domínios novos criados
- Gaps preenchidos nas specs existentes
- Itens que requerem decisão humana (comportamento ambíguo no código)
- Domínios no código ainda sem cobertura suficiente

---

## Referências

- [Conceitos OpenSpec](references/openspec-concepts.md) — Filosofia e formato completo
