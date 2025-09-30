**Resumo Executivo — Incidente Web (simulado)**

**Visão geral.** Upload malicioso (dupla extensão) habilitou execução remota e tentativa de exfiltração HTTP.
**Impacto.** Risco de acesso indevido e exposição de dados sensíveis.
**Evidências.** POST multipart para endpoint de upload; acesso a `*.php` em diretório de uploads; conexão de saída p/ :8080; POSTs volumosos em 443.
**Causa raiz.** Validação insuficiente + execução habilitada em diretório de uploads.
**Prioridades (24–72h).**
1) Bloquear IoCs e saídas 8080; 2) WAF para `multipart` e **dupla extensão**; 3) **no-exec** em `/uploads`; 4) tuning de regras e revisão de permissões.
