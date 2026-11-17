# Laboratório Wazuh — Monitoramento de Segurança e Detecção de Ameaças

Laboratório prático de monitoramento de segurança construído com **Wazuh 4.11.2**, logs de segurança do Windows, Sysmon, PowerShell Script Block Logging, regras de detecção personalizadas e simulações controladas.

O projeto demonstra coleta e análise de eventos, monitoramento de endpoints, investigação de atividades no Windows, threat hunting e correlação de eventos em um ambiente de laboratório isolado.

> **Escopo:** as atividades descritas foram realizadas em laboratório controlado. Eventos e serviços de teste não devem ser interpretados como evidência de comprometimento real.

## Objetivos

- Implantar e validar um ambiente de monitoramento com Wazuh.
- Monitorar endpoints Windows e Linux.
- Coletar e analisar eventos de Segurança e Aplicativos do Windows.
- Integrar telemetria do Sysmon para ampliar a visibilidade de processos.
- Monitorar atividades do PowerShell por meio do Script Block Logging.
- Desenvolver e validar regras personalizadas do Wazuh.
- Monitorar criação de contas e serviços do Windows.
- Investigar reconhecimento de rede por meio de eventos do Windows Filtering Platform.
- Validar correlação de eventos usando regras personalizadas.
- Praticar threat hunting pelo Wazuh Dashboard.

## Arquitetura do laboratório

### Componentes

| Componente | Função |
|---|---|
| Wazuh Manager | Análise de eventos e processamento de regras |
| Wazuh Indexer | Armazenamento e indexação dos eventos |
| Wazuh Dashboard | Monitoramento, consultas e threat hunting |
| SEC01 | Servidor Wazuh baseado em Ubuntu |
| DC01 | Endpoint Windows Server / Active Directory |
| WIN-TARGET01 | Endpoint Windows usado nas simulações controladas |
| Kali Linux | Testes controlados e reconhecimento de rede |

### Rede de testes

A rede `SECURITYLAB` (`192.168.250.0/24`) é isolada da Internet e utilizada para atividades controladas.

| Host | Endereço IP | Finalidade |
|---|---|---|
| Kali Linux | `192.168.250.20` | Testes controlados |
| WIN-TARGET01 | `192.168.250.30` | Alvo Windows de laboratório |

Consulte o diagrama em [`docs/arquitetura.md`](docs/arquitetura.md).

## Casos de uso documentados

1. Visão geral do dashboard.
2. Monitoramento de PowerShell e busca no sistema de arquivos.
3. Monitoramento de criação de processos.
4. Monitoramento de criação e controle de serviços.
5. Auditoria de contas e autenticação.
6. Monitoramento de rede e eventos do Windows Filtering Platform.
7. Análise de eventos e threat hunting.

As capturas de tela estão organizadas em [`evidencias/`](evidencias/). O índice de evidências está em [`evidencias/README.md`](evidencias/README.md).

## Engenharia de detecção

As regras personalizadas observadas nas evidências incluem:

| ID | Descrição | Nível |
|---|---|---:|
| 100100 | Controle suspeito de serviço: `sc.exe` executado pelo PowerShell | 10 |
| 100103 | Serviço Windows suspeito configurado para executar `cmd.exe` | Não confirmado |
| 100200 | Teste controlado de criação de processo via Sysmon | 3 |
| 100201 | Execução de `whoami` no Windows detectada | 3 |
| 100202 | Descoberta de informações do sistema Windows detectada | 5 |
| 100203 | Parâmetro de comando codificado do PowerShell detectado | Não confirmado |
| 100400 | Evento sintético de teste de correlação | 3 |
| 100401 | Correlação de três eventos correspondentes confirmada | 10 |

**Observação:** os níveis de 100103 e 100203 não foram confirmados nas evidências disponíveis. As condições XML completas dessas regras também não foram fornecidas para documentação.

Detalhes e limitações estão em [`docs/regras-de-deteccao.md`](docs/regras-de-deteccao.md).

### Correlação de eventos

Foi implementada uma correlação de eventos sintéticos:

- **Regra 100400:** identifica a string `LAB_CORRELATION_TEST` e gera evento de nível 3.
- **Regra 100401:** referencia a regra 100400 por `if_matched_sid`, exige três eventos correspondentes em 60 segundos e gera alerta de nível 10.

Validação realizada com `logger`, `wazuh-logtest`, processamento pelo Wazuh Manager, registro em `alerts.json` e confirmação no Dashboard.

## Monitoramento Windows

### PowerShell Script Block Logging

Canal monitorado: `Microsoft-Windows-PowerShell/Operational`.

Evento relevante: `4104`.

Comandos testados:

```powershell
Get-ChildItem C:\Windows
Get-ChildItem C:\Windows -Recurse
```

Regras nativas observadas:

| ID | Descrição | Nível |
|---|---|---:|
| 91819 | PowerShell script searching filesystem | 4 |
| 91820 | PowerShell script recursively collected files from a filesystem search | 4 |

A regra 91820 possui mapeamentos MITRE ATT&CK T1083 (File and Directory Discovery) e T1119 (Automated Collection). A busca recursiva é um comportamento de descoberta/coleta que precisa ser interpretado no contexto; não é, isoladamente, prova de atividade maliciosa.

### Criação de processos

- Evento Windows: `4688`.
- Descrição nativa observada: `A process was created.`
- A telemetria inclui nome e ID do processo, conta e informações de elevação do token.
- Uma consulta histórica no Dashboard retornou 569 eventos em uma janela de 24 horas.

