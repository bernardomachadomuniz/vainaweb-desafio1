# Relatório Técnico – Lab Segmentação de Rede

**Autor:** Bernardo Machado Muniz  
**Data:** 28 de Julho de 2025  
**Versão:** 1.0

---

## Sumário Executivo

A análise da rede simulada revelou um ambiente com 3 redes segmentadas contendo 20 hosts ativos distribuídos em funções corporativas, infraestrutura e convidados. Foram identificados 6 serviços críticos na rede de infraestrutura, sendo 2 de alto risco (MySQL e FTP expostos). A principal vulnerabilidade encontrada é a falta de isolamento adequado entre redes críticas e a exposição de serviços sensíveis sem controles de acesso apropriados. Recomenda-se implementação imediata de firewall interno e hardening dos serviços MySQL e FTP.

---

## Objetivo

Analisar a rede simulada para identificar exposição, segmentação e riscos operacionais através de mapeamento ativo de hosts, descoberta de serviços e avaliação de vulnerabilidades de segurança.

---

## Escopo

Ambiente Docker simulado com múltiplos hosts distribuídos em redes segmentadas:
- **corp_net**: 10.10.10.0/24 (Rede Corporativa)
- **guest_net**: 10.10.50.0/24 (Rede de Convidados) 
- **infra_net**: 10.10.30.0/24 (Rede de Infraestrutura)

> ⚠️ **NOTA TÉCNICA**: Durante a coleta, arquivos `guest_net_ping.txt` e `infra_net_ping.txt` foram executados com redes trocadas. A correção foi aplicada baseando-se nos hostnames dos dispositivos.

---

## Metodologia

### Ferramentas Utilizadas
- **nmap**: Descoberta de hosts e análise de serviços
- **rustscan**: Varredura rápida de portas
- **ping**: Teste de conectividade  
- **curl**: Análise de serviços web
- **scripts NSE**: Detecção de vulnerabilidades específicas

### Processo de Coleta
1. **Reconhecimento Inicial**: Identificação de interfaces via `ip a`
2. **Varredura de Hosts**: Ping sweep com `nmap -sn` em cada rede
3. **Descoberta de Portas**: Scan rápido com `rustscan` 
4. **Análise de Serviços**: Scripts NSE específicos para cada serviço
5. **Coleta de Evidências**: Documentação e organização dos resultados

---

## Redes Identificadas

| Nome Estimado | Subnet Descoberta | Finalidade Suposta |
|---------------|-------------------|-------------------|
| corp_net | 10.10.10.0/24 | Rede principal dos dispositivos corporativos |
| guest_net | 10.10.50.0/24 | Rede de visitantes e dispositivos não gerenciados |
| infra_net | 10.10.30.0/24 | Infraestrutura / servidores críticos |

---

## Dispositivos por Rede

### corp_net (10.10.10.0/24)

| IP | Função | Evidência |
|---|---|---|
| 10.10.10.1 | Gateway | Portas 111, 60787 (RPC services) |
| 10.10.10.10 | Estação de trabalho (WS_001) | Responde ping, hostname identificado |
| 10.10.10.101 | Estação de trabalho (WS_002) | Responde ping, hostname identificado |
| 10.10.10.127 | Estação de trabalho (WS_003) | Responde ping, hostname identificado |
| 10.10.10.222 | Estação de trabalho (WS_004) | Responde ping, hostname identificado |
| 10.10.10.2 | Container analyst | Interface eth2 do host de análise |

### guest_net (10.10.50.0/24)

| IP | Função | Evidência |
|---|---|---|
| 10.10.50.1 | Gateway | Responde ping, função de roteamento |
| 10.10.50.2 | Dispositivo guest (notebook-carlos) | Hostname indica usuário visitante |
| 10.10.50.3 | Dispositivo guest (laptop-vastro) | Hostname indica usuário visitante |
| 10.10.50.4 | Dispositivo guest (laptop-luiz) | Hostname indica usuário visitante |
| 10.10.50.5 | Dispositivo guest (macbook-aline) | Hostname indica usuário visitante |
| 10.10.50.6 | Container analyst | Interface eth0 do host de análise |

