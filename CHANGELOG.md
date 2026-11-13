# TalentSynk — Histórico de mudanças

> **Registro de marcos técnicos e documentais, não de releases.** Não existem tags ou versões públicas formalmente certificadas por este documento. As datas abaixo são usadas apenas quando registradas nas evidências; os demais marcos são identificados por fase/commit. A preparação documental é distinta de commits, publicação Git e deploy.

## Consolidação documental — G5B.1–G5B.3

- Preparados `ARCHITECTURE.md`, `DATABASE.md`, `API.md`, `DECISIONS.md`, `ROADMAP.md` e este `CHANGELOG.md`.
- `BUSINESS.md` revisado para coerência com a visão do produto, **sem alteração de conteúdo**.
- `README.md` e `PROJECT_STATE.md` foram revisados na G5B.1, com referências atualizadas na G5B.3.
- A conferência direta completa dos URLs e contratos do GitHub não integra as validações documentais; verificar links e código de referência a cada evolução.

## Marcos do código publicado

| Fase | Marco | Referência | Observação |
|---|---|---|---|
| G3 | Front-end publicado após build isolado aprovado (`tsc -b && vite build`) | [`idp-frontend@01cd6d3`](https://github.com/nisyDev/idp-frontend/commit/01cd6d322c1efb4b843756f13471c196851ecf42) | Quatro arquivos locais pendentes não incluídos; sem deploy |
| G4C.1 | Compilação isolada do back-end aprovada | [`idp@38573f9`](https://github.com/nisyDev/idp/commit/38573f9ab19b9181a3adf516533bbd8849b80def) | 74 fontes de produção e 25 de teste; sem executar JUnit ou banco |
| G4C.2 | 267 testes isolados aprovados em 20 classes | [`idp@38573f9`](https://github.com/nisyDev/idp/commit/38573f9ab19b9181a3adf516533bbd8849b80def) | Inclui 82 testes de edição de ações; exclui PostgreSQL |
| G4C.3 | Back-end atual publicado no repositório oficial | [`idp@38573f9`](https://github.com/nisyDev/idp/commit/38573f9ab19b9181a3adf516533bbd8849b80def) | Protótipo histórico preservado no repositório destino; origem local antiga não foi incorporada |
| G5A | Auditoria documental de três repositórios e revisão de estados | [PROJECT_STATE.md](PROJECT_STATE.md) | Nenhuma alteração autorizada nessa fase |
| G5B.1 | README e PROJECT_STATE preparados/revisados no clone isolado | Repositório documental, base `7afc5e2` | Sem commit/push no encerramento da etapa |

## Marcos anteriores de validação

| Data ou etapa | Registro | Limitação |
|---|---|---|
| 11/11/2026 — 184G.145 | `PdiPersistenceIT`: um cenário pós-V3 aprovado no Neon | Execução controlada e independente; histórico Flyway e rollback conforme contrato |
| 11/11/2026 — 184G.147 | `PdiServicePersistenceIT`: seis cenários pós-V3 aprovados no Neon | Reexecução de casos históricos; não cobre novos cinco cenários de edição de ações |
| 184G.153 | `UsuarioCredencialLocalPersistenceIT`: cinco testes no PostgreSQL aprovados | Teste dedicado da V3, distinto das demais execuções |
| Incremento 3A | PUT de edição de ações com `ACAO_EDITADA` implementado no back-end | Interface e rollback físico específico pendentes |
| Incremento 3B.1.1 | 148 testes isolados aprovados no recorte então executado, incluindo 82 da edição | Não somar aos 267 posteriores |
| Incremento 3B.2A/2A.1 | Cinco cenários PostgreSQL novos acrescentados e compilados | Não executados ou aprovados fisicamente |

O detalhe das evidências anteriores, incluindo o catálogo de cargos e os testes CORE, permanece em [PROJECT_STATE.md](PROJECT_STATE.md). Uma execução histórica não pode ser estendida automaticamente a novos commits, ambientes ou versões de teste.

## Convenções para futuras entradas

Para cada alteração confirmada, registrar:

- **Data** comprovada, quando disponível, e commit/repositório.
- **Escopo:** API, interface, persistência, segurança ou documentação.
- **Evidência:** build, testes isolados, testes reais de banco e/ou validação manual, distinguindo cada uma.
- **Impacto e compatibilidade:** rotas, migrations, regras, perfis e eventuais migrações de dados.
- **Status de publicação:** preparado, commitado, enviado ao GitHub, implantado e validado; essas etapas não são sinônimas.

Não criar entradas como `v1.0.0` ou “beta lançado” sem release e evidência correspondentes.
