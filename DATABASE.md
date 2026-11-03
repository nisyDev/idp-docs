# TalentSynk — DATABASE.md

> **Estado:** especificação documental extraída da planilha `PDI_Modelagem - Copia(2).xlsx` em 03/11/2026. Não é uma migração SQL, nem implica que os pontos pendentes estejam resolvidos.

## 1. Objetivo e fonte de verdade

Este documento orienta a implementação do esquema relacional do TalentSynk. A aba **Entidades** define campos e regras locais; **Relacionamentos** define vínculos e integridade; **Matriz de Permissões** e **Perfis de Acesso** orientam a autorização da aplicação. Textos da planilha foram mantidos sem substituição silenciosa de decisões. Quando um campo está vazio na origem, a documentação mostra `—`.

**Inventário:** 25 entidades, 199 linhas de campos, 44 relacionamentos, 33 ações de permissão e 3 perfis.

## 2. Convenções de implementação

- Os tipos (`string`, `timestamp`, `UUID PK`, `JSON`, `ENUM` etc.) são os **tipos lógicos descritos na planilha**, não DDL validado para um SGBD específico.
- Manter identificadores e nomes de campos conforme o catálogo até resolver as divergências registradas na seção 6.
- Não converter automaticamente regras textuais em `CHECK`, `UNIQUE`, gatilhos ou cascatas sem confirmar seu comportamento e o SGBD escolhido.
- Para histórico imutável, impedir edição/exclusão pela aplicação e definir política de retenção e auditoria antes da produção.
- Controle de acesso é aplicado nas APIs/serviços e consultas, não apenas na interface. Dados de feedback anônimo exigem proteção inclusive nos históricos.

## 3. Catálogo de entidades e campos

### 3.1 `Usuario`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identidade do usuário. |
| `ad_object_id` | string | Azure AD | Âncora de sincronização. Imutável após criação. |
| `nome` | string | — | — |
| `email` | string | — | — |
| `cargo_id` | UUID FK | Cargo | Atualiza campo ao trocar cargo. Não cria novo usuário. Impedir novas atribuições a cargos inativos, mantendo os vínculos existentes. |
| `gestor_id` | UUID FK (auto-ref) | Usuario | Atualiza campo ao trocar cargo. Não cria novo usuário. Ao alterar o gestor, redirecionar os PDIs pendentes ao novo gestor e notificá-lo. |
| `perfil_acesso` | ENUM: COLABORADOR, GESTOR, RH_ADMIN | — | Mapeado a partir dos grupos do Azure AD. |
| `ativo` | boolean | — | Soft delete. Exclusão preserva histórico. |
| `data_criacao` | timestamp | — | Armazena data de criação de usuário. |
| `data_atualizacao` | timestamp | — | Armazena data de atualização de usuário. |
| `equipe_atual_id` | UUID FK | Equipe | Identifica a equipe atual do colaborador |

### 3.2 `Cargo`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | — |
| `nome` | string | — | Denominação do cargo, como Desenvolvedor, Analista ou Coordenador. |
| `descricao` | string | — | — |
| `nivel` | string | ENUM: JUNIOR, PLENO, SENIOR, NAO_APLICAVEL | Classifica a senioridade do cargo. |
| `ativo` | boolean | — | Não excluir fisicamente se tiver usuários vinculados. Quando descontinuados, devem receber ativo = false, preservando seus vínculos e histórico. |

### 3.3 `Equipe`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | — |
| `nome` | string | — | — |
| `lider_id` | UUID FK | Usuario | Referencia o usuário responsável pela liderança da equipe. O líder deve estar ativo e pertencer à própria equipe. Uma transferência que retire o líder da equipe exige a definição prévia de um substituto ou a remoção da liderança. |
| `descricao` | text | — | — |
| `ativo` | boolean | — | — |

### 3.4 `HistoricoEquipe`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador único do evento de movimentação. |
| `usuario_id` | UUID FK | — | Identifica o colaborador transferido. Obrigatório. |
| `equipe_origem_id` | UUID FK (nullable) | — | Equipe anterior. Pode ser nulo no primeiro vínculo do colaborador. |
| `equipe_destino_id` | UUID FK | — | Equipe de destino. Obrigatória e deve estar ativa no momento da movimentação. |
| `data_transferencia` | timestamp | — | Registra a data e hora efetivas da movimentação. Obrigatório. |
| `realizado_por_id` | UUID FK, nullable | Usuario | Identifica quem registrou a transferência. Null para importações ou eventos automáticos. |
| `observacao` | text, nullable | — | Justificativa ou observação opcional sobre a movimentação. |