### infra_net (10.10.30.0/24)

| IP | Função | Evidência |
|---|---|---|
| 10.10.30.1 | Gateway | Portas 111, 60787 (RPC services) |
| 10.10.30.10 | Servidor FTP | Porta 21 ativa (ftp-server) |
| 10.10.30.11 | Servidor MySQL | Porta 3306 ativa, versão 8.0.43 |
| 10.10.30.15 | Servidor Samba | Portas 139, 445 (SMB/NetBIOS) |
| 10.10.30.17 | Servidor LDAP | Portas 389, 636 (LDAP/LDAPS) |
| 10.10.30.117 | Servidor Zabbix | Porta 80, nginx + PHP 7.3.14 |
| 10.10.30.227 | Servidor Legado | Sem serviços detectados |
| 10.10.30.2 | Container analyst | Interface eth1 do host de análise |

---

## Diagrama de Rede

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          AMBIENTE DOCKER SEGMENTADO                             │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────────────┐
│   CORP_NET      │    │   GUEST_NET     │    │        INFRA_NET                │
│  10.10.10.0/24  │    │  10.10.50.0/24  │    │      10.10.30.0/24              │
│                 │    │                 │    │                                 │
│ Gateway (.1)    │    │ Gateway (.1)    │    │ Gateway (.1)                    │
│ WS_001 (.10)    │    │ notebook-carlos │    │ ftp-server (.10) [RISCO ALTO]   │
│ WS_002 (.101)   │    │ laptop-vastro   │    │ mysql-server (.11) [RISCO ALTO] │
│ WS_003 (.127)   │    │ laptop-luiz     │    │ samba-server (.15)              │
│ WS_004 (.222)   │    │ macbook-aline   │    │ openldap (.17)                  │
│                 │    │                 │    │ zabbix-server (.117)            │
│                 │    │                 │    │ legacy-server (.227)            │
└─────────────────┘    └─────────────────┘    └─────────────────────────────────┘
         │                       │                              │
         └───────────────────────┼──────────────────────────────┘
                                 │
                        ┌─────────────────┐
                        │     ANALYST     │
                        │   (Multi-homed) │
                        │                 │
                        │ eth2: .10.10.2  │
                        │ eth0: .50.50.6  │
                        │ eth1: .30.30.2  │
                        └─────────────────┘
