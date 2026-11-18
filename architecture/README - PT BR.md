# Laboratório Wazuh SIEM – Arquitetura

Este diretório contém o diagrama da arquitetura lógica do laboratório Wazuh SIEM.

## Visão Geral do Ambiente

O laboratório foi construído utilizando VMware e inclui os seguintes componentes:

- **SEC01:** Ubuntu Server executando o Wazuh Manager, o Wazuh Indexer e o Wazuh Dashboard.
- **DC01:** Controlador de domínio Windows Server executando o agente Wazuh.
- **WIN-TARGET01:** Endpoint Windows executando o agente Wazuh.
- **Kali Linux:** Máquina para testes de segurança, utilizada para gerar atividades controladas.
- **SECURITYLAB:** Rede virtual isolada (`192.168.250.0/24`).

## Arquitetura de Monitoramento

Os endpoints Windows enviam telemetria de segurança, por meio de seus agentes Wazuh, para o Wazuh Manager.

O Manager processa os eventos recebidos e encaminha os dados para o Indexer, onde ficam disponíveis para investigação por meio do Dashboard.

O Kali Linux é utilizado para realizar testes de segurança controlados contra o ambiente alvo.

## Diagrama

O arquivo `architecture.mmd` contém o código-fonte Mermaid referente ao diagrama da arquitetura lógica.

Você pode visualizá-lo utilizando o [Mermaid Live Editor](https://mermaid.live/).

> Nota: Este diagrama representa a arquitetura lógica de monitoramento e não pretende documentar todas as interfaces de rede virtual do VMware, regras de firewall ou rotas de rede física.