### 3.5 `PDI`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | — |
| `usuario_id` | UUID FK | Usuario | Proprietário obrigatório do PDI; alvo da regra de 'próprio PDI'. É permitido múltiplos PDIs simultaneamente, com acompanhamento independente. |
| `titulo` | string | — | — |
| `objetivo` | text | — | — |
| `prazo` | date | — | — |
| `status_aprovacao` | ENUM: RASCUNHO, PENDENTE, APROVADO, REJEITADO | — | Inicia em RASCUNHO. Controla exclusivamente o fluxo de aprovação. Feito somente por usuários autorizados. Um PDI rejeitado pode retornar a RASCUNHO para edição e posteriormente ser reenviado para aprovação. |
| `status_execucao` | ENUM: NAO_INICIADO, EM_ANDAMENTO, CONCLUIDO, CANCELADO | — | Rejeitado pode voltar a RASCUNHO para reedição. Fluxo precisa ser implementado como máquina de estados validada no service. Quando todas as ações vinculadas estiverem CONCLUIDAS, atualizar automaticamente o PDI para CONCLUIDO. |
| `decido_por` | UUID FK (nullable) | Usuario | Inicia em NAO_INICIADO. A execução somente pode avançar após aprovação (Gestor direto do colaborador ou RH, ambos registrados após a decição) |
| `data_aprovacao` | timestamp (nullable) | — | — |
| `motivo_rejeicao` | text (nullable) | — | Obrigatório ao rejeitar: decisão pendente. Preenchido pelo perfil GESTOR e RH |
| `data_criacao` | timestamp | — | — |
| `data_atualizacao` | timestamp | — | Registrar a data da última atualização do PDI. |

### 3.6 `AcaoPDI`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | — |
| `pdi_id` | UUID FK | PDI | Ação sempre pertence a um PDI. |
| `titulo` | string | — | Resumo curto da ação. |
| `descricao` | text | — | Detalhamento (opcional) |
| `tipo_acao` | ENUM: CURSO, MENTORIA, PROJETO, LEITURA, OUTRO | — | — |
| `prazo` | date | — | Prazo individual da atividade |
| `status_acao` | ENUM: PENDENTE, EM_ANDAMENTO, CONCLUIDA, CANCELADA | — | Cada ação possui status independente. |
| `recurso` | string | — | URL ou nome do recurso. |
| `metrica` | string | — | Como medir o resultado da ação. |
| `data_conclusao` | date (nullable) | — | Data real de conclusão. |
| `ordem` | integer | — | Posição da atividade na apresentação |
| `progresso_percentual` | decimal (0-1000) | — | Progresso parcial da ação; conclusão de capacitação vinculada pode atualizar este valor, sem definir automaticamente status_acao = CONCLUIDA. |

### 3.7 `OKR`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador único |
| `usuario_id` | UUID FK | Usuario | Proprietário do OKR pessoal. Responsável pelo OKR de equipe é NULL. |
| `equipe_id` | UUID FK (nullable) | Equipe | Null = OKR pessoal. Preenchido = OKR de equipe. |
| `pdi_id` | UUID FK (nullable) | PDI | Vínculo opcional, permitido somente para OKRs pessoais. |
| `titulo` | string | — | Objetivo do OKR |
| `descricao` | text | — | Detalhamento (opcional) |
| `ciclo` | string | — | Ex: 2025-S1, 2025-Q3. |
| `data_inicio` | date | — | Inicio do ciclo |
| `data_fim` | date | — | Fim do ciclo |
| `status_aprovacao` | ENUM: RASCUNHO, PENDENTE, APROVADO, REJEITADO, CONCLUIDO | — | Controla a aprovação inicial do OKR, sem bloquear alterações posteriores autorizadas. |
| `status_execucao` | ENUM: NAO_INICIADO, EM_ANDAMENTO, CONCLUIDO, CANCELADO | — | Acompanhamento do objetivo |
| `aprovado_por` | UUID FK (nullable) | Usuario | Registrar o usuário responsável pela aprovação inicial, respeitando a hierarquia e as permissões do RH. |
| `data_aprovacao` | timestamp (nullable) | — | — |

### 3.8 `OkrEvento`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador único do evento. |
| `okr_id` | UUID FK NOT NULL | OKR | OKR ao qual o evento pertence. |
| `usuario_id` | UUID FK NOT NULL | Usuário | Responsável pela alteração. |
| `tipo_evento` | ENUM | — | Registrar alterações posteriores à aprovação, preservando o histórico. |
| `dados_anteriores` | JSON nullable | — | Valores anteriores dos campos modificados. |
| `dados_novos` | JSON | — | Valores novos após a alteração. |
| `data_evento` | timestamp | — | Data e hora do acontecimento. |
| `observação` | text nullable | — | Comentário ou justificativa, quando necessário. |

