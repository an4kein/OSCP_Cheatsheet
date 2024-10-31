## Enumeration Cheatsheet


## Port Scanning

```
sudo nmap -p- 192.168.120.121   # Varredura simples em todas as portas (1-65535) para identificar portas abertas
sudo nmap -p 4505,4506 192.168.120.121 -sV   # Varredura detalhada nas portas 4505 e 4506 com detecção de versão (Ele observou portas diferentes e, por isso, decidiu começar por elas.)
sudo nmap -sC -sV -p 80 192.168.120.227      # Varredura na porta 80 com scripts padrão e detecção de versão do serviço
sudo nmap -p 2181,8080 -A 192.168.120.227
sudo nmap -p22,80 -sC -sV -oA nmap/gravity 192.168.145.160
```

## HTTP Enumeration

### Initial Diagnostics

```
curl http://192.168.120.121:8000 -v                # Realiza uma requisição HTTP na porta 8000 com saída detalhada, exibindo cabeçalhos e resposta do servidor
```

### Silent Enumeration

```
curl http://exfiltrated.offsec/ -s | html2text | tail   # Realiza uma requisição HTTP silenciosa e converte o HTML em texto simples, exibindo as últimas linhas da resposta
curl http://exfiltrated.offsec/ -s | grep login   # Realiza uma requisição HTTP silenciosa e filtra o conteúdo retornado, exibindo apenas linhas que contenham a palavra "login"
curl http://exfiltrated.offsec/ -s | grep dashboard   # Realiza uma requisição HTTP silenciosa e filtra o conteúdo retornado, exibindo apenas linhas que contenham a palavra "dashboard"
curl http://exfiltrated.offsec/panel/ -s | html2text | tail   # Realiza uma requisição HTTP silenciosa para o endpoint "/panel", converte o HTML em texto e exibe as últimas linhas da resposta
```

**Nota**: Essa abordagem permite uma enumeração rápida e discreta da aplicação, identificando links importantes e a versão do CMS com baixo risco de deixar rastros. Dessa forma, é possível mapear pontos de entrada e possíveis vulnerabilidades sem gerar tráfego visível no navegador.

### Host Configuration

```
# Adiciona o domínio exfiltrated.offsec ao arquivo /etc/hosts local
echo "192.168.120.121 exfiltrated.offsec" | sudo tee -a /etc/hosts
```

**Nota**: Essa configuração permite que o domínio `exfiltrated.offsec` resolva para o IP `192.168.120.121` localmente, facilitando a comunicação com o alvo usando o nome de domínio. 

## Google Dorks para Pesquisa de Exploits

### 1. Básico de Exploits e Vulnerabilidades
- `application_name exploit` – Pesquisa geral para exploits da aplicação.
- `application_name version exploit` – Exploits específicos para uma versão.
- `application_name vulnerability` – Vulnerabilidades conhecidas da aplicação.

### 2. Sites de Exploits e Documentação Técnica
- `site:exploit-db.com application_name` – Busca exploits no Exploit-DB.
- `site:packetstormsecurity.com application_name` – Busca exploits no PacketStorm.
- `site:github.com application_name exploit` – Repositórios no GitHub que podem conter scripts de exploração.
- `site:nvd.nist.gov application_name` – Vulnerabilidades catalogadas na NVD (National Vulnerability Database).

### 3. Buscando Vulnerabilidades em Aplicações Específicas
- `application_name CVE` – Lista possíveis CVEs associados à aplicação.
- `application_name version CVE` – CVEs de uma versão específica.
- `application_name security advisory` – Consultas para encontrar boletins de segurança da aplicação.

### 4. Documentação Técnica e API
- `application_name documentation` – Documentação oficial ou manuais.
- `application_name API reference` – Referência de API para entender endpoints potencialmente exploráveis.

### 5. Fóruns e Discussões
- `site:stackoverflow.com application_name exploit` – Perguntas no Stack Overflow sobre possíveis vulnerabilidades.
- `site:reddit.com application_name exploit` – Discussões no Reddit sobre vulnerabilidades.
- `site:securityfocus.com application_name` – Exploits e discussões sobre segurança no SecurityFocus.

### 6. Identificação de Painéis de Administração e Interfaces
- `application_name admin login` – Identificar possíveis páginas de login ou de administração.
- `application_name intitle:"index of" "admin"` – Páginas de diretórios abertos com "admin".
- `application_name "version" inurl:login` – Identifica logins específicos de versões da aplicação.

### 7. Busca em Repositórios de Vulnerabilidades
- `site:cvedetails.com application_name` – Para encontrar vulnerabilidades detalhadas.
- `site:rapid7.com database application_name` – Vulnerabilidades e exploits da Rapid7.
- `site:snyk.io vulnerabilities application_name` – Busca vulnerabilidades da aplicação no Snyk.

### Exemplo Prático

Se você identificou o **Exhibitor para Zookeeper**, use dorks como:

- `site:exploit-db.com Exhibitor Zookeeper exploit`
- `site:nvd.nist.gov Exhibitor Zookeeper CVE`
- `Exhibitor Zookeeper unauthenticated command injection`
- `Exhibitor Zookeeper "vulnerability" OR "exploit"`

Durante o OSCP, sempre comece as pesquisas com o nome e a versão da aplicação, caso disponível. Isso restringe os resultados e aumenta a chance de encontrar exploits específicos e aplicáveis.

