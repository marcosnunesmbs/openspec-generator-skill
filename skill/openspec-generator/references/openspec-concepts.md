# OpenSpec — Conceitos e Referência

## Filosofia

OpenSpec é construído em torno de quatro princípios:

```
fluid not rigid       — sem phase gates, trabalhe no que faz sentido
iterative not waterfall — aprenda enquanto constrói, refine enquanto avança
easy not complex      — setup leve, mínima cerimônia
brownfield-first      — funciona com codebases existentes, não só greenfield
```

## Estrutura de Alto Nível

```
openspec/
├── specs/           # Source of truth — como o sistema funciona agora
│   ├── auth/
│   │   └── spec.md
│   └── payments/
│       └── spec.md
└── changes/         # Modificações propostas (uma pasta por change)
    └── add-oauth/
        ├── spec.delta.md
        ├── design.md
        └── tasks.md
```

## Formato de spec.md

```markdown
# {Domínio} Specification

## Purpose
{Descrição de alto nível do domínio.}

## Requirements

### Requirement: {Nome}
The system SHALL/MUST/SHOULD {comportamento observável}.

#### Scenario: {Caso}
- GIVEN {pré-condição}
- WHEN {ação}
- THEN {resultado}
- AND {resultado adicional}
```

## RFC 2119 Keywords

| Keyword | Significado |
|---------|-------------|
| MUST / SHALL | Requisito absoluto — sem exceções |
| SHOULD | Recomendado — exceções justificadas existem |
| MAY | Opcional |
| MUST NOT / SHALL NOT | Absolutamente proibido |

## O que é uma Spec (e o que não é)

**É:** contrato de comportamento  
**Não é:** plano de implementação

✅ Comportamento observável externamente  
✅ Entradas, saídas, condições de erro  
✅ Restrições (segurança, privacidade, performance)  
✅ Cenários testáveis (Given/When/Then)

❌ Nomes de classes/funções internas  
❌ Escolhas de biblioteca/framework  
❌ Detalhes de implementação  
❌ Planos de execução passo a passo

## Níveis de Rigor

### Lite (padrão — use na maioria dos casos)
- Requisitos curtos focados em comportamento
- Escopo e não-escopo claros
- Alguns cenários de aceitação concretos

### Full (apenas para mudanças de alto risco)
- Mudanças cross-team ou cross-repo
- Mudanças de API/contrato
- Migrações de dados
- Requisitos de segurança/privacidade
- Onde ambiguidade causa retrabalho caro

## Changes vs Specs

**Specs** (`openspec/specs/`) = source of truth atual  
**Changes** (`openspec/changes/`) = modificações propostas

Uma change pode conter:
- `spec.delta.md` — o que muda na spec
- `design.md` — decisões de implementação
- `tasks.md` — trabalho concreto a fazer

Quando a change é arquivada (mergeada), seus deltas se incorporam às specs.
