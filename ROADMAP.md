# TalentSynk — Roadmap de evolução

> **Estado:** planejamento editorial, **sem prazos prometidos**. As fases descrevem uma ordem recomendada, não uma autorização para implementar todos os itens. O estado vigente e suas evidências permanecem em [PROJECT_STATE.md](PROJECT_STATE.md).

## 1. Ponto de partida

**Já existe no código publicado:** autenticação local com sessão e CSRF; rascunhos próprios de PDI (listar, consultar, criar, editar); cadastrar e consultar ações; consulta de cargos ativos; PUT de edição de ações no back-end. A interface de edição de ações ainda não existe e há módulos demonstrativos.

**Já foi validado em recortes:** build front-end; compilação do back-end; 267 testes isolados, dos quais 82 são de edição de ações; persistência histórica de PDI e credenciais no Neon. **Não** foi realizada a bateria PostgreSQL recente para edição de ações. Não há deploy ou beta público.

## 2. Incrementos propostos

| Prioridade | Frente | Entrega a planejar | Condição para considerar concluída |
|---|---|---|---|
| 0 | Documentação (G5) | Concluir os documentos centrais, revisão de consistência e publicação autorizada | Links e estados coerentes com commits; nenhuma credencial; revisão e autorização de publicação |
| 1 | Edição de ações | Validar cenários físicos PostgreSQL sob autorização **separada** e integrar PUT à interface com segurança/titularidade | Testes específicos executados no destino autorizado e resultado registrado; interface validada, sem mocks para dados reais |
| 2 | Segurança e base de portfólio | Revisar pendências do front-end local, configuração de desenvolvimento, dados demonstrativos e exposição de histórico sensível | Diff e escopo revisados; segredos eventualmente reais revogados; dados públicos fictícios; evidências verificadas |
| 3 | Ciclo do PDI | Submissão, aprovação/rejeição justificada, reedição/reenvio, execução, acompanhamento e conclusão formal | Regras da Etapa 21B aplicadas por caso de uso, RBAC testado, histórico e persistência validados |
| 4 | Competências e desenvolvimento | Conectar perfil, trilhas/capacitação e evidências a ações de PDI | Modelo e fluxos aprovados, persistência e UX integradas, dados demonstrativos removidos dos módulos concluídos |
| 5 | Gestão e acompanhamento | Recursos de gestor/RH, organograma, reuniões, feedbacks, OKRs, indicadores e demais módulos | Contratos e escopos de acesso definidos, testes por papel; nenhum módulo apresentado como integrado sem evidências |
| 6 | Integrações corporativas | Avaliar Microsoft Entra ID/AD e Teams em incrementos independentes | Integração real, mapeamento seguro de identidade/perfil, tratamento de falhas e testes próprios |
| 7 | Beta público | Selecionar hospedagem, preparar HTTPS, segurança, backups, observabilidade, termos/dados de teste e operação | Checklist de prontidão e autorização; deploy publicado e testado, sem segredos ou dados reais indevidos |

As prioridades acima são **proposta de sequência**, não substituem aprovações do escopo ou tarefas já decididas. Itens podem ser reordenados em uma decisão posterior documentada.

## 3. Gatilhos de qualidade por entrega

1. **Fonte e escopo:** verificar o commit, objetivo e o que ainda é demonstrativo.
2. **Regras:** vincular autorização, titularidade, privacidade e mudanças de estado às decisões vigentes.
3. **Persistência:** conferir a necessidade de migration, DDL, contratos JPA, concorrência e transações. Não executar SQL sem autorização.
4. **Testes:** separar compilação, testes isolados, integração PostgreSQL e validação manual; não transformar um resultado no outro.
5. **Segurança:** revisar credentials/configuração, exposição de dados de teste, rotas fechadas e histórico.
6. **Git:** usar cópia isolada quando necessário, revisar diff, index e histórico; commit e push precisam de autorização específica.
7. **Documentação:** atualizar `PROJECT_STATE.md`, `API.md`, `DECISIONS.md` e `CHANGELOG.md` conforme o incremento, sem afirmar release inexistente.

## 4. Dependências e riscos atuais

- **Conexão aos testes controlados PostgreSQL:** conectividade/VPN e validação explícita do destino; novas gravações estão suspensas.
- **Teste excepcional de rollback:** um cenário de linha de base persiste fixtures iniciais e exige guarda própria e aprovação expressa; não misturar com os quatro casos de rollback integral.
- **Interface incompleta:** back-end de edição de ações não significa fluxo de usuário concluído.
- **Back-end local original:** o Git do projeto IntelliJ antigo conserva outro remoto/histórico e alterações pendentes. Não apontá-lo simplesmente para o repositório oficial nem misturar históricos sem planejamento específico.
- **Front-end local:** quatro arquivos pendentes fora do commit publicado, incluindo problemas identificados na auditoria.
- **Propriedade e privacidade:** dados demonstrativos com procedência não comprovada não devem ser expostos como dados reais ou fictícios seguros.
- **Beta:** GitHub não hospeda automaticamente o back-end nem certifica segurança de produção.

## 5. O que não está previsto como conclusão imediata

Finalizar todo o organograma, histórico de competências, feedbacks, avaliações, notícias, IA, Teams, SSO e dashboards **não é um único incremento**. Esses módulos pertencem à visão do produto e devem ganhar definição, implementação e validação específicas antes de serem anunciados como disponíveis.

[Decisões](DECISIONS.md) · [Arquitetura](ARCHITECTURE.md) · [API atual](API.md) · [Histórico](CHANGELOG.md)
