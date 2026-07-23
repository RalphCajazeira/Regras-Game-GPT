# Arquitetura Anterior — Custom GPT com Actions

**Status:** legado e compatibilidade

Este documento preserva a decisão arquitetural anterior baseada em:

```text
Custom GPT
→ REST + OpenAPI Actions
→ backend
```

Ela foi substituída como arquitetura principal por:

```text
ChatGPT App/Plugin
→ Apps SDK
→ MCP Tools + Widget JS
→ serviços de aplicação
→ backend e banco
```

## O que permanece útil

- REST interno ou público;
- OpenAPI para documentação e testes;
- idempotência;
- `stateVersion`;
- erros acionáveis;
- serviços de domínio;
- snapshots;
- bundles;
- checkpoints;
- compatibilidade textual sem widget.

## O que deixou de ser canônico

- limite de 30 Actions como restrição central da arquitetura;
- catálogo principal exposto somente por `operationId` OpenAPI;
- Custom GPT Actions como interface principal do jogo;
- GPT como único caminho para cada comando comum.

## Quando usar

- fallback de desenvolvimento;
- testes isolados do backend;
- compatibilidade temporária durante migração;
- cliente textual alternativo;
- integração administrativa específica.

Nenhuma regra de domínio deve ser implementada exclusivamente nesta fachada.
