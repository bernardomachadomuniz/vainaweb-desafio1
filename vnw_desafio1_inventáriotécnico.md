# Inventário Técnico Detalhado 

> ⚠️ **CORREÇÃO TÉCNICA APLICADA**: Durante a coleta, os arquivos `guest_net_ping.txt` e `infra_net_ping.txt` foram executados com redes trocadas. Este inventário foi corrigido baseando-se nos hostnames reais dos dispositivos para garantir precisão técnica.

## ⚠️ Observação Metodológica Importante

**Inconsistência Detectada nos Arquivos de Scan:**
- **Arquivo `guest_net_ping.txt`**: Executou scan na rede 10.10.30.0/24 (que é infra_net)
- **Arquivo `infra_net_ping.txt`**: Executou scan na rede 10.10.50.0/24 (que é guest_net)

**Correção Aplicada:**
A identificação correta das redes foi realizada através da análise dos hostnames dos dispositivos:
- Hostnames terminados em `guest_net` → Rede 10.10.50.0/24 
- Hostnames terminados em `infra_net` → Rede 10.10.30.0/24
- Servidores com funções específicas (ftp-server, mysql-server, etc.) → Rede 10.10.30.0/24

Esta correção garante a precisão técnica do relatório e não afeta a validade dos dados coletados.

---------|--------|
| **Total de Hosts Descobertos** | 20 |
| **Redes Mapeadas** | 3 |
| **Serviços Críticos** | 6 |
| **Vulnerabilidades Alta Prioridade** | 2 |
| **Data do Levantamento** | 28/07/2025 23:39 UTC |

---

## Rede Corporativa (corp_net) - 10.10.10.0/24

### Hosts Ativos: 6

| IP Address | Hostname | MAC Address | Função | Portas | Serviços | Nível de Risco | Observações |
|------------|----------|-------------|---------|---------|----------|----------------|-------------|
| 10.10.10.1 | - | 12:6d:16:27:c8:40 | Gateway | 111, 60787 | RPC, NFS | 🟡 Baixo | Gateway da rede |
| 10.10.10.2 | 05c2b3804225 | - | Analyst Container | - | - | 🟢 Seguro | Host de análise |
| 10.10.10.10 | WS_001 | e6:05:22:28:32:61 | Workstation | - | - | 🟢 Baixo | Estação de trabalho |
| 10.10.10.101 | WS_002 | 02:0f:88:c2:e9:7b | Workstation | - | - | 🟢 Baixo | Estação de trabalho |
| 10.10.10.127 | WS_003 | 2a:9e:13:f9:aa:e9 | Workstation | - | - | 🟢 Baixo | Estação de trabalho |
| 10.10.10.222 | WS_004 | 52:c6:82:61:82:f9 | Workstation | - | - | 🟢 Baixo | Estação de trabalho |

---

## Rede de Convidados (guest_net) - 10.10.50.0/24

### Hosts Ativos: 6

| IP Address | Hostname | MAC Address | Função | Portas | Serviços | Nível de Risco | Observações |
|------------|----------|-------------|---------|---------|----------|----------------|-------------|
| 10.10.50.1 | - | - | Gateway | - | - | 🟢 Baixo | Gateway da rede |
| 10.10.50.2 | notebook-carlos | - | Dispositivo Guest | - | - | 🟡 Médio | Dispositivo não gerenciado |
| 10.10.50.3 | laptop-vastro | - | Dispositivo Guest | - | - | 🟡 Médio | Dispositivo não gerenciado |
| 10.10.50.4 | laptop-luiz | - | Dispositivo Guest | - | - | 🟡 Médio | Dispositivo não gerenciado |
| 10.10.50.5 | macbook-aline | - | Dispositivo Guest | - | - | 🟡 Médio | Dispositivo não gerenciado |
| 10.10.50.6 | 05c2b3804225 | - | Analyst Container | - | - | 🟢 Seguro | Host de análise |

---

## Rede de Infraestrutura (infra_net) - 10.10.30.0/24

### Hosts Ativos: 8

