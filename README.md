# dashboard-nginx-wazuh
<img width="1677" height="610" alt="image" src="https://github.com/user-attachments/assets/49c10c9d-480c-4a69-a984-5bcbe4b9a4ab" />


# Wazuh Dashboard Implementation - Nginx OCSP Monitoring

Dokumen ini menjelaskan langkah-langkah melakukan filtering log Nginx (OCSP Request) pada menu **Discover** di OpenSearch/Wazuh, menyimpannya sebagai *Saved Search*, hingga menggunakannya untuk membangun komponen visualisasi pada **Wazuh Dashboard**.

---

## 🚀 Alur Kerja Utama (Workflow)

```mermaid
graph LR
    A[Discover / OpenSearch] -->|Apply DQL Filter| B[Save Search Query]
    B -->|Import Query| C[Create Visualization]
    C -->|Assemble Panel| D[Wazuh Dashboard]

