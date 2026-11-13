# TalentSynk — Registro de decisões

> **Referência documental:** consolidação editorial das decisões vigentes; os identificadores `DEC-XX` abaixo são novos **índices de documentação**, não números de decisões históricas nem autorizações de desenvolvimento. Fontes: [BUSINESS.md](BUSINESS.md), [PROJECT_STATE.md](PROJECT_STATE.md), Etapa 21B citada no estado e código de referência registrado na auditoria G5A.

## 1. Como interpretar este registro

- **Aprovada/adotada:** decisão já descrita como vigente na documentação do projeto; sua implementação pode ser **parcial**.
- **Prevista:** escolha de produto ou arquitetura aprovada como direção, mas não concluída no código.
- **Pendente:** questão sem definição suficiente; não pode ser tratada como solução adotada.

Decisão de negócio **não equivale** a feature validada, permissão HTTP liberada, migration aplicada ou autorização para executar testes e deploy. Estados de implementação e evidências ficam em [PROJECT_STATE.md](PROJECT_STATE.md).

## 2. Decisões registradas

| Referência | Tema | Decisão/documento de origem | Situação da implementação |
|---|---|---|---|
| DEC-01 | Núcleo do produto | PDI é o eixo do desenvolvimento contínuo; outros módulos apoiam sua evolução. [BUSINESS.md](BUSINESS.md) | Visão adotada; plataforma completa não concluída |
| DEC-02 | Tecnologia principal | Java 21/Spring Boot no back-end, React/TypeScript/Vite no front-end, PostgreSQL e Flyway para versionamento. [PROJECT_STATE.md](PROJECT_STATE.md) | Parcialmente validada em builds e testes delimitados |
| DEC-03 | Repositórios | Código de front-end, back-end e documentação em três repositórios oficiais separados. | Os dois de código publicados; documentação ainda em atualização |
| DEC-04 | Privacidade dos rascunhos | Rascunhos nunca submetidos, assim como conteúdo novo ainda não submetido de revisões, são privados de seu titular. Etapa 21B. | Recorte de rascunho implementado; demais fluxos pendentes |
| DEC-05 | Limites de RH | `RH_ADMIN` não recebe acesso global automático aos rascunhos privados. Consulta excepcional exige permissão expressa, justificativa e auditoria; sem atingir esses conteúdos privados. Etapa 21B. | Regra para autorização futura; sem API excepcional liberada |
| DEC-06 | Gestor direto | Direitos do gestor em fluxos futuros dependem do vínculo hierárquico vigente, não apenas da authority. Etapa 21B. | Não significa que acesso à equipe já esteja implementado |
| DEC-07 | Identidade e papéis | `COLABORADOR`, `GESTOR` e `RH_ADMIN` são definidos pelo servidor; interface não concede direitos. | Auth-local e authorities implementados; RBAC geral parcial |
| DEC-08 | Credencial local | Usuário pode existir sem senha local; credencial separada opcional 1:1 por UUID compartilhado; BCrypt para senha. | V3 e autenticação local implementadas e validadas em recortes |
| DEC-09 | Histórico PDI | Registrar eventos e preservar versões, sem sobrescrever o histórico; mudanças de ação geram `ACAO_EDITADA` se houver diferença real. | Registro parcial implementado; API de leitura de histórico pendente |
| DEC-10 | Múltiplos PDIs | A modelagem admite múltiplos PDIs por colaborador. | Não equivale ao ciclo completo de aprovação e execução |
| DEC-11 | Aprovação e rejeição | Aprovação pelo gestor direto ou RH autorizado; rejeição justificada com possibilidade de reenvio; sem acesso implícito. | Requisito pendente de implementação |
| DEC-12 | Revisão de compromissos | Atualizações de compromissos já aprovados exigem novo processo de aprovação, preservando a versão aprovada durante análise. | Requisito futuro; não confundir com edição do rascunho inicial |
| DEC-13 | Conclusão | Progresso e conclusão formal são conceitos distintos; conclusão não deve ser inferida automaticamente. | Regras futuras não implementadas |
| DEC-14 | Integridade de edição | Rascunho inicial e ação elegíveis têm titularidade, estado, versão e datas controlados pelo servidor; auditoria e conteúdo modificados na transação aplicável. | Fluxos parciais implementados; novos testes PostgreSQL pendentes |
| DEC-15 | Cargos ativos | `GET /api/cargos` retorna apenas cargos `ativo=true` nos três perfis autorizados. | Código e testes isolados; validação real de exclusão de inativos pendente |
| DEC-16 | SSO corporativo | Planejar Microsoft Active Directory/Microsoft Entra ID como provedor de identidade futuro; não confundir com RBAC da aplicação. | Previsto, não implementado |
| DEC-17 | Reuniões corporativas | Microsoft Teams é integração futura para reuniões; links demonstrativos atuais não são integração real. | Previsto, não implementado |
| DEC-18 | Publicação | Preparar o produto para portfólio e beta público somente após critérios de segurança, privacidade, operação e validação. | GitHub publicado não equivale a beta/deploy |

## 3. Decisões que **não** foram tomadas por esta documentação

- Não há autorização para acesso administrativo irrestrito, exclusão física de PDI aprovado ou alteração de regras da Etapa 21B.
- Não foi definida uma arquitetura operacional de produção: HTTPS, domínio, sessão distribuída, backup, observabilidade e hospedagem dependem de projeto próprio.
- Não foi implementado SSO corporativo nem integração Teams, apesar de serem direções previstas.
- Não há definição formal de release versionada ou licença de distribuição neste registro.
- Nenhuma execução de testes PostgreSQL, aplicação de migration ou alteração de banco é autorizada.

## 4. Questões pendentes de decisão ou comprovação

| Tema | Pendência | Próxima evidência/decisão esperada |
|---|---|---|
| Edição de ações | Cinco testes PostgreSQL recentes pendentes; interface ainda não integrada | Execução com guarda e autorização própria; desenvolvimento visual em incremento separado |
| RBAC e privacidade | Matriz completa ainda não operacional | Revisão de contratos de acesso por caso de uso e testes positivos/negativos |
| Catálogo de cargos | Exclusão de inativos não demonstrada com conjunto real misto | Validação autorizada em ambiente controlado |
| Modelo de dados | Catálogo lógico não equivale às 13 tabelas V1–V3 | Comparação linha a linha com SQL/JPA antes de detalhamento físico |
| Dados demonstrativos | Procedência de alguns dados visuais não comprovada | Substituir por dados comprovadamente fictícios antes de exposição pública |
| Segurança do histórico | Possibilidade de credencial antiga em outro histórico Git | Inspeção privada e revogação/rotação se a credencial for real; não expor o valor |
| Publicação | Requisitos operacionais de beta e licença ainda não aprovados | Checklist de segurança, custos e operação e decisão expressa |

## 5. Governança documental

Mudanças futuras devem registrar: **decisão**, **motivo**, **fonte/etapa**, **efeitos técnicos**, **estado de implementação**, **evidência** e **data**, quando comprovada. Não reescrever decisões históricas para justificar comportamento implementado diferente; registrar divergência e resolução com autorização.

Ver [ROADMAP.md](ROADMAP.md), [ARCHITECTURE.md](ARCHITECTURE.md) e [CHANGELOG.md](CHANGELOG.md).
