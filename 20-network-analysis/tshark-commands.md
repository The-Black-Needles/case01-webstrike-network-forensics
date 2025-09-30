# Endpoints / top talkers
tshark -r capture.pcap -q -z endpoints,ip > outputs/endpoints.txt

# POSTs de upload (multipart)
tshark -r capture.pcap -Y 'http.request.method=="POST" && http contains "boundary="'   -T fields -e frame.time -e ip.src -e ip.dst -e http.request.uri > outputs/post_uploads.tsv

# Dupla extensão em upload
tshark -r capture.pcap -Y 'http contains "filename=" && http matches "\\.(jpg|png|gif)\\.php"'   -T fields -e frame.time -e http.file_data > outputs/double_ext.tsv

# Reverse shell (saída :8080)
tshark -r capture.pcap -Y 'tcp.port==8080'   -T fields -e frame.time -e ip.src -e ip.dst -e tcp.len > outputs/revshell.tsv

# Exfil por HTTP (volume)
tshark -r capture.pcap -Y 'http.request.method in {"POST","PUT"}'   -T fields -e frame.time -e ip.src -e ip.dst -e http.content_length > outputs/http_posts.tsv
