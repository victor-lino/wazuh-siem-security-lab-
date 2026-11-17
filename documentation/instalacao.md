# Instalação e configuração — modelo para completar

Este arquivo é um esqueleto para documentar a implantação real. Os detalhes abaixo não devem ser preenchidos por suposição.

## Ambiente

- Plataforma de virtualização: VMware
- Servidor: `SEC01` — Ubuntu
- Versão do Wazuh: `4.11.2`
- Componentes: Wazuh Manager, Indexer e Dashboard
- Endpoints: `DC01` e `WIN-TARGET01`

## Pré-requisitos

> Completar com as versões exatas de Ubuntu, Windows Server, Windows cliente, recursos de CPU/RAM/disco e requisitos de rede usados no laboratório.

## Etapas de implantação

1. [ ] Registrar a versão e a edição do sistema operacional de `SEC01`.
2. [ ] Documentar o método e os comandos usados para instalar Wazuh 4.11.2.
3. [ ] Registrar a configuração do Manager, Indexer e Dashboard.
4. [ ] Documentar o processo real de instalação e registro dos agentes.
5. [ ] Documentar a configuração de coleta de eventos Windows e Sysmon.
6. [ ] Documentar como as regras personalizadas foram implantadas.
7. [ ] Validar comunicação, coleta e geração de alertas.

## Dados que faltam para finalizar este documento

- Versão exata do Ubuntu e dos endpoints Windows.
- Comandos de instalação realmente utilizados.
- Método de instalação (assistente, repositório ou instalação distribuída).
- Configurações relevantes do `ossec.conf` sem segredos.
- Procedimento de instalação/registro dos agentes.
- Configuração de Sysmon e canais de eventos.
