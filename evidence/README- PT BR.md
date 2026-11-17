# Índice de evidências

As capturas estão separadas por caso de uso. Os números no início dos nomes preservam uma referência de organização do conjunto original.

| Pasta | Conteúdo |
|---|---|
| `01-visao-geral-dashboard` | Visão geral do Wazuh Dashboard e resumo de alertas |
| `02-monitoramento-powershell` | Script Block Logging, busca de arquivos e regras 91819/91820 |
| `03-monitoramento-processos` | Criação e investigação de processos |
| `04-monitoramento-servicos` | Evento 7045, regra 61138 e regras customizadas de serviços |
| `05-contas-e-autenticacao` | Gerenciamento de contas, logons e eventos de autenticação |
| `06-monitoramento-de-rede` | Teste Nmap e eventos Windows Filtering Platform |
| `07-analise-de-eventos` | Consultas, resultados e análise de eventos no Wazuh |

## Como apresentar cada caso

Sempre que possível, mantenha a ordem:

**Teste executado → evento original → regra Wazuh → alerta no Dashboard.**

Nem todos os casos possuem uma captura individual de cada etapa. Quando uma etapa não estiver documentada, não a apresente como evidência confirmada.
