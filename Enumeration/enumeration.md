## Enumeration Cheatsheet


## Port Scanning

```
sudo nmap -p- 192.168.120.121   #simple nmap scan
sudo nmap -p 4505,4506 192.168.120.121 -sV #detailed scan  (Ele observou portas diferentes e, por isso, decidiu começar por elas.)
```

## HTTP Enumeration

```
curl http://192.168.120.121:8000 -v
```