### 3.9 `KeyResult`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador único do resultado-chave. |
| `okr_id` | UUID FK | OKR | Cada KeyResult pertence a um único OKR. |
| `descricao` | text | — | Resultado mensurável. Editável pelo responsável autorizado, com histórico. |
| `tipo_metrica` | ENUM: NUMERICO, PERCENTUAL, BOOLEANO | — | Define como o resultado será medido. |
| `unidade` | string (nullable) | — | Unidade de medida, como horas, entregas ou pontos. |
| `valor_inicial` | numeric | — | Valor de referência para calcular a evolução. |
| `meta` | numeric | — | Resultado esperado. Alterações devem preservar a meta anterior. |
| `progresso_atual` | numeric | — | Valor alcançado até o momento. |
| `data_atualizacao` | timestamp | — | Data e hora da última atualização do resultado-chave. |

### 3.10 `Feedback`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | — |
| `autor_id` | UUID FK | Usuario | — |
| `destinatario_id` | UUID FK | Usuario | — |
| `situacao` | text | — | S do modelo SBI: contexto onde ocorreu o comportamento. |
| `comportamento` | text | — | B do modelo SBI: o que foi observado. |
| `impacto` | text | — | I do modelo SBI: efeito do comportamento. |
| `tipo_feedback` | ENUM: POSITIVO, CONSTRUTIVO, DESENVOLVIMENTO | — | — |
| `anonimo` | boolean | — | Oculta autor_id na interface. Preserva no banco para auditoria. |
| `editado` | boolean | — | True após qualquer edição pós-envio. |
| `data_envio` | timestamp | — | Registrar a data e hora do primeiro envio. |
| `data_edicao` | timestamp (nullable) | — | Registrar a data e hora da última edição. |

### 3.11 `Avaliacao`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | — |
| `avaliador_id` | UUID FK | Usuario | — |
| `avaliado_id` | UUID FK | Usuario | Identifica o colaborador avaliado. |
| `periodo` | string | — | Ex: 2025-S1. Constraint única: (avaliado_id, periodo). Deve ser único em conjunto com avaliado_id, impedindo avaliações duplicadas no mesmo semestre. |
| `tipo_nota` | ENUM: NUMERICA, CONCEITUAL | — | Define semântica do campo nota. |
| `escala` | string | — | Ex: 1-5, A-E, Abaixo/Dentro/Acima. |
| `nota` | numeric | — | — |
| `observacao` | text | — | — |
| `status_avaliacao` | ENUM: RASCUNHO, FINALIZADA | — | Avaliações FINALIZADAS somente podem ser reabertas pelo RH autorizado, mediante justificativa. |
| `data_criacao` | timestamp | — | — |
| `data_finalizacao` | timestamp (nullable) | — | Preservar o registro das finalizações anteriores no histórico, mesmo após reabertura. |

### 3.12 `Reuniao`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | — |
| `solicitante_id` | UUID FK | Usuario | — |
| `participante_id` | UUID FK | Usuario | Qualquer usuário pode ser participante. |
| `data_hora` | timestamp | — | A nova reunião recebe a data e o horário reagendados. |
| `teams_meeting_id` | string (nullable) | — | ID retornado pelo Graph API. Necessário para cancelar pelo Teams. |
| `teams_join_url` | string (nullable) | — | Link de acesso à reunião. Preenchido após aprovação. |
| `agenda` | text (nullable) | — | — |
| `reuniao_origem_id` | UUID FK | Reuniao | FK opcional para Reuniao.id, identificando a reunião anterior. |
| `notas` | text (nullable) | — | Preenchido após a reunião. |
| `status_reuniao` | ENUM: SOLICITADA, APROVADA, REALIZADA, CANCELADA | — | A aprovação inicia a integração. A reunião original passa para REAGENDADA, preservando seus dados. Falhas devem ser tratadas sem duplicar eventos ou reuniões. |
| `data_criacao` | timestamp | — | — |

