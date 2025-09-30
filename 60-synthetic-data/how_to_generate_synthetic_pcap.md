Este guia descreve como gerar um **PCAP sintético** para demonstrar consultas/fluxos **sem** usar o PCAP do laboratório.

1) Use `tcpreplay` com tráfego próprio (sintético) em uma VM de teste.
2) Gere requisições HTTP (POST multipart) e GET para um servidor local (nginx).
3) Emita conexões TCP para uma porta alta (ex.: 8080) simulando reverse shell (somente conexão/dados inócuos).
4) Capture com `tcpdump` e analise no Wireshark/tshark.
