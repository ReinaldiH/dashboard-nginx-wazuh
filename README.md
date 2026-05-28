# dashboard-nginx-queryfilter-wazuh
<img width="1677" height="610" alt="image" src="https://github.com/user-attachments/assets/49c10c9d-480c-4a69-a984-5bcbe4b9a4ab" />


# Wazuh Dashboard Implementation - Nginx OCSP Monitoring

Dokumen ini menjelaskan langkah-langkah melakukan filtering log Nginx (OCSP Request) pada menu **Discover** di OpenSearch/Wazuh, menyimpannya sebagai *Saved Search*, hingga menggunakannya untuk membangun komponen visualisasi pada **Wazuh Dashboard**.

---

# Panduan Filtering Log Nginx OCSP di Wazuh Discover

Dokumen ini menjelaskan analisis kueri filter dan langkah-langkah teknis untuk menyaring log Nginx terkait lalu lintas **OCSP (Online Certificate Status Protocol)** pada menu **Discover** di Wazuh/OpenSearch.

---

## 1. Penjelasan Komponen Query Filter

<img width="1678" height="364" alt="image" src="https://github.com/user-attachments/assets/ef8d4c7a-d73f-4a22-94b1-7f2e8d817530" />

Berdasarkan filter aktif pada menu Discover, sistem menyaring log dengan parameter spesifik untuk memantau request OCSP yang sukses. Berikut adalah fungsi dari masing-masing komponen filter:

* **`location: /var/log/nginx/access.log`** Memastikan Wazuh hanya mencari dan menampilkan log yang berasal dari berkas *access log* milik web server Nginx.
* **`full_log: ocsp.privyca.id`** Menyaring log yang mengandung string `ocsp.privyca.id` di dalam isi log mentahnya (*full log*). Ini digunakan untuk memastikan *request* tersebut memang ditujukan ke endpoint/domain OCSP Anda.
* **`data.id: 200`** Memfilter HTTP Status Code yang sukses saja (`200 OK`). Respons sukses ini menandakan proses validasi status sertifikat berjalan lancar tanpa error server atau klien.
* **`data.protocol: POST`** Memfilter log yang menggunakan HTTP *Request Method* `POST`. Pada arsitektur OCSP, *request* via `POST` biasanya membawa data *binary* ASN.1 yang berisi informasi serial number sertifikat yang sedang dicek statusnya.

> **💡 Kesimpulan Fungsi Query:** Filter ini digunakan untuk memantau tren volume harian dan total *request* sukses (`HTTP 200`) yang dikirim oleh klien ke server OCSP PrivyCA melalui metode `POST`.

---

## 2. Langkah-Langkah Membuat Filter di Wazuh

Ikuti panduan berikut untuk menyusun ulang kueri filter dari awal di tab Discover:

### Langkah 1: Persiapan Menu Discover
1. Buka menu utama Wazuh, lalu pilih **Discover** (atau **Security events** > **Discover**).
2. Pastikan *Index Pattern* yang terpilih di pojok kiri atas adalah index alert Anda (misalnya: `wazuh-alerts-*`).
3. Atur *Time Range* di pojok kanan atas menjadi **Last 24 hours** (atau sesuaikan dengan kebutuhan analisis Anda).

### Langkah 2: Menambahkan Filter (UI Method)
Manfaatkan tombol **`+ Add filter`** yang terletak di bawah kolom search bar utama untuk memasukkan parameter berikut satu per satu:

1. **Filter Location:**
   * Klik **`+ Add filter`**.
   * *Field:* `location`
   * *Operator:* `is`
   * *Value:* `/var/log/nginx/access.log`
   * Klik **Save**.

2. **Filter Domain (Full Log):**
   * Klik **`+ Add filter`**.
   * *Field:* `full_log`
   * *Operator:* `contains`
   * *Value:* `ocsp.privyca.id`
   * Klik **Save**.

3. **Filter Status Code:**
   * Klik **`+ Add filter`**.
   * *Field:* `data.id` *(Catatan: Pada beberapa tipe decoder, field ini bisa berupa `data.status`)*
   * *Operator:* `is`
   * *Value:* `200`
   * Klik **Save**.

4. **Filter HTTP Method:**
   * Klik **`+ Add filter`**.
   * *Field:* `data.protocol` *(Catatan: Pada beberapa tipe decoder, field ini bisa berupa `data.method`)*
   * *Operator:* `is`
   * *Value:* `POST`
   * Klik **Save**.

---

# 3. Wazuh Dashboard Implementation - Nginx OCSP Monitoring

Dokumen ini menjelaskan langkah-langkah melakukan filtering log Nginx (OCSP Request) pada menu **Discover** di OpenSearch/Wazuh, menyimpannya sebagai *Saved Search*, hingga menggunakannya untuk membangun komponen visualisasi pada **Wazuh Dashboard**.

---

## 🚀 Alur Kerja Utama (Workflow)

```mermaid
graph LR
    A[Discover / OpenSearch] -->|Apply DQL Filter| B[Save Search Query]
    B -->|Import Query| C[Create Visualization]
    C -->|Assemble Panel| D[Wazuh Dashboard]
