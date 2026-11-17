# Solução de problemas — registro de ocorrências

Este documento deve conter apenas problemas que realmente ocorreram no laboratório e os procedimentos que resolveram ou mitigaram cada um.

## Modelo de registro

### [Título do problema]

- **Componente afetado:**
- **Sintoma observado:**
- **Mensagem de erro/log:**
- **Causa identificada:**
- **Procedimento executado:**
- **Resultado após a correção:**
- **Evidência:**

## Verificações úteis

As verificações abaixo são sugestões gerais, não afirmações de que todos esses problemas ocorreram neste laboratório:

- Estado dos serviços Wazuh.
- Conectividade entre agente e Manager.
- Estado e configuração dos agentes.
- Logs do Manager e do Indexer.
- Validação das regras com `wazuh-logtest`.
- Confirmação de eventos em `alerts.json` e no Dashboard.

Inclua comandos específicos somente após validar que correspondem à versão e à configuração efetivamente utilizadas.
