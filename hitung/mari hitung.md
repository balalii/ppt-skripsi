# Panduan Menjelaskan Simulasi Perhitungan Rumus Skripsi

**Pendamping berkas:** `simulasi_perhitungan_rumus_skripsi.xlsx`
**Untuk:** Iqbal Ali Ar-Ridho (2022150078) — sinkron dengan `skripsi_final_v2.docx`

---

## Inti yang harus Anda pahami dulu (penting!)

Berkas Excel ini berisi **13 rumus** skripsi, masing-masing di sheet sendiri. Bedanya dengan versi lama: **semua sheet sekarang SALING TERHUBUNG**, seperti rantai. Keluaran satu sheet otomatis jadi masukan sheet berikutnya.

Bayangkan seperti **air mengalir dari hulu ke hilir**:

```
PARAMETER (sumber air: k, bobot, ambang, N, R)
   │  menyuplai semua sheet
   ▼
RRF → Normalisasi → Reranking → Context Precision ┐
                                                   ├─► Rata-rata per Run → Skor Akhir → Persentase
Faithfulness ──────────────────────────────────────┘
Answer Relevancy ──────────────────────────────────┘
```

Artinya: kalau Anda **ubah satu angka di sheet PARAMETER** (misal k dari 60 jadi 50), maka seluruh rantai ikut berubah otomatis sampai ke skor akhir. Inilah yang membuktikan ke dosen bahwa Anda paham rumusnya, bukan sekadar menempel angka.

**Hanya satu sheet yang sengaja dipisah:** `4.4 Black Box`. Sheet itu menguji *fungsi* sistem (jalan/tidak), bukan *kualitas jawaban*, jadi tidak nyambung ke rantai RAGAS.

---

## Arti warna sel (hafalkan ini saja)

| Warna | Arti | Boleh diubah? |
|---|---|---|
| 🟡 Kuning | Input manual | Ya |
| 🔵 Biru | Nilai **ditarik dari sheet lain** | Jangan diketik ulang |
| 🟠 Oranye | Hasil akhir perhitungan | Otomatis |
| 🟢 Hijau | Kotak rumus | Otomatis |

Aturan sederhana: **kalau sel biru, jangan diutak-atik** — itu hasil dari sheet sebelumnya. Yang boleh diubah hanya yang kuning.

---

## Cara demonstrasi 30 detik ke dosen

1. Buka sheet **PARAMETER**, tunjuk sel kuning `k = 60`.
2. Buka sheet **4.10 Skor Akhir**, lihat skor akhir (misal context precision 0,7991).
3. Kembali ke PARAMETER, ubah satu angka (misal bobot w0 dari 4 jadi 3).
4. Buka lagi 4.10 — angkanya **berubah sendiri**. Selesai. Itu bukti rantai berfungsi.

(Setelah demo, kembalikan angka ke semula.)

---

## Penjelasan tiap sheet (urut dari hulu ke hilir)

### PARAMETER — sumber semua angka
Semua konstanta penting ada di sini: k=60, bobot 4:1, ambang reranking 0,40, N=10, R=3. Tiap sheet lain menarik dari sini. **Ini jantung workbook.**

### 2.1 RRF Dasar — `RRFscore(d) = Σ 1/(k+r(d))`
Skor dokumen dari posisi peringkatnya. Versi teori (Bab 2), peringkat mulai 1. *Menarik:* k dari PARAMETER. *Dasar bagi:* sheet 4.1.

### 2.2 RRF Berbobot — `RRFw(d) = Σ wi·1/(k+ri(d))`
Sama, tapi tiap daftar diberi bobot. *Menarik:* k & bobot dari PARAMETER.

### 4.1 RRF Implementasi — `scoreRRF(d) = Σ wi·1/(k+ri(d)+1)` ⭐
Rumus RRF yang BENAR-BENAR dipakai sistem (peringkat mulai 0, jadi ada "+1"). Berisi simulasi penuh Tabel 4.7→4.8. *Menarik:* parameter dari PARAMETER. *Mengalir ke:* sheet 4.2 (skor mentah D-A & skor maks). **Angka kunci: D-A = 0,09783.**

> Hafalkan alur D-A: peringkat (0,1,1) → 4·(1/61) + 1·(1/62) + 1·(1/62) = 0,09783.

### 4.2 Normalisasi — `min( skor / skor_maks , 1 )`
Ubah skor mentah ke rentang 0–1. *Menarik:* skor mentah & skor maks **dari sheet 4.1**. **Angka kunci: 0,09783/0,09836 = 0,9946.**

### 4.3 Sigmoid Rerank — `σ(s) = 1/(1+e^−s)`
Ubah skor mentah cross-encoder (logit) ke 0–1, lalu saring ambang 0,40 → menghasilkan **5 dokumen konteks final** beserta indikator relevansi v_k. *Menarik:* ambang dari PARAMETER. *Mengalir ke:* sheet 4.7 (kolom v_k). **Pola v_k = (1,0,1,0,0).**

