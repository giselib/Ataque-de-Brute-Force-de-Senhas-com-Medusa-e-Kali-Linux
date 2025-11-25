# Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux
Implementar, documentar e compartilhar um projeto prático utilizando Kali Linux e a ferramenta Medusa, em conjunto com ambientes vulneráveis (por exemplo, Metasploitable 2 e DVWA), para simular cenários de ataque de força bruta e exercitar medidas de prevenção.

Abaixo está o passo a passo detalhado do que foi feito:
I. Configuração e Preparação do Ambiente
1. Instalação das Máquinas Virtuais: Kali Linux e o Metasploitable 2.
2. Configuração de Rede: As redes de ambas as máquinas foram configuradas para "Placa de rede exclusiva de hospedeiro" (Host-only). Isso permite que elas se comuniquem diretamente, como se estivessem em uma rede local privada, mas sem acesso à internet.
3. Criação de Snapshot: Foi criado um snapshot do Metasploitable 2 antes dos testes para permitir que a máquina fosse rapidamente restaurada ao estado original caso fosse comprometida.
4. Ambas as máquinas foram iniciadas. No Metasploitable 2, o login foi feito utilizando as credenciais padrão: msfadmin para usuário e msfadmin para senha.
5. No terminal do Metasploitable 2, foi executado o comando IP a para descobrir o IP da máquina (192.168.56.101), que seria o alvo dos ataques.
6. No Kali Linux, o comando ping -C 3 seguido do IP (192.168.56.101) do alvo foi usado para confirmar que a comunicação entre as duas máquinas estava funcionando corretamente.
   
II. Ataque de Força Bruta em Serviço FTP
O foco inicial foi o serviço FTP, simulando a auditoria de um servidor FTP antigo e potencialmente inseguro.
1. Enumeração de Serviços (NMAP): Foi utilizada a ferramenta NMAP no terminal do Kali Linux para a enumeração. O comando NMAP -sV -p 21,22,80,445,139 seguido do IP (192.168.56.101) escaneou portas comuns e tentou identificar a versão dos serviços.
2. A saída do NMAP confirmou que a porta 21 (FTP) estava aberta. Para confirmar que o FTP estava aceitando conexões, foi tentada uma conexão direta com o comando FTP seguido do IP do Metasploitable 2.
3. Criação das Wordlists: Foi necessário criar duas listas de texto simples (Wordlists) para o ataque de força bruta:
   
# Criando a lista de usuários
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

# Criando a lista de senhas
echo -e "password\n123456\nqwerty\nmsfadmin" > pass.txt

4. A ferramenta Medusa foi usada para automatizar as tentativas de login. O comando utilizado foi:

   medusa -h 192.168.56.101 -U users.txt -P pass.txt -M FTP -T 6
   
6. O Medusa encontrou a combinação válida: usuário msfadmin e senha msfadmin. A conexão foi validada manualmente no terminal com o comando FTP e as credenciais descobertas, resultando em "Login successful".

III. Ataque de Força Bruta em Formulário Web (DVWA)
A segunda parte simulou um ataque de força bruta contra formulários de login em sistemas web.
1. Foi acessado o formulário de login do sistema DVWA (Damn Vulnerable Web Application) no navegador do Kali Linux.
2. As ferramentas de desenvolvedor foram usadas para analisar a requisição POST enviada ao servidor. Foi identificado o nome dos parâmetros esperados (username, password, login) e a frase de resposta para falha (login failed), que o Medusa usa como indicador de insucesso.
3. Foi utilizado o Medusa com o módulo HTTP (-M HTTP). O comando forneceu:

   medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http
-m PAGE:'/dvwa/login.php'
-m FORM:'username=^USER^&password=^PASS^&Login=Login'
-m 'FAIL=Login failed' -t 6

4. O Medusa encontrou credenciais válidas (admin e password). O acesso foi validado logando-se no formulário web com sucesso.

IV. Enumeração SMB e Password Spray
A última parte simulou um ataque em cadeia contra o protocolo SMB, focando em password spray.
1. Para descobrir usuários válidos, foi usado comando:
   
enum4Linux -a 192.168.56.101 | tee enum4_output.txt

O comando ativou todas as técnicas de enumeração e extraiu nomes de contas reais do sistema alvo.

2. Foram criadas wordlists focadas na técnica de password spray com os comandos:

echo -e "user\nmsfadmin\nservice" > smb_users.txt
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt
    
3. O Medusa foi configurado para usar o módulo (-M SMBNT). O comando utilizou as listas criadas e tentou o login usando as poucas senhas em todos os usuários listados.
4. O Medusa obteve sucesso, encontrando novamente o usuário msfadmin e a senha msfadmin, e indicando "admin access allowed".
5. O acesso foi provado com a ferramenta smbclient. Após digitar a senha correta para o usuário msfadmin, o sistema forneceu uma lista de compartilhamentos, confirmando que o ataque foi bem-sucedido e permitiu o acesso ao sistema.
   
Esses exercícios demonstram como credenciais fracas, serviços expostos e a negligência do básico podem ser explorados por invasores usando ferramentas simples e gratuitas, como o Medusa.
