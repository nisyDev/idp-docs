# TalentSynk — Dados e persistência

> **Escopo documental:** inventário físico conhecido, sem novo acesso ao banco nem execução de migrations. Fonte de referência: [PROJECT_STATE.md](PROJECT_STATE.md), auditoria G5A e migrations [V1–V3 do back-end `38573f9`](https://github.com/nisyDev/idp/tree/38573f9ab19b9181a3adf516533bbd8849b80def/src/main/resources/db/migration). **Este é um inventário técnico de alto nível, não um DDL reconstruído.**

## 1. Duas camadas de modelagem

A visão de negócio inclui um **catálogo lógico mais amplo** de aproximadamente 25 entidades, usado para planejar o produto. Já as migrations publicadas V1–V3 representam **13 tabelas da aplicação**. As duas contagens descrevem níveis diferentes: uma entidade conceitual pode se desdobrar em tabelas de identidade, versão e evento; uma tabela implementada não implica funcionalidade disponível na interface.

A antiga [documentação de banco no back-end](https://github.com/nisyDev/idp/blob/38573f9ab19b9181a3adf516533bbd8849b80def/docs/DATABASE.md) deve ser mantida como registro de origem até decisão explícita de migração. Seus esquemas históricos e tipos de IDs não devem substituir o SQL publicado.

## 2. Esquema físico registrado

| Migration | Tabela | Finalidade identificada na documentação |
|---|---|---|
| V1 — CORE | `cargo` | Catálogo de cargos; GET disponível somente para ativos |
| V1 — CORE | `equipe` | Estrutura de equipes |
| V1 — CORE | `usuario` | Identidade e perfil organizacional |
| V1 — CORE | `historico_equipe` | Transferências e histórico organizacional |
| V2 — PDI | `pdi` | Identidade e estado principal do PDI |
| V2 — PDI | `pdi_versao` | Conteúdo versionado do PDI |
| V2 — PDI | `acao_pdi` | Identidade e vínculo de ações |
| V2 — PDI | `acao_pdi_versao` | Conteúdo versionado de ações |
| V2 — PDI | `pdi_componente` | Identidade de componentes do PDI |
| V2 — PDI | `pdi_componente_versao` | Conteúdo versionado de componentes |
| V2 — PDI | `entrega_submissao` | Base persistente de entregas/submissão, ainda sem ciclo operacional |
| V2 — PDI | `pdi_evento` | Eventos de histórico e snapshots JSONB |
| V3 — credencial local | `usuario_credencial_local` | Credencial opcional associada a usuário |

**Arquivos do versionamento:** `V1__create_core_tables.sql`, `V2__create_pdi_initial_flow.sql`, `V3__create_usuario_credencial_local.sql`.

Este documento **não afirma** ter validado todas as colunas, constraints, índices ou checksums a partir de um banco ativo. Para essa precisão, o SQL da migration é a fonte executável; o mapeamento JPA e o contrato de teste devem ser conferidos na revisão correspondente.

## 3. Identificadores e relacionamentos

- A parte CORE utiliza identificadores **UUID**.
- O recorte PDI/V2 utiliza **`BIGINT`/`Long`** para suas identidades. Não converter o PDI para UUID com base apenas na modelagem lógica antiga.
- O PDI possui versões; ações possuem identidade própria e conteúdo versionado; componentes e entregas têm estrutura de persistência, sem que isso comprove seus endpoints ou fluxos.
- A versão do PDI mantém regras de titularidade e privacidade. O versionamento não corresponde a uma nova identidade de PDI a cada edição.
- `usuario_credencial_local` é uma associação opcional 1:1 a `usuario`, por UUID compartilhado e `@MapsId`; o usuário pode existir sem credencial local.
- Integridade e pertencimento de versões são apoiados por associações e chaves estrangeiras compostas no mapeamento correspondente, sem listar aqui combinações exatas não verificadas.

### Auditoria de PDI

Eventos atualmente reconhecidos no código: `CRIACAO`, `EDICAO_RASCUNHO`, `ACAO_ADICIONADA`, `ACAO_EDITADA`. `PdiEvento` utiliza campos JSONB para dados/snapshot, é mapeado como `@Immutable` e preserva eventos anteriores por fluxo transacional.

Na edição de ações, apenas o conteúdo de `acao_pdi_versao` é atualizado. IDs, pertencimento, ordem, estado e versão não são controlados pelo corpo do PUT. Não se cria nova versão do PDI; o evento `ACAO_EDITADA` registra o antes/depois quando há alteração efetiva dos cinco campos. A validação física específica dessa operação permanece pendente.

## 4. Limites do esquema implementado

| Recurso de negócio | Situação de dados | Situação de uso |
|---|---|---|
| Autenticação local | V1 + V3 com credencial opcional | Login real local implementado |
| Rascunhos e ações de PDI | Estrutura V2 | Operações parciais implementadas |
| Componentes e entregas | Tabelas V2 existentes | Sem ciclo funcional correspondente concluído |
| Reuniões | Código legado no back-end; V1–V3 não criam tabela de reuniões | Fluxo legado bloqueado ou demonstrativo |
| OKRs, feedbacks, competências, capacitações, avaliações e notícias | Pertencem à modelagem/visão de produto | Não declarar persistência operacional implementada no recorte atual |

A ausência de determinada tabela em V1–V3 **não invalida** a regra de negócio planejada. Também não autoriza anunciar persistência pronta.

## 5. Ambientes, migrations e testes

O banco principal definido para o produto é PostgreSQL; ambientes Neon foram utilizados para testes históricos. O perfil `auth-local` mantém o Flyway automático desativado, Hibernate com `ddl-auto=none` e SQL init desativado. Portanto, **iniciar a aplicação nesse perfil não cria ou migra o esquema**.

Evidências registradas, todas em seus ambientes e revisões específicos:

- Etapas 184G.145 e 184G.147: um cenário de `PdiPersistenceIT` e seis de `PdiServicePersistenceIT` aprovados após V3, em execuções controladas distintas.
- Etapa 184G.153: cinco testes de `UsuarioCredencialLocalPersistenceIT` aprovados no PostgreSQL, em outra execução.
- G4C.2: 267 testes isolados aprovados **sem banco**; essa contagem inclui os 82 testes de edição de ações.
- Os **cinco novos** cenários físicos da edição de ações e a revisão dedicada de `CoreSchemaValidationIT` permanecem pendentes; a compilação não conta como aprovação integrada.

Testes que gravam dados devem passar por preflight, destino autorizado, privilégios mínimos e controles de rollback específicos. Um cenário excepcional de linha de base com fixtures persistentes depende de aprovação separada. **Este documento não autoriza execução, rollback destrutivo nem aplicação de migrations.**

## 6. Critérios para uma revisão futura do esquema

1. Ler os SQLs V1–V3 na revisão pretendida e confrontá-los com os mapeamentos JPA.
2. Registrar tipos exatos, nulabilidade, PK/FK, índices e constraints **somente** após verificação no código ou banco autorizado.
3. Separar invariantes já verificadas por testes isolados das comprovações em PostgreSQL.
4. Identificar diferenças entre o catálogo lógico e a persistência vigente sem converter automaticamente uma na outra.
5. Vincular mudanças futuras de estrutura a migrations novas e testes explicitamente autorizados, preservando V1–V3.

Ver também [ARCHITECTURE.md](ARCHITECTURE.md), [API.md](API.md) e [DECISIONS.md](DECISIONS.md).