### 4.5 Faithfulness — `C(didukung) / C(total)`
Rasio klaim jawaban yang didukung konteks. *Mengalir ke:* sheet 4.9 (Pertanyaan ke-6). **Angka kunci: 9/11 = 0,8182.**

### 4.6 Answer Relevancy — `(1/N)·Σ cos(E(gi),E(o))`
Rata-rata kemiripan pertanyaan tiruan vs asli. *Menarik:* N dari PARAMETER. *Mengalir ke:* sheet 4.9 (Pertanyaan ke-3). **Angka kunci: (0,8612+0,8455+0,8250)/3 = 0,8439.**

### 4.7 Precision per-k — `Precision(k) = TP(k)/(TP(k)+FP(k))`
Presisi kumulatif tiap peringkat. *Menarik:* v_k **dari sheet 4.3**. *Mengalir ke:* sheet 4.8. **Hasil: 1; 0,5; 0,667; 0,5; 0,4.**

### 4.8 Context Precision — `Σ(Precision(k)·v_k) / Σ v_k`
Ketepatan peringkat dokumen relevan. *Menarik:* Precision_k & v_k **dari sheet 4.7**. *Mengalir ke:* sheet 4.9 (Pertanyaan ke-3). **Angka kunci: 1,6667/2 = 0,8333.**

### 4.9 Rata-rata per Run — `S̄(m) = (1/N)·Σ s(m,i)`
Rata-rata skor 10 pertanyaan (run ke-3). Pertanyaan ke-6 faithfulness, ke-3 AR & CP **ditarik dari sheet 4.5/4.6/4.8**; sisanya data asli. *Mengalir ke:* sheet 4.10. **Hasil: faith 0,9818 · AR 0,9146 · CP 0,8333.**

### 4.10 Skor Akhir — `S̄(m)akhir = (1/R)·Σ S̄(m,r)`
Rata-rata 3 run. Run ke-3 **ditarik dari sheet 4.9**; run 1 & 2 input. *Mengalir ke:* Persentase. **Hasil akhir: faith 0,9812 · AR 0,9146 · CP 0,7991.**

### Persentase Peningkatan — `Δ = (Enhanced−Baseline)/Baseline × 100%`
Bandingkan Enhanced vs Baseline. *Menarik:* nilai Enhanced **dari sheet 4.10**. **Hasil: AR +12,40% · CP +14,16% · faith −1,13%.**

### 4.4 Black Box (TERPISAH) — `(Σ berhasil/Σ total) × 100%`
Tingkat keberhasilan fungsional = 17/17 = 100%. **Dipisah** karena menguji fungsi, bukan kualitas jawaban.

---

## Peta koneksi singkat ("kalau ini berubah, itu ikut berubah")

| Ubah di sini | Yang ikut berubah |
|---|---|
| PARAMETER → k atau bobot | 2.1, 2.2, 4.1 → 4.2, dan seterusnya |
| PARAMETER → ambang 0,40 | 4.3 (lolos/tidaknya dokumen) |
| 4.1 → peringkat dokumen | 4.2 (normalisasi) |
| 4.3 → v_k dokumen | 4.7 → 4.8 (context precision) → 4.9 → 4.10 → Persentase |
| 4.5 / 4.6 → input klaim/cosine | 4.9 → 4.10 → Persentase |
| 4.10 → skor akhir | Persentase Peningkatan |

---

## Antisipasi pertanyaan dosen (jawaban singkat)

**T: Kenapa k = 60?** Mengikuti paper asli RRF (Cormack dkk., 2009). k besar membuat peringkat menengah tetap dapat skor kompetitif.

**T: Kenapa ada "+1" di rumus 4.1?** Karena peringkat di program mulai 0, sedangkan rumus asli mulai 1. "+1" menyetarakan keduanya.

**T: Kenapa bobot 4:1?** Pertanyaan asli adalah jangkar maksud pengguna. Dengan 1 asli + 2 variasi, rasio 4:1 membuat asli (4) dua kali lipat total variasi (2) — dominan tapi variasi tetap berperan. Didukung uji parameter Tabel 4.6.

**T: Bagaimana context precision dihitung?** (Buka sheet 4.8) Dari pola relevansi 5 dokumen (1,0,1,0,0): jumlahkan Precision×v_k lalu bagi jumlah dokumen relevan → 1,6667/2 = 0,8333. Dokumen relevan kedua di peringkat 3 (bukan 2) menurunkan skor.

**T: Kenapa faithfulness sedikit turun di Enhanced?** Turun 0,0112 poin saja (tetap "Sangat Baik"). Konteks lebih beragam sedikit menaikkan peluang melenceng, tapi AR & CP naik signifikan.

**T: Apakah peningkatan ini kontribusi RRF saja?** Bukan. Mode Enhanced memuat tiga teknik sekaligus, jadi peningkatan adalah kontribusi gabungan. Pemisahan kontribusi tiap teknik disarankan sebagai penelitian lanjutan (Bab V).

**T: Coba hitung ulang di depan saya.** Buka sheet PARAMETER, ubah satu angka, buka sheet hasil — angkanya berubah otomatis. Lalu jelaskan alurnya.