| IP Address | Hostname | MAC Address | Função | Portas Abertas | Serviços Detectados | Nível de Risco | Detalhes Técnicos |
|------------|----------|-------------|---------|----------------|-------------------|----------------|-------------------|
| 10.10.30.1 | - | 2a:9c:a3:4a:20:41 | Gateway | 111, 60787 | RPC | 🟡 Baixo | Gateway da infraestrutura |
| 10.10.30.2 | 05c2b3804225 | - | Analyst Container | 33282, 52410 | - | 🟢 Seguro | Host de análise |
| 10.10.30.10 | ftp-server | 7a:a0:f5:c4:6f:4f | Servidor FTP | 21 | FTP | 🔴 ALTO | FTP ativo - Verificar acesso anônimo |
| 10.10.30.11 | mysql-server | 6e:05:3e:6f:cf:74 | Servidor BD | 3306, 33060 | MySQL 8.0.43 | 🔴 ALTO | MySQL exposto - Auth: caching_sha2_password |
| 10.10.30.15 | samba-server | 6a:88:13:da:bf:d2 | Servidor Arquivos | 139, 445 | SMB/NetBIOS | 🟡 Médio | Serviços de compartilhamento |
| 10.10.30.17 | openldap | 32:65:1b:58:1f:87 | Servidor LDAP | 389, 636 | LDAP/LDAPS | 🟡 Médio | Diretório organizacional |
| 10.10.30.117 | zabbix-server | 76:a2:24:6d:d0:07 | Monitoramento | 80, 10051, 10052 | Zabbix + Nginx + PHP 7.3.14 | 🟡 Médio | Interface web de monitoramento |
| 10.10.30.227 | legacy-server | 4a:30:d2:43:df:7e | Servidor Legado | - | Desconhecido | 🟡 Médio | Requer investigação adicional |

---

## Análise Detalhada de Serviços Críticos

### 🔴 MySQL Server (10.10.30.11)
```
Versão: MySQL 8.0.43
Protocolo: 10
Thread ID: 12
Autenticação: caching_sha2_password
Capabilities: SupportsCompression, Support41Auth, FoundRows, etc.
Status: Autocommit ativo
```
**Recomendação**: Implementar autenticação robusta e firewall

### 🔴 FTP Server (10.10.30.10)
```
Porta: 21/tcp
Status: Aberto
Teste de acesso anônimo: Necessário
```
**Recomendação**: Avaliar necessidade e implementar SFTP

### 🟡 Zabbix Server (10.10.30.117)
```
Servidor: nginx
Framework: PHP/7.3.14
Autenticação: Formulário web
Headers de segurança: Implementados parcialmente
```
**Recomendação**: Atualizar PHP e implementar 2FA

---

## Mapa de Conectividade

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   CORP_NET      │    │   GUEST_NET     │    │   INFRA_NET     │
│  10.10.10.0/24  │    │  10.10.50.0/24  │    │  10.10.30.0/24  │
│                 │    │                 │    │                 │
│ • 4 Workstations│    │ • 5 Dispositivos│    │ • 6 Servidores  │
│ • 1 Gateway     │    │ • 1 Gateway     │    │ • 1 Gateway     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                        ┌─────────────────┐
                        │   ANALYST       │
                        │  (Multi-homed)  │
                        │                 │
                        │ eth0: 10.10.50.6│
                        │ eth1: 10.10.30.2│
                        │ eth2: 10.10.10.2│
                        └─────────────────┘
```

---

## Plano de Ação Imediata

### Próximas 24 horas
- [ ] Verificar acesso anônimo ao FTP (10.10.30.10)
- [ ] Testar autenticação MySQL (10.10.30.11)
- [ ] Avaliar configurações do Zabbix

### Próxima semana
- [ ] Implementar segmentação de rede
- [ ] Configurar firewall interno
- [ ] Audit do servidor legacy

### Próximo mês
- [ ] Atualização de sistemas
- [ ] Implementação de monitoramento de segurança
- [ ] Treinamento da equipe

---

**Responsável pela Análise**: Bernardo Machado Muniz  
**Data de Geração**: 28/07/2025  