### 3.13 `Competencia`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador único da competência. |
| `nome` | string | — | Nome obrigatório. Não permitir competências duplicadas, ignorando diferenças entre maiúsculas e minúsculas. |
| `descricao` | text | — | Descrição opcional do conhecimento ou habilidade. |
| `categoria` | string | — | Categoria definida e gerenciada pelo RH. Deve seguir uma nomenclatura padronizada. |
| `ativo` | boolean | — | Permite inativar competências sem excluir registros históricos. Competências inativas não podem ser vinculadas a novos PDIs. |
| `data_criacao` | timestamp | — | Registra automaticamente a data e hora de criação. Imutável após o cadastro. |

### 3.14 `Noticia`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | — |
| `titulo` | string | — | — |
| `conteudo` | text | — | — |
| `autor_id` | UUID FK | Usuario | — |
| `publicado` | boolean | — | False = rascunho (apenas RH vê). True = visível a todos. Notícias publicadas podem ser editadas exclusivamente pelo RH autorizado. |
| `data_publicacao` | timestamp (nullable) | — | Preenchido ao publicar. Null enquanto rascunho. |
| `data_atualizacao` | timestamp | — | Registrar a data e hora da última alteração. |

### 3.15 `Aprovacao`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Proposta: somente se compartilhamento por PDI for confirmado. |
| `pdi_id` | UUID FK NOT NULL | PDI | Proposta. |
| `usuario_id` | UUID FK NOT NULL | Usuario | Proposta; usuário que recebeu acesso. |
| `permissao` | ENUM: VISUALIZAR, COMENTAR, EDITAR, ADMINISTRAR | — | Proposta; permissões exatas ainda precisam ser validadas pelo negócio. |
| `concedido_por` | UUID FK NOT NULL | Usuario | Proposta; registra quem concedeu. |
| `data_concessao` | timestamp | — | Proposta. |
| `data_revogacao` | timestamp nullable | — | Proposta; null = acesso vigente. |

### 3.16 `PdiEvento`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Proposta: necessária se auditoria/histórico for requisito. |
| `pdi_id` | UUID FK NOT NULL | PDI | Identifica o PDI relacionado ao evento. |
| `usuario_id` | UUID FK NOT NULL | Usuario | Identifica quem realizou a operação. |
| `tipo_evento` | ENUM | — | Registrar rejeições, alterações relevantes e reenvios para aprovação, preservando o histórico. |
| `tipo_status` | ENUM nullable | — | Identifica se a transição corresponde à aprovação ou à execução. |
| `status_anterior` | string (nullable) | — | Valor anterior do status alterado. |
| `status_novo` | string (nullable) | — | Valor resultante da alteração. |
| `data_evento` | timestamp | — | Registra automaticamente a data e hora do evento. |
| `descricao` | text nullable | — | Registra justificativa ou detalhes relevantes da movimentação. |
| `versao_atual` | integer | — | Identifica a versão de trabalho mais recente do PDI. Inicia em 1 e aumenta quando uma alteração relevante gera nova versão. |
| `versao_aprovada` | integer nullable | — | Identifica a última versão aprovada pelo gestor. Nulo enquanto nenhuma versão tiver sido aprovada. |
| `snapshot_pdi` | JSON | — | Preserva uma cópia completa e imutável dos dados de negócio da versão do PDI, incluindo os componentes relacionados que façam parte do plano. Versões aprovadas permanecem disponíveis independentemente de alterações posteriores. |

### 3.17 `KeyResultHistorico`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador do registro |
| `key_result_id` | UUID FK NOT NULL | KeyResult | Resultado-chave atualizado |
| `usuario_id` | UUID FK NOT NULL | Usuario | Responsável pela atualização |
| `tipo_evento` | ENUM | — | CRIACAO, EDICAO, ATUALIZACAO_PROGRESSO. |
| `valor_anterior` | JSON nullable | — | Valor antes da atualização |
| `valor_novo` | JSON | — | Valor após a atualização |
| `data_registro` | timestamp | — | Momento da alteração |
| `observacao` | text nullable | — | Justificativa ou evidência informada |

### 3.18 `UsuarioCompetencia`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador único |
| `usuario_id` | UUID FK NOT NULL | Usuario | Colaborador |
| `competencia_id` | UUID FK NOT NULL | Competencia | Competência cadastrada |
| `data_aquisicao` | timestamp | — | Data do registro |
| `origem` | ENUM: CAPACITACAO, CADASTRO_INICIAL | — | Origem do registro |

### 3.19 `CapacitacaoCompetencia`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador único |
| `capacitacao_id` | UUID FK NOT NULL | Capacitacao | Capacitação relacionada |
| `competencia_id` | UUID FK NOT NULL | Competencia | Competência desenvolvida |

