# TalentSynk — Contratos HTTP disponíveis

> **Escopo documental:** endpoints observados e descritos no [PROJECT_STATE.md](PROJECT_STATE.md) para o back-end publicado em [`38573f9`](https://github.com/nisyDev/idp/tree/38573f9ab19b9181a3adf516533bbd8849b80def). Não é um contrato OpenAPI gerado nem uma bateria de testes HTTP executada nesta fase. Campos/status não confirmados são indicados como tal.

## 1. Premissas

- API REST Spring Boot, com prefixo `/api` nos endpoints aqui listados.
- No desenvolvimento local o front-end acessa `/api` pelo proxy Vite; `auth-local` usa loopback e a porta configurada 8081.
- Sessão HTTP e proteção CSRF: obter token por `GET /api/csrf` antes das operações que o exigem, segundo o fluxo implementado no front-end. Cabeçalhos e formato exato da resposta CSRF devem ser confirmados pelo contrato do controller antes de publicar exemplos executáveis.
- Identidade e authorities são estabelecidas **no back-end**. Não enviar um perfil escolhido pelo cliente como prova de autorização.
- Dados de outros titulares não devem ser expostos por endpoints de PDI e ações.
- `demo-local` e o perfil padrão têm regras de segurança distintas de `auth-local`: a existência do controller não implica exposição em todos os perfis.

## 2. Autenticação — perfil `auth-local`

| Método | Rota | Propósito | Resposta/controle registrado |
|---|---|---|---|
| GET | `/api/csrf` | Obter token CSRF | Disponível antes de login; `Cache-Control: no-store` |
| POST | `/api/auth/login` | Login com e-mail e senha | Sucesso **204**; credenciais inválidas **401**; CSRF obrigatório |
| GET | `/api/auth/me` | Obter identidade autenticada | Campos `id`, `nome`, `email`, `perfil`; `no-store` |
| POST | `/api/auth/logout` | Invalidar sessão | Sucesso **204**; CSRF obrigatório |

Senha local validada por BCrypt no back-end. `me` não expõe hash nem dados internos da credencial. A opção “lembrar acesso” afeta a persistência do cookie de sessão, sem garantir renovação automática da sessão no servidor.

## 3. Rascunhos de PDI

| Método | Rota | Operação documentada | Controle registrado |
|---|---|---|---|
| POST | `/api/pdis` | Criar PDI e versão inicial | **201**, DTO e cabeçalho `Location`; titular vem da sessão |
| GET | `/api/pdis` | Listar rascunhos do titular | Não é consulta de todos os PDIs da organização |
| GET | `/api/pdis/{id}` | Consultar rascunho próprio | Recurso inexistente/de outro titular tratado como **404** |
| PUT | `/api/pdis/{id}` | Editar dados básicos | Limitado à versão inicial em rascunho nunca submetido |

Contrato funcional conhecido: título, objetivo e datas inicial/final de PDI são opcionais; intervalo de datas invertido é inválido. Os nomes JSON exatos dos campos devem ser conferidos nos DTOs antes de documentar um payload literal. `PDI` é criado em `NAO_INICIADO`, versão inicial `1`/`RASCUNHO`; o servidor define titular, estados e IDs.

**Não estão disponíveis** submissão, aprovação, rejeição, execução ou conclusão do PDI por essa API.

## 4. Ações vinculadas a um rascunho

| Método | Rota | Operação documentada | Controle registrado |
|---|---|---|---|
| POST | `/api/pdis/{pdiId}/acoes` | Cadastrar ação com conteúdo versionado | **201**, DTO e `Location` |
| GET | `/api/pdis/{pdiId}/acoes` | Listar ações da versão autorizada | Autenticação e titularidade |
| GET | `/api/pdis/{pdiId}/acoes/{acaoId}` | Consultar uma ação | Ação e PDI devem pertencer à versão autorizada |
| PUT | `/api/pdis/{pdiId}/acoes/{acaoId}` | Atualizar cinco campos de conteúdo | **200**, `AcaoPdiResponse`, `Cache-Control: no-store`, apenas em `auth-local` |

### Edição de ação — recorte implementado no back-end

A edição por PUT **ainda não possui interface no front-end publicado**. Recebe somente os campos de conteúdo:

| Campo documentado | Tipo conceitual | Regra conhecida |
|---|---|---|
| `titulo` | texto | Opcional; admite `null` |
| `objetivo` | texto | Opcional; admite `null` |
| `resultadoEsperado` | texto | Opcional; admite `null` |
| `inicioPlanejado` | data | Opcional; admite `null` |
| `fimPlanejado` | data | Opcional; admite `null`; se ambas as datas existirem, início não pode superar fim |

O PUT substitui integralmente esses cinco campos — inclusive por nulos — sem receber controle de identidade, vínculo, ordem, versão, estado ou progresso. Exige titular ativo, PDI `NAO_INICIADO`, versão inicial `1` em `RASCUNHO`, nunca submetida, e ausência de versão aprovada vigente. **Sem mudança de conteúdo, não realiza UPDATE nem cria evento**. Se houver alteração, o evento `ACAO_EDITADA` registra o antes/depois na mesma transação. A validação de rollback físico PostgreSQL permanece pendente.

| Caso da edição de ação | Código HTTP registrado |
|---|---|
| Atualização válida | 200 |
| JSON ou data inválidos | 400 |
| Sessão inválida | 401 |
| CSRF inválido ou acesso negado | 403 |
| PDI/ação inexistente ou de outro titular | 404 |
| Estado incompatível com edição | 409 |

Esses códigos pertencem ao recorte do **PUT de ações**; não são uma garantia automática dos mesmos códigos para todas as rotas. Exclusão, ordenação, início, progresso e conclusão de ações permanecem indisponíveis.

## 5. Consulta do catálogo de cargos ativos

| Método | Rota | Disponibilidade | Resposta conhecida |
|---|---|---|---|
| GET | `/api/cargos` | `auth-local`, usuários ativos com papéis `COLABORADOR`, `GESTOR` ou `RH_ADMIN` | **200**, lista de `CargoResponse`; pode ser `[]` |

Os DTOs de cargo expõem `id`, `nome`, `descricao`, `nivel`, `ativo`. A consulta usa somente cargos `ativo=true` para **todos os três papéis**, inclusive RH. A validação integrada histórica utilizou apenas cargos ativos; excluir registros inativos de um conjunto real misto permanece como evidência pendente.

**Não existem operações HTTP de escrita de cargos autorizadas** neste recorte. O serviço CORE interno não é sinônimo de API administrativa pública. Perfis `demo-local` e padrão bloqueiam esse catálogo na configuração descrita.

## 6. O que não anunciar como API operacional

- APIs completas de administração de usuários/equipes/cargos.
- API de histórico de PDI/eventos.
- PDI submetido/aprovado, ações em execução e conclusão formal.
- OKRs, avaliações, feedbacks, competências, capacitações ou notícias.
- Autenticação corporativa Microsoft Entra ID ou Microsoft Teams.
- Rotas legadas eventualmente existentes no código, mas **bloqueadas** pelas regras atuais.

## 7. Fonte de verdade e manutenção

| Finalidade | Evidência |
|---|---|
| Controllers e rotas | [Controllers do back-end](https://github.com/nisyDev/idp/tree/38573f9ab19b9181a3adf516533bbd8849b80def/src/main/java/com/empresa/pdi/controller) |
| Segurança | [LocalHttpSecurityConfig.java](https://github.com/nisyDev/idp/blob/38573f9ab19b9181a3adf516533bbd8849b80def/src/main/java/com/empresa/pdi/security/LocalHttpSecurityConfig.java) |
| Edição de ações | [AcaoPDIController.java](https://github.com/nisyDev/idp/blob/38573f9ab19b9181a3adf516533bbd8849b80def/src/main/java/com/empresa/pdi/controller/AcaoPDIController.java) |
| Modelo e regras | [PdiService.java](https://github.com/nisyDev/idp/blob/38573f9ab19b9181a3adf516533bbd8849b80def/src/main/java/com/empresa/pdi/service/PdiService.java) e [AcaoPdiService.java](https://github.com/nisyDev/idp/blob/38573f9ab19b9181a3adf516533bbd8849b80def/src/main/java/com/empresa/pdi/service/AcaoPdiService.java) |
| Limites de validação | [PROJECT_STATE.md](PROJECT_STATE.md) |

Antes de apresentar exemplos completos de requisições e respostas, revisar DTOs, serialização, validação Bean Validation, conteúdo de erros, cookies e configurações reais no commit de referência. Não incluir tokens, valores de sessão ou credenciais em exemplos públicos.
