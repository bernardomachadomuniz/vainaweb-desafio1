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
<img width="1571" height="931" alt="vnw-diagrama drawio" src="https://github.com/user-attachments/assets/41e1f400-c79d-4b87-b882-8f4beb7c31b9" />
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

### Evidências em Arquivos
[vnw-evidências-arquivos.zip](https://github.com/user-attachments/files/21517277/vnw-evidencias-arquivos.zip)

- recon-redes.txt: Interfaces de rede identificadas no container analyst
- recon_ip_maps.txt: Mapeamento ARP de IPs para endereços MAC
Rede Corporativa (corp_net - 10.10.10.0/24)
- corp_net_ping.txt: Descoberta de hosts corporativos (WS_001 a WS_004)
- corp_net_ips.txt: Lista de IPs da rede corporativa
- corp_net_ips_hosts.txt: Mapeamento IP-hostname dos workstations
- corp_net_ips_ports.txt: Portas abertas na rede corporativa
- ports_corp_net.txt: Arquivo de portas (truncado)
Rede de Infraestrutura (infra_net - 10.10.30.0/24)
- guest_net_ping.txt: Descoberta de hosts de infraestrutura (serviços críticos)
- infra_net_ips.txt: Lista de IPs da rede de infraestrutura
- infra_net_ips_ports.txt: Portas abertas na infraestrutura
- infra_net_servico_ftp-anon.txt: Informações do servidor FTP
- infra_net_servico_mysql-info.txt: Detalhes do servidor MySQL 8.0.43
- infra_net_servico_webserver.txt: Headers HTTP do servidor web Zabbix
- infra_net_servico_zabbix.txt: Página de autenticação do Zabbix
Rede de Convidados (guest_net - 10.10.50.0/24)
- infra_net_ping.txt: Descoberta de dispositivos pessoais (laptops/notebooks)
- guest_net_ips.txt: Lista de IPs da rede de convidados
- guest_net_ips_hosts.txt: Mapeamento IP-hostname dos dispositivos pessoais

### Evidências em Imagens
[vnw-evidências-imagens.zip](https://github.com/user-attachments/files/21517286/vnw-evidencias-imagens.zip)

Imagem 1 - docker-compose.yml (Estações Corporativas - corp_net):
- Configuração das estações de trabalho WS_001 a WS_004
- Rede corporativa (10.10.10.0/24) com IPs fixos
- Containers Ubuntu com comando tail -f /dev/null

Imagem 2 - docker-compose.yml (Estações Pessoais - guest_net):
- Laptops pessoais (laptop-vastro, laptop-luiz, macbook-aline, notebook-carlos)
- Rede de convidados sem IPs fixos definidos

Imagem 3 - docker-compose.yml (Infraestrutura - infra_net):
- Serviços críticos: MySQL, Samba, OpenLDAP, Zabbix, Legacy Server
- Rede de infraestrutura (10.10.30.0/24) com IPs fixos
- FTP server com imagem específica stillstudio/pure-ftpd

Imagem 4 - docker-compose.yml (Configuração de Redes e Analyst):
- Definição das três redes (corp_net, infra_net, guest_net)
- Container analyst com acesso a todas as redes
- Configuração baseada em Kali Linux com ferramentas de pentest

Imagem 5 - analyst/Dockerfile (início):
- Dockerfile do container analyst baseado em kalilinux/kali-rolling
- Início da instalação de ferramentas de reconhecimento

Imagem 6 - analyst/Dockerfile (ferramentas):
- Continuação do Dockerfile com instalação de ferramentas
- Inclui: iproute2, iputils-ping, dnsutils, net-tools, nmap, curl, wget, git
- Download e instalação do RustScan
- Script de wordlist personalizada

Imagem 7 - Execução do docker-compose up:
- Log de inicialização de todos os containers
- Status "Built/Created/Started" para redes e containers
- Confirmação que o ambiente de lab foi criado com sucesso

Imagem 8 - recon-redes.txt:
- Output do comando ip a | grep inet
- Interfaces de rede identificadas no container analyst
- Redes: 127.0.0.1/8, 10.10.50.6/24, 10.10.30.2/24, 10.10.10.2/24

Imagem 9 - Testes de conectividade:
- Ping para diferentes redes (10.10.10.1, 10.10.30.1, 10.10.50.1)
- Confirmação de conectividade entre as redes
- Latências baixas confirmando ambiente local

Imagem 10 - corp_net_ping.txt:
- Scan Nmap da rede corporativa (10.10.10.0/24)
- Descoberta de hosts: WS_001, WS_002, WS_003, WS_004 e analyst
- 6 hosts identificados como ativos

Imagem 11 - guest_net_ping.txt:
- Scan Nmap da rede de convidados (10.10.30.0/24)
- Descoberta de 8 hosts incluindo serviços de infraestrutura
- FTP, MySQL, Samba, OpenLDAP, Zabbix, Legacy servers identificados

Imagem 12 - infra_net_ping.txt:
- Scan Nmap da rede de infraestrutura (10.10.50.0/24)
- Descoberta de laptops pessoais e macbook
- 6 hosts identificados na rede de convidados

Imagem 13 - Execução do RustScan:
- Início do scan de portas com RustScan na rede 10.10.10.0/24
- Banner do RustScan "The Modern Day Port Scanner"
- Preparação para scan de 1000 portas

Imagem 14 - infra_net_servico_mysql-info.txt:
- Scan detalhado do MySQL server (10.10.30.11:3306)
- MySQL versão 8.0.43 identificado
- Configurações de autenticação e capacidades do servidor

Imagem 15 - infra_net_servico_ldap-rootdse.txt:
- Scan do OpenLDAP (10.10.30.17:389)
- Informações do Root DSE e contextos suportados
- Mecanismos SASL disponíveis (SCRAM-SHA-1, GSSAPI, etc.)

Imagem 16 - infra_net_servico_smb.txt:
- Scan do servidor Samba (10.10.30.15:445)
- Serviço Microsoft-DS identificado
- Porta 445/tcp aberta

Imagem 17 - Teste de conectividade HTTP:
- Tentativas de conexão HTTP para 10.10.30.117
- Primeira tentativa sem resposta, segunda com conteúdo
- Possível servidor web no Zabbix

Imagem 18 - Resumo dos hosts descobertos:
- Lista consolidada de todos os hosts nas três redes
- Status "Up" para todos os sistemas
- Mapeamento completo do ambiente

Imagem 19 - Listagem de IPs por rede:
- Extração organizada dos IPs de cada rede
- corp_net_ips.txt: 6 IPs da rede corporativa
- infra_net_ips.txt: 8 IPs da infraestrutura
- guest_net_ips.txt: 6 IPs da rede de convidados

Imagem 20 - Scan de portas com RustScan:
- Execução do RustScan nos arquivos de IPs gerados
- Geração dos arquivos de portas abertas por rede
- Preparação para análise detalhada dos serviços

Imagem 21 -  Scan inicial de descoberta de serviços na infraestrutura
- Nmap scan do FTP server (10.10.30.10) - porta 21/tcp aberta
- Nmap scan do MySQL server (10.10.30.11) - porta 3306/tcp aberta com informações detalhadas do MySQL 8.0.43
- Início do scan LDAP server (10.10.30.17) - porta 389/tcp

Imagem 22 - Continuação do scan LDAP e descoberta SMB
- Detalhes completos do servidor OpenLDAP (10.10.30.17) - porta 389/tcp com informações de schema e extensões suportadas
- Scan do Samba server (10.10.30.15) - porta 445/tcp aberta (microsoft-ds)

Imagem 23 - Teste de conectividade HTTP
- Teste curl no servidor web (10.10.30.117) - HTTP 200 OK
- Headers revelam nginx com PHP 7.3.14 e sessão PHP ativa
- Comando para salvar resposta em arquivo txt

Imagem 24 - Download de conteúdo do Zabbix
- Curl baixando página completa do Zabbix (10.10.30.117)
- 3412 bytes transferidos com sucesso

Imagem 25 - Descoberta de rede através de ARP
- Comando arp -a mostrando mapeamento de IPs para MACs
- Múltiplos hosts descobertos nas redes corporativa e de infraestrutura
- Salvamento dos resultados em recon_ip_maps.txt

Imagem 26 - Configuração de DNS
- Conteúdo do arquivo /etc/resolv.conf
- Nameserver configurado para 127.0.0.11 (Docker internal)
- Configurações baseadas no host file interno

Imagem 27 - Scan da rede corporativa (10.10.10.0/24)
- RPC service na porta 111/tcp em múltiplos hosts
- Descoberta de workstations (WS_001, WS_002, WS_003, WS_004)
- Todos os hosts com portas em estado "ignored"

Imagem 28 - Scan da rede guest/infra (10.10.30.0/24)
- Confirmação dos serviços já identificados:
- FTP (10.10.30.10), MySQL (10.10.30.11), Samba (10.10.30.15)
- OpenLDAP (10.10.30.17) com portas 389 e 636
- Zabbix web server (10.10.30.117) - porta 80/tcp
- Legacy server (10.10.30.227) - todos os ports em ignored state

Imagem 29 - Scan da rede infra estendida (10.10.50.0/24)
- RPC service na porta 111/tcp no primeiro host
- Múltiplos dispositivos cliente (notebook-carlos, laptop-vastro, laptop-luiz, macbook-aline)
- Todos os dispositivos com 1000 portas em estado "ignored"


### Observação sobre Nomenclatura
> ⚠️ **Inconsistência nos Arquivos**: Os arquivos `guest_net_ping.txt` e `infra_net_ping.txt` foram executados com redes trocadas. A correção foi aplicada baseando-se nos hostnames dos dispositivos, garantindo precisão técnica do relatório.

---

**Classificação**: CONFIDENCIAL