### 3.20 `Capacitacao`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador único |
| `titulo` | varchar(200) | — | Nome da capacitação |
| `descricao` | text nullable | — | Descrição do conteúdo |
| `tipo` | ENUM: CURSO, TREINAMENTO, LIVRO, WORKSHOP, OUTRO | — | Classificação do conteúdo |
| `fornecedor` | varchar(150) nullable | — | Instituição ou plataforma responsável |
| `carga_horaria` | decimal nullable | — | Duração em horas, quando aplicável |
| `url_acesso` | text nullable | — | Link para acessar o conteúdo |
| `ativo` | boolean | — | Define se está disponível no catálogo |
| `data_criacao` | timestamp | — | Data do cadastro |

### 3.21 `UsuarioCapacitacao`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador único |
| `usuario_id` | UUID FK NOT NULL | Usuario | Colaborador |
| `capacitacao_id` | UUID FK NOT NULL | Capacitacao | Capacitação realizada |
| `status` | ENUM: NÃO_INICIADA, EM_ANDAMENTO, CONCLUIDA | — | Situação da capacitação |
| `data_inicio` | date nullable | — | Início da realização |
| `data_conclusao` | timestamp nullable | — | Preenchida na conclusão |

### 3.22 `AcaoPdiCapacitacao`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | UUID PK | — | Identificador único |
| `acao_pdi_id` | UUID FK NOT NULL | AcaoPDI | Ação que utiliza a capacitação |
| `capacitacao_id` | UUID FK NOT NULL | Capacitacao | Capacitação vinculada |
| `obrigatoria` | boolean | — | Define se a conclusão é requisito da ação |

### 3.23 `FeedbackHistorico`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | BIGINT (PK) | — | Identificador do histórico |
| `feedback_id` | BIGINT (FK) | Feedback | Referência ao feedback original |
| `editado_por_id` | BIGINT (FK) | Usuario | Responsável pela edição |
| `snapshot_anterior` | JSON | — | Cópia dos dados anteriores à alteração |
| `data_edicao` | timestamp | — | Data e hora da edição |

### 3.24 `AvaliacaoHistorico`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | BIGINT (PK) | — | Identificador do histórico |
| `avaliacao_id` | BIGINT (FK) | Avaliacao | Avaliação relacionada |
| `tipo_evento` | ENUM | — | REABERTURA, EDICAO, FINALIZACAO |
| `snapshot_anterior` | JSON | — | Estado da avaliação antes da alteração |
| `alterado_por_id` | BIGINT (FK) | Usuario | Responsável pela alteração |
| `justificativa` | text | — | Obrigatória quando tipo_evento = REABERTURA |
| `data_evento` | timestamp | — | Data e hora do evento |

### 3.25 `NoticiaHistorico`

| Campo | Tipo / ENUM (origem) | FK para (origem) | Regra de negócio (origem) |
|---|---|---|---|
| `id` | BIGINT (PK) | — | Identificador do histórico |
| `noticia_id` | BIGINT (FK) | Noticia | Referência à notícia original |
| `editado_por_id` | BIGINT (FK) | Usuario | Usuário responsável pela edição |
| `snapshot_anterior` | JSON | — | Cópia completa da notícia antes da alteração |
| `data_edicao` | timestamp | — | Data e hora da edição |

## 4. Relacionamentos e cardinalidades

