## Enumeration Cheatsheet


## Port Scanning

```
sudo nmap -p- 192.168.120.121   # Varredura simples em todas as portas (1-65535) para identificar portas abertas
sudo nmap -p 4505,4506 192.168.120.121 -sV   # Varredura detalhada nas portas 4505 e 4506 com detecção de versão (Ele observou portas diferentes e, por isso, decidiu começar por elas.)
sudo nmap -sC -sV -p 80 192.168.120.227      # Varredura na porta 80 com scripts padrão e detecção de versão do serviço
```

## HTTP Enumeration

```
curl http://192.168.120.121:8000 -v
```

### Host Configuration

```
# Adiciona o domínio exfiltrated.offsec ao arquivo /etc/hosts local
echo "192.168.120.121 exfiltrated.offsec" | sudo tee -a /etc/hosts
```

**Nota**: Essa configuração permite que o domínio `exfiltrated.offsec` resolva para o IP `192.168.120.121` localmente, facilitando a comunicação com o alvo usando o nome de domínio. 

