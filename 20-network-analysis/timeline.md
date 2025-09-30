- 10:14:02Z — SRC 198.51.100.25 → DST 192.0.2.10 — POST /reviews/upload.php (multipart)
- 10:14:05Z — SRC 198.51.100.25 → DST 192.0.2.10 — GET /reviews/uploads/image.jpg.php
- 10:14:07Z — 192.0.2.10 → 203.0.113.10:8080 — conexão de saída (possível reverse shell)
- 10:18:30Z — POST volumoso para https://203.0.113.10/ (443) — possível exfiltração

- 10:14:06Z — Inspeção do artefato de upload revela **padrão de reverse shell** (FIFO + /bin/sh + nc → `<ATTACKER_IP>`:8080) [generalizado]