| Origem | Cardinalidade | Destino | Campo FK | Regra |
|---|---|---|---|---|
| Usuario | 1:N | HistoricoEquipe | historicoequipe.usuario_id | Histórico de movimentações |
| Equipe | 1:N | HistoricoEquipe | historicoequipe.equipe_origem_id | Equipe à qual o colaborador pertenceu |
| Equipe | 1:N | HistoricoEquipe | historicoequipe.equipe_destino_id | Equipe para o qual o colaborador foi transferido. |
| Usuario | N:1 | Cargo | usuario.cargo_id | Atualiza cargo_id na troca. Não cria novo usuário. |
| Usuario | N:1 (auto) | Usuario (gestor) | usuario.gestor_id | Auto-referência. Nullable para colaboradores sem gestor. |
| Usuario | N:1 | Equipe | usuario.equipe_atual_id | Tabela de junção com data_entrada e data_saida. |
| Equipe | N:1 | Usuario (líder) | equipe.lider_id | — |
| Usuario | 1:N | PDI (dono) | pdi.usuario_id | — |
| Usuario | 1:N | PDI (aprova) | pdi.decidido_por | Apenas gestor direto aprova. |
| PDI | 1:N | AcaoPDI | acaopdi.pdi_id | — |
| Usuario | 1:N | OKR (dono) | okr.usuario_id | — |
| Usuario | 1:N | OKR (aprova) | okr.aprovado_por | — |
| OKR | 1:N | Equipe (opcional) | okr.equipe_id | Nullable. Nulo = OKR pessoal. Preenchido = OKR de equipe. |
| OKR | 1:N | KeyResult | keyresult.okr_id | — |
| Usuario | 1:N | Feedback (autor) | feedback.autor_id | — |
| Usuario | 1:N | Feedback (destinat.) | feedback.destinatario_id | — |
| Usuario | 1:N | Reuniao (colaborador) | reuniao.solicitante_id | Colaborador vinculado à reunião 1:1. |
| Usuario | 1:N | Reuniao (gestor) | reuniao.participante_id | Gestor responsável pela reunião 1:1. |
| Usuario | 1:N | Avaliacao (avaliador) | avaliacao.avaliador_id | — |
| Usuario | 1:N | Avaliacao (avaliado) | avaliacao.avaliado_id | — |
| Usuario | 1:N | Noticia (autor) | noticia.autor_id | — |
| PDI | 1:N | PdiEvento | pdievento.pdi_id | Eventos e versões do PDI |
| Usuario | 1:N | PdiEvento | pdievento.usuario_id | Autor do evento |
| PDI | 1:N | OKR | okr.pdi_id | Vínculo opcional, somente para OKR pessoal |
| OKR | 1:N | OkrEvento | okrevento.okr_id | Histórico de alterações do objetivo |
| Usuario | 1:N | OkrEvento | okrevento.usuario_id | Autor da alteração |
| KeyResult | 1:N | KeyResultHistorico | keyresulthistorico.key_result_id | Histórico das metas e progresso |
| Usuario | 1:N | KeyResultHistorico | keyresulthistorico.usuario_id | Autor da alteração |
| Usuario | 1:N | UsuarioCompetencia | usuariocompetencia.usuario_id | — |
| Competencia | 1:N | UsuarioCompetencia | usuariocompetencia.competencia_id | — |
| Usuario | 1:N | UsuarioCapacitacao | usuariocapacitacao.usuario_id | — |
| Capacitacao | 1:N | UsuarioCapacitacao | usuariocapacitacao.capacitacao_id | — |
| Capacitacao | 1:N | CapacitacaoCompetencia | capacitacaocompetencia.capacitacao_id | — |
| Competencia | 1:N | CapacitacaoCompetencia | capacitacaocompetencia.competencia_id | — |
| AcaoPDI | 1:N | AcaoPdiCapacitacao | acaopdicapacitacao.acao_pdi_id | — |
| Capacitacao | 1:N | AcaoPdiCapacitacao | acaopdicapacitacao.capacitacao_id | — |
| Reuniao | 1:0...1 (auto) | Reuniao (reagendada) | reuniao.reuniao_origem_id | Cada novo agendamento aponta para o imediatamente anterior; anterior REAGENDADA; preservar cadeia, sem ciclos ou sucessores duplicados. |
| Feedback | 1:N | FeedbackHistorico | feedbackhistorico.feedback_id | Versões anteriores em JSON; histórico imutável e anonimato protegido. |
| Usuario | 1:N | FeedbackHistorico (editor) | feedbackhistorico.editado_por_id | Identifica editor; autorização de acesso respeita anonimato. |
| Avaliacao | 1:N | AvaliacaoHistorico | avaliacaohistorico.avaliacao_id | Versões e eventos de reabertura/edição/finalização; justificativa obrigatória na reabertura. |
| Usuario | 1:N | AvaliacaoHistorico (editor) | avaliacaohistorico.alterado_por_id | RH autorizado é obrigatório para reabrir avaliação finalizada. |
| Noticia | 1:N | NoticiaHistorico | noticiahistorico.noticia_id | Versões anteriores em JSON, imutáveis. |
| Usuario | 1:N | NoticiaHistorico (editor) | noticiahistorico.editado_por_id | Somente RH autorizado edita notícia publicada. |
| Usuario | 1:N | HistoricoEquipe (responsável) | historicoequipe.realizado_por_id | Identifica responsável pela transferência; NULL apenas em eventos automáticos/importações. |

## 5. Acesso e permissões

### 5.1 Perfis