```

---

## Diagnóstico (Achados)

### 🔴 **RISCOS CRÍTICOS**

#### 10.10.30.10 - Servidor FTP - Porta 21
- **Risco identificado**: Serviço FTP ativo sem criptografia
- **Evidência**: `Open 10.10.30.10:21` via rustscan
- **Impacto**: Transferência de dados em texto claro, possível acesso não autorizado

#### 10.10.30.11 - Servidor MySQL - Porta 3306
- **Risco identificado**: Banco de dados MySQL exposto na rede
- **Evidência**: 
  ```
  mysql-info: 
  Version: 8.0.43
  Auth Plugin Name: caching_sha2_password
  ```
- **Impacto**: Acesso direto ao banco de dados, possível exfiltração de informações

### 🟡 **RISCOS MÉDIOS**

#### 10.10.30.15 - Servidor Samba - Portas 139, 445
- **Risco identificado**: Serviços SMB/NetBIOS ativos
- **Evidência**: `Open 10.10.30.15:139` e `Open 10.10.30.15:445`
- **Impacto**: Possível enumeração de compartilhamentos e usuários

#### 10.10.30.117 - Servidor Zabbix - Porta 80
- **Risco identificado**: Interface web de monitoramento exposta
- **Evidência**: 
  ```
  Server: nginx
  X-Powered-By: PHP/7.3.14
  ```
- **Impacto**: Acesso não autorizado ao sistema de monitoramento

### ⚠️ **PROBLEMAS ARQUITETURAIS**

#### Falta de Segmentação Lógica
- **Evidência**: Container analyst com acesso simultâneo às 3 redes
- **Impacto**: Possibilidade de movimentação lateral entre redes

#### Servidor Legado sem Serviços Identificados
- **Evidência**: Host 10.10.30.227 responde ping mas sem portas abertas detectadas
- **Impacto**: Possível configuração inadequada ou serviços não standard

---

## Observações de Risco

- A **guest_net** aparenta estar isolada de serviços críticos, porém o roteamento entre redes não foi testado completamente
- A **infra_net** contém servidores críticos que deveriam estar isolados de estações de trabalho comuns
- **Ausência de firewall interno** permite comunicação irrestrita entre redes através do container analyst
- **Serviços legacy** podem representar vetores de ataque não documentados

---

## Recomendações

### **Segurança de Rede**
1. **Implementar firewall interno** para controlar tráfego entre segmentos
2. **Isolar completamente** a infra_net das demais redes
3. **Revisar necessidade** de conectividade entre corp_net e guest_net

### **Hardening de Serviços**
1. **Desabilitar serviço FTP** ou migrar para SFTP/FTPS
2. **Configurar autenticação robusta** no MySQL com usuários específicos
3. **Atualizar PHP** no servidor Zabbix para versão suportada
4. **Implementar HTTPS** em todos os serviços web

### **Monitoramento e Controle**
1. **Implementar SIEM** para detecção de anomalias
2. **Configurar logs centralizados** de todos os serviços
3. **Estabelecer baseline** de comunicação normal entre redes

---

## Plano de Ação (80/20)

| Ação | Impacto | Facilidade | Prioridade |
|------|---------|------------|------------|
| **Isolar infra_net** | Alto | Médio | **Alta** |
| **Desabilitar FTP ou migrar SFTP** | Alto | Alto | **Alta** |
| **Configurar autenticação MySQL** | Alto | Alto | **Alta** |
| **Implementar firewall interno** | Alto | Baixo | **Alta** |
| **Investigar servidor legado** | Médio | Alto | Média |
| **Atualizar PHP do Zabbix** | Médio | Médio | Média |
| **Implementar monitoramento de segurança** | Alto | Baixo | Média |
| **Hardening geral dos serviços** | Médio | Baixo | Baixa |

---

## Conclusão

A rede apresenta exposição significativa devido à falta de controles adequados de acesso e segmentação insuficiente. Os serviços críticos na infra_net representam o maior risco, especialmente MySQL e FTP que estão expostos sem proteção adequada. 

**Próximos passos imediatos**:
1. Implementação de firewall para isolar a infra_net
2. Hardening dos serviços MySQL e FTP  
3. Auditoria completa do servidor legado

A arquitetura atual permite movimentação lateral através do container analyst, necessitando revisão urgente da topologia de rede para garantir isolamento adequado entre os segmentos.

---

## Anexos

### Evidências dos Scans
- **recon-redes.txt**: Interfaces de rede identificadas
- **corp_net_ping.txt**: Descoberta de hosts corporativos
- **infra_net_ips_ports.txt**: Portas abertas na infraestrutura  
- **infra_net_servico_mysql-info.txt**: Detalhes do servidor MySQL
- **infra_net_servico_webserver.txt**: Headers HTTP do Zabbix
- **infra_net_servico_zabbix.txt**: Página de autenticação

### Prints de Ferramentas
- Saída do comando `docker ps` mostrando containers ativos
- Resultado do `ip a` identificando interfaces multi-homed
- Scans do nmap com descoberta de hosts e serviços
- Evidências do rustscan com portas abertas por rede

### Observação sobre Nomenclatura
> ⚠️ **Inconsistência nos Arquivos**: Os arquivos `guest_net_ping.txt` e `infra_net_ping.txt` foram executados com redes trocadas. A correção foi aplicada baseando-se nos hostnames dos dispositivos, garantindo precisão técnica do relatório.

---

**Classificação**: CONFIDENCIAL