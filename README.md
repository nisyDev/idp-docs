# TalentSynk

> Uma plataforma de desenvolvimento profissional centrada no Plano de Desenvolvimento Individual (PDI), projetada para transformar o acompanhamento de carreira em um processo contínuo, mensurável e orientado por evolução.

---

## Sobre o projeto

O TalentSynk nasceu da observação de um problema recorrente em plataformas de gestão de pessoas: o Plano de Desenvolvimento Individual (PDI) costuma ser tratado como um documento estático, preenchido durante ciclos de avaliação e raramente consultado novamente.

Na prática, isso faz com que o desenvolvimento profissional deixe de ser um processo contínuo e passe a ser apenas uma atividade burocrática.

O objetivo deste projeto é propor uma abordagem diferente.

Em vez de centralizar a experiência em avaliações periódicas, o TalentSynk posiciona o PDI como o núcleo da jornada de desenvolvimento do colaborador, permitindo acompanhar objetivos, competências, ações, feedbacks e evolução ao longo do tempo.

---

# Visão do Produto

O projeto foi concebido para responder continuamente três perguntas fundamentais:

- **Onde estou?**
- **Onde quero chegar?**
- **O que preciso fazer para chegar lá?**

Todas as funcionalidades da plataforma existem para apoiar essas três respostas.

O Plano de Desenvolvimento Individual deixa de ser um registro isolado e passa a representar uma linha do tempo da evolução profissional do colaborador.

---

# Objetivos

O TalentSynk busca:

- incentivar o acompanhamento contínuo do desenvolvimento profissional;
- tornar a evolução do colaborador visual e mensurável;
- aproximar gestores e colaboradores através de acompanhamento frequente;
- consolidar competências, objetivos e ações em uma única plataforma;
- fornecer informações estratégicas para lideranças e RH.

---

# Módulos da Plataforma

A solução foi organizada em módulos independentes, cada um responsável por uma etapa da jornada de desenvolvimento.

| Módulo | Objetivo |
|---------|----------|
| Perfil | Centralizar informações profissionais do colaborador |
| Competências | Gerenciar Hard Skills e Soft Skills |
| Desenvolvimento | Registrar cursos, treinamentos e ações de aprendizagem |
| Plano de Desenvolvimento Individual | Gerenciar objetivos, ações, prazos e progresso |
| Feedback | Registrar feedbacks entre colaboradores e gestores |
| Reuniões 1:1 | Acompanhar alinhamentos e evolução dos PDIs |
| Organograma | Visualizar a estrutura organizacional |
| Sugestões | Incentivar participação e melhoria contínua |

---

# Público-alvo

A plataforma foi concebida para atender três perfis principais.

## Colaborador

Responsável por acompanhar sua própria evolução profissional, registrar ações de desenvolvimento e monitorar seus objetivos.

## Gestor

Responsável por acompanhar o desenvolvimento da equipe, apoiar PDIs e registrar feedbacks.

## Recursos Humanos

Responsável por acompanhar indicadores organizacionais, competências e iniciativas de desenvolvimento.

---

# Arquitetura

A solução está sendo construída seguindo uma arquitetura em camadas, priorizando separação de responsabilidades, baixo acoplamento e facilidade de evolução.

Fluxo simplificado da aplicação:

```text
Frontend (React)

        ↓

REST API

        ↓

Spring Boot

        ↓

Camada de Serviços

        ↓

Repositórios

        ↓

Banco de Dados
```

A documentação completa da arquitetura encontra-se em `docs/ARCHITECTURE.md`.

---

# Tecnologias

## Front-end

- React
- TypeScript
- Vite

## Back-end

- Java 21
- Spring Boot
- Spring Security
- Spring Data JPA

## Banco de Dados

- PostgreSQL *(planejado)*

---

# Status do Projeto

O projeto encontra-se em desenvolvimento.

Atualmente as principais atividades concentram-se em:

- definição da arquitetura da solução;
- modelagem de dados;
- definição das regras de negócio;
- implementação das APIs principais;
- construção da autenticação e autorização.

As próximas etapas incluem:

- desenvolvimento das interfaces;
- integração entre Front-end e Back-end;
- persistência dos dados;
- evolução dos módulos da plataforma.

---

# Roadmap

Entre as funcionalidades previstas para evolução da plataforma estão:

- acompanhamento visual do progresso dos PDIs;
- histórico de evolução das competências;
- dashboards de desenvolvimento;
- indicadores organizacionais;
- gestão de trilhas de aprendizagem;
- notificações e lembretes;
- assistente baseado em Inteligência Artificial para apoiar o desenvolvimento profissional.

---

# Documentação

A documentação completa do projeto está organizada em documentos específicos.

| Documento | Descrição |
|------------|-----------|
| BUSINESS.md | Visão de negócio |
| ARCHITECTURE.md | Arquitetura da solução |
| DATABASE.md | Modelagem de dados |
| API.md | Documentação das APIs |
| PROJECT_STATE.md | Estado atual do desenvolvimento |
| ROADMAP.md | Evolução planejada |
| DECISIONS.md | Decisões arquiteturais |

---

# Objetivos de Aprendizado

Além da proposta de negócio, este projeto também tem como objetivo aprofundar conhecimentos em:

- arquitetura de software;
- desenvolvimento Full Stack;
- modelagem de domínio;
- APIs REST;
- autenticação e autorização;
- boas práticas de engenharia de software;
- documentação técnica;
- escalabilidade e manutenção de aplicações.

---

# Licença

Este projeto está sendo desenvolvido para fins de estudo, evolução técnica e demonstração de arquitetura e engenharia de software.