| Perfil | Role Spring Security | Descrição | Exemplos de ação |
|---|---|---|---|
| Colaborador | ROLE_EMPLOYEE | Usuário padrão. Consulta e gerencia informações próprias conforme a Matriz de Permissões; cria PDIs e OKRs pessoais, registra feedbacks, acompanha competências e solicita reuniões 1:1. | Criar e acompanhar PDI próprio; editar OKR pessoal com histórico; atualizar KeyResults próprios; enviar feedback SBI; solicitar 1:1; consultar notícias publicadas. |
| Gestor | ROLE_MANAGER | Responsável por uma equipe. Consulta dados de sua equipe conforme o vínculo; aprova PDIs dos subordinados diretos, atua nos OKRs conforme a hierarquia e conduz reuniões 1:1 e avaliações. | Aprovar ou rejeitar PDI de subordinado direto; criar OKR de equipe; aprovar OKR pessoal de subordinado ou OKR de equipe quando for superior do líder; agendar 1:1; avaliar equipe. |
| RH / ADM | ROLE_HR_ADMIN | Perfil administrativo para gestão de usuários, cargos, competências, avaliações e notícias. Algumas ações sensíveis exigem autorização específica de RH ou auditoria, além da role geral. | Gerenciar usuários e cargos (exclusão lógica); aprovar PDI/OKR quando autorizado; reabrir avaliação finalizada com justificativa; publicar e editar notícias com histórico; auditar autoria anônima somente com autorização expressa. |

### 5.2 Matriz de ações

| Entidade | Ação | Colaborador | Gestor | RH / ADM | Condição / escopo |
|---|---|---|---|---|---|
| Usuario | Visualizar | Próprio perfil | Integrantes da equipe | Todos | Respeitar acesso aos dados pessoais |
| Usuario | Criar / editar / inativar | — | — | Permitido | Inativação lógica; não excluir fisicamente |
| Cargo | Visualizar | Permitido | Permitido | Permitido | Catálogo de cargos |
| Cargo | Criar / editar / inativar | — | — | Permitido | Somente exclusão lógica; cargo inativo não recebe novas atribuições (RN022) |
| Equipe | Visualizar | Permitido | Própria equipe | Permitido | Escopo de visualização por perfil |
| Equipe | Criar / editar / inativar | — | Criar (conforme matriz original) | Permitido | Demais autorizações de gestão a validar |
| HistoricoEquipe | Consultar histórico | — | Gerencia membros (escopo da equipe) | Gerencia todos | Preservar histórico de movimentações |
| HistoricoEquipe | Transferir colaborador | — | Permitido (escopo da equipe) | Permitido | Atualização atômica; proibir transferência para mesma equipe (RN024) |
| PDI | Criar / editar | Próprios PDIs | Editar conforme permissão original | Visualizar todos | Múltiplos PDIs permitidos; alterações registradas |
| PDI | Aprovar / rejeitar | — | Gestor direto | RH autorizado | Rejeição permite correção e reenvio; preservar histórico |
| PDI | Visualizar | Próprios | Equipe | Todos | Acesso conforme vínculo |
| AcaoPDI | Criar / editar / excluir | Próprias ações | Visualizar | Visualizar | Status independente; conclusão de todas as ações conclui PDI |
| OKR | Criar / editar | Próprio OKR | OKR de equipe | Visualizar todos | OKRs pessoais e de equipe; edição com histórico |
| OKR | Aprovar / rejeitar | — | Gestor direto (pessoal); superior do líder (equipe) | RH autorizado | Critério depende do proprietário do OKR |
| KeyResult | Atualizar progresso | Próprios resultados | Visualizar | Visualizar | Validar conforme tipo_metrica |
| Feedback | Criar / visualizar | Enviar e receber | Enviar e receber | Enviar e receber | SBI obrigatório; anonimato protege identidade |
| Feedback | Editar | Autor | Autor | Autor | Preservar versões em FeedbackHistorico |
| Feedback | Excluir | — | Equipe (matriz original) | Todos (matriz original) | Conferir política de exclusão antes da implementação |
| FeedbackHistorico | Consultar versões | Conforme acesso ao feedback | Conforme acesso ao feedback | Conforme acesso ao feedback | Autor anônimo visível apenas à auditoria expressamente autorizada |
| FeedbackHistorico | Alterar / excluir | — | — | — | Histórico imutável; gravação automática em edição |
| Reuniao | Solicitar / agendar | Solicitar | Aprovar / agendar | Agendar / cancelar | Reuniões 1:1 entre gestor e colaborador |
| Reuniao | Aprovar / reagendar | — | Permitido | Permitido | Reagendamento cria novo registro; integração Teams após aprovação |
| Avaliacao | Criar / avaliar | — | Equipe | Geral | Uma avaliação por colaborador/semestre |
| Avaliacao | Visualizar | Própria | Equipe | Todas | Preservar escopo de acesso |
| Avaliacao | Editar antes da finalização | — | Permitido (conforme vínculo) | Permitido | Após finalização, somente reabertura excepcional |
| Avaliacao | Reabrir finalizada | — | — | RH autorizado | Justificativa obrigatória; registrar AvaliacaoHistorico |
| AvaliacaoHistorico | Consultar versões | — | Conforme acesso à avaliação | RH autorizado | Histórico imutável de reabertura, edição e finalização |
| Competencia | Visualizar | Permitido | Permitido | Permitido | Catálogo de competências |
| Competencia | Criar / editar / inativar | — | — | Permitido | Categorias administradas pelo RH |
| Noticia | Visualizar | Publicadas | Publicadas | Publicadas e rascunhos | Notícias publicadas visíveis a usuários ativos (RN020) |
| Noticia | Criar / editar / publicar | — | — | RH autorizado | Pode editar após publicação, com histórico (RN021) |
| NoticiaHistorico | Consultar versões | — | — | RH autorizado | Versões anteriores em snapshot JSON |
| NoticiaHistorico | Alterar / excluir | — | — | — | Histórico imutável |