A contagem é referente à consulta e ao período selecionado, não a incidentes únicos.

### Gerenciamento de contas

- Evento Windows: `4720`.
- Conta criada no teste: `RedTeam-Test`.
- Conta que iniciou a ação: `Labadmin`.
- Endpoint: `WIN-TARGET01`.
- Referência MITRE ATT&CK: T1136 — Create Account.

As evidências disponíveis não demonstram que a conta de teste recebeu privilégios administrativos.

### Criação de serviços

- Evento Windows: `7045`.
- Regra nativa Wazuh: `61138` — `New Windows Service Created`.
- Serviços de teste documentados: `RedTeam-TestSvc`, `Suspicious-TestSvc`, `WazuhRuleTest_DO_NOT_EXIST`, `WazuhRuleTest_100103` e `Detection-TestSvc`.
- Algumas configurações documentadas executaram `cmd.exe`, com inicialização automática e conta `LocalSystem`.
- A regra personalizada `100103` também foi validada contra um serviço configurado para executar `cmd.exe`.
- Referência MITRE ATT&CK: T1543.003 — Windows Service.

Essas atividades foram testes controlados, não evidências de comprometimento.

### Autenticação

| Evento | Atividade | Regra Wazuh |
|---|---|---|
| 4624 | Logon bem-sucedido | 60118 |
| 4625 | Falha de logon | 60122 |
| 4634 | Logoff | Observado no Dashboard |
| 4672 | Privilégios especiais atribuídos a um novo logon | Observado no Dashboard |

Foi observada uma falha isolada com a descrição `Logon Failure - Unknown user or bad password`. Uma única falha não comprova brute force.

## Sysmon e Threat Hunting

Canal: `Microsoft-Windows-Sysmon/Operational`.

Atividades observadas:

- Testes de criação de processos.
- Execução de `whoami`.
- Descoberta de informações do sistema Windows.
- Comandos de descoberta de contas com `net.exe`.
- Atividade de descoberta com `net group`.
- Execução de processo PowerShell envolvendo comando codificado.
- Atividade relacionada ao Application Compatibility Database.

Regras observadas: `100200`, `100201`, `100202`, `100203`, `92039` e `92031`.

Parâmetros codificados do PowerShell são indicadores para investigação, não prova isolada de execução maliciosa.

## Reconhecimento de rede e Windows Filtering Platform

Foi executado um teste controlado a partir do Kali Linux:

```bash
sudo nmap -sS -sV 192.168.250.30
```

Resultado observado:

- O host de destino estava ativo.
- As 1.000 portas TCP examinadas foram reportadas como filtradas.
- Não houve respostas para essas portas filtradas.

Eventos Windows investigados:

| Evento | Descrição |
|---|---|
| 5152 | Windows Filtering Platform bloqueou um pacote |
| 5157 | Windows Filtering Platform bloqueou uma conexão |

A telemetria coletada incluiu tráfego originado em `192.168.250.20` com destino a `192.168.250.30`. Referência MITRE ATT&CK: T1046 — Network Service Discovery.

**Limite da evidência:** os registros confirmam que o Windows Filtering Platform registrou atividade de rede bloqueada e que o Wazuh coletou esses eventos. Eles não provam, por si só, que o Wazuh identificou o Nmap ou que cada porta examinada gerou um evento individual.

## Visão geral do Dashboard

Uma captura histórica de 17 de novembro de 2026 mostrou:

| Métrica | Valor |
|---|---:|
| Agentes ativos | 2 |
| Agentes desconectados | 0 |
| Alertas totais | 8.586 |
| Alertas críticos | 0 |
| Alertas altos | 1 |
| Alertas médios | 94 |
| Alertas baixos | 8.490 |
| Falhas de autenticação | 32 |
| Autenticações bem-sucedidas | 3.271 |

Esses números são contadores históricos do Dashboard, não incidentes de segurança únicos confirmados.

## Documentação complementar

- [Arquitetura do laboratório](docs/arquitetura.md)
- [Regras de detecção](docs/regras-de-deteccao.md)
- [Validação e evidências](docs/validacao-e-evidencias.md)
- [Instalação e configuração](docs/instalacao.md) — modelo a completar com os procedimentos efetivamente executados
- [Solução de problemas](docs/solucao-de-problemas.md) — modelo a completar com ocorrências e correções confirmadas

## Segurança e cuidados

- As simulações foram realizadas em ambiente controlado.
- A rede SECURITYLAB é isolada da Internet.
- Os testes devem permanecer restritos a endpoints autorizados.
- Eventos sintéticos servem para validar a lógica de detecção.
- Contas e serviços de teste não devem ser descritos como comprometimento real.
- Não publique senhas, tokens, chaves privadas, certificados privados ou dados sensíveis de rede.

## Competências demonstradas

- Implantação e administração de SIEM.
- Administração do Wazuh Manager e agentes.
- Análise de Windows Event Logs.
- Integração com Sysmon.
- Monitoramento de segurança do PowerShell.
- Engenharia de regras de detecção.
- Correlação de eventos.
- Threat hunting.
- Investigação de autenticação, processos e serviços.
- Monitoramento de segurança de rede.
- Mapeamento de contexto para MITRE ATT&CK.
- Validação de eventos e alertas.

---

**Projeto:** Wazuh Security Monitoring & Threat Detection Lab  
**Plataforma:** VMware  
**SIEM:** Wazuh 4.11.2  
**Ambiente de testes:** laboratório isolado Windows e Linux
