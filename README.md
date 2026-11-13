# TalentSynk

> Uma plataforma de desenvolvimento profissional centrada no Plano de Desenvolvimento Individual (PDI), projetada para transformar o acompanhamento de carreira em um processo contínuo, mensurável e orientado por evolução.

## Sobre o projeto

O TalentSynk nasceu da observação de um problema recorrente em plataformas de gestão de pessoas: o Plano de Desenvolvimento Individual costuma ser tratado como um documento estático, preenchido durante ciclos de avaliação e raramente consultado novamente. Isso transforma o desenvolvimento profissional em uma atividade burocrática.

A proposta é posicionar o PDI como núcleo de uma jornada contínua de objetivos, competências, ações, feedbacks e evolução. O produto está em desenvolvimento incremental para portfólio e futuro beta; não está finalizado nem possui deploy público realizado.

## Visão do produto

O projeto foi concebido para responder continuamente três perguntas:

- **Onde estou?**
- **Onde quero chegar?**
- **O que preciso fazer para chegar lá?**

O PDI deve representar uma linha do tempo da evolução profissional. Essa visão orienta o produto; não significa que todo o ciclo funcional esteja implementado.

## Objetivos

- Incentivar o acompanhamento contínuo do desenvolvimento profissional.
- Tornar a evolução do colaborador visual e mensurável.
- Aproximar gestores e colaboradores por meio de acompanhamento frequente.
- Consolidar competências, objetivos e ações em uma única plataforma.
- Fornecer informações estratégicas para lideranças e RH.

## Público-alvo

| Público | Papel na visão de negócio |
|---|---|
| Colaborador | Acompanhar a própria evolução, registrar ações de desenvolvimento e monitorar objetivos. |
| Gestor | Apoiar o desenvolvimento da equipe, acompanhar PDIs e registrar feedbacks. |
| RH/Administrador | Acompanhar indicadores organizacionais, competências e iniciativas de desenvolvimento. |

Os perfis internos são COLABORADOR, GESTOR e RH_ADMIN. A matriz completa de permissões ainda é parcial. Atualmente, o acesso a rascunhos e ações permanece restrito ao próprio titular, inclusive para Gestor e RH_ADMIN; a interface não concede acesso à equipe ou acesso global ao RH.

## Organização dos repositórios

