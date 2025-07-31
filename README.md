# 🔐 Projeto Final - Módulo 1: Fundamentos de CyberSec

**Escola:** Vai na Web  
**Curso:** Formação em Cybersegurança  
**Módulo:** 1 - Fundamentos de CyberSec  
**Autor** Bernardo Machado Muniz

## 📋 Descrição do Projeto

Este projeto consiste na implementação e análise de um laboratório de cybersegurança utilizando containers Docker, com foco em reconhecimento de rede, mapeamento de serviços e análise de vulnerabilidades em um ambiente corporativo simulado.

**⚠️ Disclaimer:** Este projeto foi desenvolvido exclusivamente para fins educacionais em ambiente controlado. Todas as técnicas apresentadas foram utilizadas apenas em sistemas próprios ou com autorização explícita.

## 🏗️ Arquitetura do Ambiente

O laboratório é composto por **3 redes segmentadas**:

### 🏢 Rede Corporativa (corp_net - 10.10.10.0/24)
- **WS_001** - Workstation 001 (10.10.10.10)
- **WS_002** - Workstation 002 (10.10.10.101)
- **WS_003** - Workstation 003 (10.10.10.127)
- **WS_004** - Workstation 004 (10.10.10.222)

### 🖥️ Rede de Infraestrutura (infra_net - 10.10.30.0/24)
- **FTP Server** (10.10.30.10) - Porta 21
- **MySQL Server** (10.10.30.11) - Portas 3306, 33060
- **Samba Server** (10.10.30.15) - Portas 139, 445
- **OpenLDAP** (10.10.30.17) - Portas 389, 636
- **Zabbix Server** (10.10.30.117) - Portas 80, 10051, 10052
- **Legacy Server** (10.10.30.227)

### 👥 Rede de Convidados (guest_net - 10.10.50.0/24)
- **notebook-carlos** (10.10.50.2)
- **laptop-vastro** (10.10.50.3)
- **laptop-luiz** (10.10.50.4)
- **macbook-aline** (10.10.50.5)

### 🕵️ Container Analyst (Kali Linux)
Container principal com acesso às 3 redes para atividades de pentest:
- **eth0**: 10.10.50.6/24 (guest_net)
- **eth1**: 10.10.30.2/24 (infra_net)
- **eth2**: 10.10.10.2/24 (corp_net)

## 🛠️ Pré-requisitos

- **Docker** (versão 20.10 ou superior)
- **Docker Compose** (versão 1.29 ou superior)
- **Git** para clonagem do repositório
- Sistema operacional Linux/macOS ou Windows com WSL2

## 🚀 Instalação e Execução

### 1. Clone o repositório do laboratório
```bash
git clone https://github.com/Kensei-CyberSec-Lab/formacao-cybersec.git
cd formacao-cybersec
```

### 2. Inicie o ambiente Docker
```bash
docker-compose up -d
```

### 3. Acesse o container analyst
```bash
docker exec -it analyst bash
```

### 4. Verifique a conectividade
```bash
# Verificar interfaces de rede
ip a | grep inet

# Testar conectividade entre redes
ping 10.10.10.1  # corp_net
ping 10.10.30.1  # infra_net  
ping 10.10.50.1  # guest_net
```

## 🔍 Atividades Realizadas

### Reconhecimento de Rede
- ✅ Mapeamento de interfaces de rede
- ✅ Descoberta de hosts ativos via ping sweep
- ✅ Identificação de hostnames e endereços MAC
- ✅ Enumeração de portas abertas com RustScan

### Análise de Serviços
- ✅ Fingerprinting de serviços críticos
- ✅ Análise do servidor MySQL 8.0.43
- ✅ Investigação do servidor web Zabbix
- ✅ Mapeamento de serviços LDAP e Samba
- ✅ Identificação de servidor FTP

### Documentação
- ✅ Relatório detalhado de findings
- ✅ Inventário técnico completo
- ✅ Mapeamento da arquitetura de rede

## 📁 Estrutura de Arquivos

```
projeto/
├── README.md
├── vnw_desafio1_inventáriotécnico.md
├── vnw_desafio1_relatório.md
```

## 🎯 Resultados Obtidos

- **20 hosts** descobertos nas 3 redes
- **14 portas abertas** identificadas na infraestrutura
- **6 serviços críticos** mapeados e analisados
- **100% conectividade** entre segmentos de rede
- **Troca de nomenclatura identificada**: guest_net (10.10.50.0/24) e infra_net (10.10.30.0/24) com nomes invertidos nos arquivos
- **Inventário completo** da infraestrutura simulada

## 🛡️ Ferramentas Utilizadas

- **Nmap** - Network discovery e port scanning
- **RustScan** - Fast port scanner
- **curl/wget** - HTTP enumeration
- **arp** - ARP table analysis
- **Docker** - Containerização do ambiente

## 📊 Entregáveis

1. **Inventário Técnico** (`vnw_desafio1_inventáriotécnico.md`)
2. **Relatório de Análise** (`vnw_desafio1_relatório.md`)
3. **Arquivos de Reconhecimento** (outputs dos scans)

## 🎓 Aprendizados

Este projeto proporcionou experiência prática em:
- Arquitetura de redes corporativas
- Técnicas de reconhecimento passivo e ativo
- Análise de serviços e identificação de vulnerabilidades
- Documentação técnica de security assessments
- Containerização de ambientes de teste