## 6. Pontos a resolver antes de gerar migrações SQL

Os itens abaixo são **divergências ou ambiguidades detectadas**, não alterações feitas na planilha:

1. **PDI e aprovação por RH:** o relacionamento `Usuario → PDI (aprova)` ainda diz “Apenas gestor direto aprova”, mas a decisão RN005 e a matriz incluem RH autorizado. Uniformizar a redação e o nome do campo (`decidido_por`/`aprovado_por`) antes do DDL.
2. **Equipe atual:** o relacionamento `Usuario → Equipe` usa `usuario.equipe_atual_id`, mas seu texto diz “Tabela de junção com data_entrada e data_saida”, embora a decisão vigente use `HistoricoEquipe` para o histórico. Não gerar tabela de junção redundante.
3. **Referências FK não normalizadas:** há valores como `Azure AD`, `Usuário` e `ENUM: JUNIOR, PLENO, SENIOR, NAO_APLICAVEL` na coluna `FK para`; distinguir integração externa, tipo e chave estrangeira real.
4. **Entidade `Aprovacao`:** aparece no catálogo; sua manutenção ou absorção por campos de aprovação de PDI/OKR precisa de decisão para evitar duplicidade de estados.
5. **SGBD e dialeto:** o arquivo não fixa o banco definitivo nem a estratégia SQL de ENUM/JSON/UUID. Selecionar banco e migração (por exemplo Flyway) antes de produzir scripts executáveis.
6. **Permissões em aberto:** a matriz contém expressões como “conforme permissão original”, “a validar” e permissões de exclusão de feedback a conferir; não interpretar isso como autorização irrestrita.
7. **Consistência dos campos:** validar que cada FK da aba Relacionamentos corresponde exatamente a um campo declarado na aba Entidades, incluindo nomes com prefixos/variações; definir nulabilidade, índices e política `ON DELETE` não explicitados.

## 7. Plano de implementação incremental para Codex

1. **Leitura e diagnóstico:** Codex deve ler este documento, listar inconsistências e sugerir decisões mínimas, sem criar tabelas ainda.
2. **Infraestrutura SQL:** definir banco, conexão local, convenções de nome, UUID, JSON, timestamps, migrações e ambiente de teste.
3. **Módulo Core:** migrar `Cargo`, `Equipe`, `Usuario`, `HistoricoEquipe` e validar integridade e transferência atômica.
4. **PDI/OKR:** implementar entidades, ações, resultados, aprovações e eventos com testes de regras e permissões.
5. **Feedback, Avaliação, Notícias e Reuniões:** implementar histórico imutável, anonimato, reabertura autorizada, reagendamento e integração Teams isolada.
6. **Capacitações/Competências:** implementar catálogos e associações com progresso de ações, respeitando a conclusão autodeclarada.
7. **Revisão:** testes de migração, FK, UNIQUE, validações, segurança por role e escopo, e regressão das regras.

## 8. Prompt inicial para o Codex

```text
Leia docs/DATABASE.md como fonte da modelagem TalentSynk. Não gere código nem migrações ainda. Faça um diagnóstico dos nomes de campos, chaves estrangeiras, cardinalidades, tipos, regras e permissões. Separe: (1) decisões já explícitas no documento, (2) inconsistências entre abas, (3) definições técnicas ainda pendentes. Não invente regras. Proponha a sequência de migrações começando por Core e aguarde minha aprovação antes de modificar o projeto.
```