| Componente | Repositório oficial | Referência utilizada nesta atualização |
|---|---|---|
| Front-end | [idp-frontend](https://github.com/nisyDev/idp-frontend) | [01cd6d3](https://github.com/nisyDev/idp-frontend/commit/01cd6d322c1efb4b843756f13471c196851ecf42) — master |
| Back-end | [idp](https://github.com/nisyDev/idp) | [38573f9](https://github.com/nisyDev/idp/commit/38573f9ab19b9181a3adf516533bbd8849b80def) — master |
| Documentação | [idp-docs](https://github.com/nisyDev/idp-docs) | Base histórica da consolidação: [7afc5e2](https://github.com/nisyDev/idp-docs/commit/7afc5e2a5c521fde7bd12efc1aa1b4ddc87f006c); documentação central organizada nas fases G5B.1–G5B.3. |

Front-end e back-end foram publicados no GitHub nas revisões indicadas. Publicação do código não significa deploy da aplicação. Alterações locais ainda não publicadas não compõem esta referência.

## Arquitetura e tecnologias

A aplicação utiliza camadas de interface, controllers REST, serviços, repositories e persistência via ORM, com separação de responsabilidades.

Fluxo conceitual preservado da documentação original:

<img width="350" height="600" alt="architecture-flow" src="https://github.com/user-attachments/assets/7c316456-07e6-4743-969c-ea1cfa500964" />

| Camada | Stack atual |
|---|---|
| Front-end | React, TypeScript e Vite |
| Back-end | Java 21, Spring Boot 3.3.5, Spring MVC e Spring Security |
| Persistência | Spring Data JPA, Hibernate e PostgreSQL; Neon nos ambientes de teste informados |
| Migrations | Flyway 11.14.1, migrations V1–V3 |
| Build | Maven Wrapper 3.9.16 no back-end; TypeScript e Vite no front-end |
| Testes locais | JUnit, Mockito e MockMvc |

Fontes: [dependências do back-end](https://github.com/nisyDev/idp/blob/38573f9ab19b9181a3adf516533bbd8849b80def/pom.xml), [dependências do front-end](https://github.com/nisyDev/idp-frontend/blob/01cd6d322c1efb4b843756f13471c196851ecf42/package.json) e [lockfile do front-end](https://github.com/nisyDev/idp-frontend/blob/01cd6d322c1efb4b843756f13471c196851ecf42/package-lock.json).

No desenvolvimento local, o front-end usa chamadas relativas a /api, com proxy Vite para 127.0.0.1:8081. O perfil auth-local mantém o servidor em loopback, Flyway automático desabilitado, Hibernate sem geração de schema e inicialização SQL desabilitada. Não há instrução de executar migrations ou expor o servidor por este documento.

## Funcionalidades e estado atual

| Recorte | Estado e alcance |
|---|---|
| Autenticação local | Implementada e integrada: login por e-mail/senha, BCrypt, sessão HTTP, CSRF, identidade atual e logout real. Validação manual informada pela responsável e testes HTTP isolados aprovados. |
| Rascunhos privados de PDI | Consulta, criação e edição integradas ao front-end e validadas manualmente. Campos e identidade vêm dos contratos reais; sem dados fictícios na listagem de PDI. |
| Ações de PDI | Cadastro e consulta integrados e validados manualmente, com associação ao PDI e à versão. |
| Edição de ações | Implementada somente no back-end: PUT /api/pdis/{pdiId}/acoes/{acaoId}, atualização dos cinco campos e evento ACAO_EDITADA. Cobertura isolada aprovada; interface de edição e validação PostgreSQL específica pendentes. |
| Catálogo de cargos | GET /api/cargos implementado no back-end para os três perfis, somente cargos ativos; testes isolados e consulta integrada histórica com limitações. |
| Dashboard e organograma | Interfaces demonstrativas, com dados ou alterações locais; não constituem integrações funcionais de persistência. |
| Capacitação e sugestões | Interfaces demonstrativas, sem fluxo real integrado correspondente. A indicação visual de IA não comprova integração de IA. |
| Perfil e reuniões | Parciais: parte da identidade vem do login real; outros dados e interações são demonstrativos. Reuniões não estão integradas ao Microsoft Teams. |
| Submissão, aprovação, rejeição, execução e conclusão do PDI | Pendentes; entidades e estados existentes não equivalem ao ciclo implementado. |
| Revisão de versões aprovadas e demais operações avançadas | Pendentes; decisões de negócio e privacidade devem ser preservadas. |
| Microsoft Entra ID e Microsoft Teams | Integrações futuras. A opção corporativa no login permanece indisponível; não há SSO nem integração real de reuniões. |
| Deploy público e beta | Pendentes; sem declaração de prontidão do MVP, da matriz RBAC completa ou do beta. |

O tema claro/escuro acompanha a preferência do sistema e mantém alternância manual. A inspeção do código não substitui validação visual de todas as telas e resoluções.

### Módulos previstos na visão de produto

A tabela preserva os objetivos dos módulos originais. A disponibilidade atual é a da tabela anterior e do [estado do projeto](PROJECT_STATE.md).

| Módulo | Objetivo |
|---|---|
| Perfil | Centralizar informações profissionais do colaborador. |
| Competências | Gerenciar Hard Skills e Soft Skills. |
| Desenvolvimento | Registrar cursos, treinamentos e ações de aprendizagem. |
| Plano de Desenvolvimento Individual | Gerenciar objetivos, ações, prazos e progresso. |
| Feedback | Registrar feedbacks entre colaboradores e gestores. |
| Reuniões 1:1 | Acompanhar alinhamentos e evolução dos PDIs. |
| Organograma | Visualizar a estrutura organizacional. |
| Sugestões | Incentivar participação e melhoria contínua. |

## Segurança e regras preservadas

- O servidor define identidade e authorities. Seleções ou telas do cliente não concedem privilégios.
- Rascunhos nunca submetidos e conteúdo novo de revisões ainda não submetidas são privados do titular, conforme a Etapa 21B. RH_ADMIN não concede automaticamente consulta excepcional; esta depende de permissão expressa, justificativa e auditoria e não abrange esse conteúdo privado.
- A condição de gestor direto depende do vínculo hierárquico vigente.
- Login local utiliza sessão e CSRF; não utiliza HTTP Basic no perfil auth-local. Credenciais e tokens de sessão não são persistidos em localStorage.
- A opção de lembrar acesso atua sobre a persistência do cookie da sessão; não prolonga o timeout de inatividade nem garante sessão após reinício do servidor.
- Aprovação, execução e conclusão formal são conceitos distintos; o progresso não implica conclusão automática.

## Estado de validação

| Evidência | Resultado e limite |
|---|---|
| Build isolado do front-end — G3 | tsc -b && vite build aprovado no commit 01cd6d3. Não equivale a teste ponta a ponta. |
| Compilação isolada do back-end — G4C.1 | BUILD SUCCESS: 74 fontes de produção e 25 de teste, Microsoft OpenJDK 21.0.11 e Maven 3.9.16. Sem execução de JUnit ou banco. |
| Testes isolados do back-end — G4C.2 | 267 aprovados em 20 classes; zero falhas, erros ou ignorados. Sem Neon, PostgreSQL, H2 ou migrations. |
| Edição de ações | 46 testes de serviço e 36 HTTP aprovados, incluídos nos 267; não são resultados adicionais nem comprovação de persistência física. |
| Validação manual informada | Login/logout, criação/edição de PDI e cadastro/consulta de ações, com persistência após recarregamento nos incrementos avaliados. Não houve repetição na preparação documental. |
| Persistência PostgreSQL histórica | Etapas 184G.145, 184G.147 e 184G.153 registram execuções controladas distintas de PDI e credenciais locais. Não validam automaticamente a nova edição de ações. |
| Novos cenários PostgreSQL de edição de ações | Implementados, compilados e ainda não validados; execução suspensa. Rollback real dessa edição permanece pendente. |

Os resultados são delimitados por revisão, ambiente e tipo de teste. Não devem ser somados como uma execução única nem generalizados a toda a plataforma.

## Próximos incrementos e roadmap

1. Concluir a atualização documental e revisar sua publicação mediante autorização.
2. Retomar, mediante autorização específica, a validação PostgreSQL da edição de ações e sua integração na interface, mantendo titularidade, histórico e transações.
3. Evoluir o ciclo do PDI de forma incremental: submissão, aprovação/rejeição, execução, acompanhamento e conclusão, sem ampliar permissões implicitamente.
4. Tratar prontidão de segurança, operação e privacidade antes de deploy ou beta público.

Permanecem na visão futura: histórico de evolução das competências, dashboards de desenvolvimento, indicadores organizacionais, trilhas de aprendizagem, notificações e lembretes e assistência por Inteligência Artificial. Essas propostas não são funcionalidades concluídas.

## Documentação

| Documento | Situação |
|---|---|
| [BUSINESS.md](BUSINESS.md) | Visão de negócio existente, preservada integralmente. |
| [PROJECT_STATE.md](PROJECT_STATE.md) | Estado transversal da implementação, decisões, histórico, evidências e pendências. |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Componentes atuais, segurança, fluxos e evolução arquitetural planejada. |
| [DATABASE.md](DATABASE.md) | Modelo lógico versus esquema físico V1–V3, relações e limitações. |
| [API.md](API.md) | Contratos HTTP existentes, autorização e funcionalidades sem endpoint. |
| [DECISIONS.md](DECISIONS.md) | Decisões aprovadas e questões pendentes, sem confundir requisitos com entregas. |
| [ROADMAP.md](ROADMAP.md) | Sequência proposta de incrementos, sem prazos prometidos nem execução autorizada. |
| [CHANGELOG.md](CHANGELOG.md) | Marcos históricos e evidências, sem anunciar releases inexistentes. |
| [DATABASE.md técnico do back-end](https://github.com/nisyDev/idp/blob/38573f9ab19b9181a3adf516533bbd8849b80def/docs/DATABASE.md) | Registro técnico de origem; deve ser contextualizado com migrations e documento central. |
| [CORE_SCHEMA_VALIDATION.md do back-end](https://github.com/nisyDev/idp/blob/38573f9ab19b9181a3adf516533bbd8849b80def/docs/CORE_SCHEMA_VALIDATION.md) | Contrato operacional do teste CORE, mantido junto ao código. |

Os documentos centrais foram consolidados nas fases G5B.1–G5B.3, com referências aos commits do front-end e back-end indicados acima. Os links relativos pressupõem os arquivos na raiz de `idp-docs`. Migrations, código, testes e configurações executáveis permanecem no back-end. A centralização documental não altera regras de negócio nem autoriza execução de comandos, testes ou banco.

## Objetivos de aprendizado

Além da proposta de negócio, o projeto busca aprofundar conhecimentos em arquitetura de software, desenvolvimento Full Stack, modelagem de domínio, APIs REST, autenticação e autorização, boas práticas de engenharia, documentação técnica, escalabilidade e manutenção de aplicações.

## Finalidade do projeto

Este projeto está sendo desenvolvido para fins de estudo, evolução técnica e demonstração de arquitetura e engenharia de software. Uma licença de distribuição não é definida por este documento.
