# Regras de detecção

Este documento registra somente os IDs, descrições, níveis e comportamentos que foram observados nas evidências disponíveis. Não substitui os arquivos XML originais das regras.

## Regras personalizadas observadas

| ID | Descrição | Nível confirmado? |
|---|---|---|
| 100100 | Suspicious service control: `sc.exe` executed from PowerShell | Sim — 10 |
| 100103 | Suspicious Windows service configured to execute `cmd.exe` | Não |
| 100200 | LAB: controlled Sysmon process creation test | Sim — 3 |
| 100201 | LAB: Windows whoami process execution detected | Sim — 3 |
| 100202 | LAB: Windows system information discovery detected | Sim — 5 |
| 100203 | LAB: PowerShell encoded command parameter detected | Não |
| 100400 | LAB: evento sintético de teste | Sim — 3 |
| 100401 | LAB: correlação de três eventos confirmada | Sim — 10 |

Os XMLs completos das regras 100100, 100103, 100200–100203 não foram incluídos nesta documentação porque as condições completas não foram fornecidas. Adicione-os somente após exportar e revisar os arquivos reais do Manager.

## Correlação customizada 100400/100401

Arquivo observado no Manager: `local_auth_test.xml`.

```xml
<group name="lab_correlation_test,">
  <rule id="100400" level="3">
    <match>LAB_CORRELATION_TEST</match>
    <description>LAB: evento sintetico de teste</description>
  </rule>
  <rule id="100401" level="10" frequency="3" timeframe="60">
    <if_matched_sid>100400</if_matched_sid>
    <description>LAB: correlacao de tres eventos confirmada</description>
  </rule>
</group>
```

### Comportamento validado

1. O evento sintético `LAB_CORRELATION_TEST` corresponde à regra 100400.
2. A regra 100401 usa `if_matched_sid` para acompanhar ocorrências da 100400.
3. Três eventos correspondentes dentro de 60 segundos geram a regra 100401, nível 10.
4. A validação foi feita com `logger`, `wazuh-logtest`, `alerts.json` e o Dashboard.

### Regras nativas observadas

| ID | Descrição | Contexto |
|---|---|---|
| 91819 | PowerShell script searching filesystem | PowerShell Script Block Logging, Event ID 4104 |
| 91820 | PowerShell script recursively collected files from a filesystem search | Busca recursiva; MITRE T1083 e T1119 |
| 61138 | New Windows Service Created | Windows Event ID 7045 |
| 60118 | Windows Workstation Logon Success | Windows Event ID 4624 |
| 60122 | Logon Failure - Unknown user or bad password | Windows Event ID 4625 |
| 60104 | Windows audit failure event | Observada com evento 5157 |

**Cuidado:** não atribua a regra 60104 ao evento 5152 sem evidência específica. Não presuma que os níveis ou condições de regras não listados aqui sejam conhecidos.
