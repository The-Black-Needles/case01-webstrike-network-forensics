http.request
http.request.method == "POST"
http contains "Content-Disposition" and http contains "filename="
http.request.uri contains "/upload" or http.request.uri contains "/reviews"
tcp.port == 8080
http.response.code == 301 and http.location contains "/uploads/"
