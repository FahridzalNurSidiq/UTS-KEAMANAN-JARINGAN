<div align="center">

# 🔐 UTS Praktikum Keamanan Jaringan
**Simulasi Serangan Jaringan, Analisis Lalu Lintas, dan Kriptografi Berbasis Docker Container**

![Universitas](https://img.shields.io/badge/Universitas-Teknologi%20Bandung-blue?style=for-the-badge)
![Prodi](https://img.shields.io/badge/Prodi-Teknik%20Informatika-00599C?style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-Lab%20Environment-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![CMD](https://img.shields.io/badge/Command%20Prompt-CMD%20Tools-121011?style=for-the-badge&logo=windows-terminal&logoColor=white)

</div>

---

## 👤 Identitas Mahasiswa

| Informasi | Detail |
| :--- | :--- |
| **Nama Lengkap** | Fahridzal Nur Sidiq |
| **NIM** | `24552011315` |
| **Kelas** | TIF RP 24D CNS |
| **Program Studi** | Teknik Informatika — Universitas Teknologi Bandung |
| **Mata Kuliah** | Keamanan Jaringan (Computer Networks and Security) |
| **Paket Soal** | **Kasus 9 (`k9`)** |

---

## 📂 Struktur Direktori Repositori

Seluruh artefak praktikum, skrip analisis, konfigurasi Docker Compose, dan dokumentasi jawaban tersimpan di dalam direktori [`uts-network-lab/`](./uts-network-lab):

```text
UTS-KEAMANAN-JARINGAN/
├── README.md
└── uts-network-lab/
    ├── b/
    │   ├── b1-compose/         # Konfigurasi multi-container Docker Compose
    │   ├── b3-syn-flood/       # Simulasi serangan TCP SYN Flood & analisis handshake
    │   └── b4-dns-exfil/       # Simulasi eksfiltrasi data melalui protokol DNS
    ├── images/
    │   ├── uts-dnsmasq/        # Dockerfile server DNS (dnsmasq)
    │   ├── uts-kali-tools/     # Dockerfile Kali Linux (arpspoof, hping3, tcpdump, aircrack-ng)
    │   └── uts-openssl/        # Dockerfile utilitas enkripsi & sertifikat OpenSSL
    ├── kasus/
    │   ├── k5-scapy/           # Skrip pengiriman paket kustom menggunakan Scapy
    │   └── k9-dns-detect/      # Skrip deteksi anomali & eksfiltrasi DNS (analisis_dns.py)
    ├── Jawaban.md              # Dokumentasi log terminal & pembahasan langkah praktikum
    ├── README.md               # Identitas peserta & ringkasan kasus praktikum
    └── UTS_Keamanan Jaringan_Fahridzal... # Berkas dokumen laporan UTS
