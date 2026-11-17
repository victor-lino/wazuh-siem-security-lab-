# Validação e evidências

## Padrão de documentação por caso de uso

Para cada caso de uso, registre a sequência:

1. **Ação/teste:** comando ou ação executada no endpoint.
2. **Evento original:** Event ID, canal e campos relevantes.
3. **Regra Wazuh:** ID, descrição e nível, quando confirmados.
4. **Alerta:** registro no `alerts.json` e/ou resultado no Dashboard.
5. **Interpretação:** o que a evidência comprova e o que ela não comprova.

## Casos documentados

| Caso de uso | Evidência principal | Sequência disponível |
|---|---|---|
| PowerShell | Event ID 4104; regras 91819/91820 | Comando → evento → regra |
| Criação de processo | Event ID 4688; Sysmon | Telemetria → consulta/alerta |
| Criação de serviço | Event ID 7045; regra 61138; custom 100103 | Serviço de teste → evento → regra |
| Controle de serviço | Custom 100100 | Execução de `sc.exe` via PowerShell → alerta |
| Criação de conta | Event ID 4720 | Ação → evento → investigação |
| Autenticação | Event IDs 4624/4625/4634/4672 | Evento → regra nativa quando confirmada |
| Reconhecimento de rede | Nmap + eventos 5152/5157 | Teste de rede → WFP → busca no Wazuh |
| Correlação | Regras 100400/100401 | `logger` → regra 100400 → regra 100401 → alerta |

## Inventário de capturas

As imagens estão organizadas em `../evidencias/`. Consulte o índice em `../evidencias/README.md`.

## Boas práticas para novas evidências

- Use capturas legíveis, sem cortar o Event ID ou o Rule ID.
- Sempre que possível, capture o comando/teste e o resultado no mesmo caso de uso.
- Preserve data/hora e hostname quando forem relevantes.
- Oculte endereços públicos, nomes de usuários reais, tokens, chaves e outros dados sensíveis.
- Não use contagens do Dashboard como quantidade de incidentes únicos.
- Diferencie eventos sintéticos, atividades administrativas e alertas de segurança.
