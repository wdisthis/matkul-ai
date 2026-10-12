# Materi AI — Pertemuan 6 & 7
**Pertemuan 6:** First-Order Logic & Inferensi (Unification, Forward/Backward Chaining)
**Pertemuan 7:** Logika Fuzzy (Tsukamoto, Mamdani, Sugeno)

> Catatan: semua soal di slide (latihan, Tugas 1, Tugas 2, mesin cuci) sudah dikerjakan **langkah demi langkah** di bawah. Bagian yang saya temukan tidak cocok dengan angka di slide ditandai ⚠️.

---

## DAFTAR ISI

**Bagian A — Pertemuan 6**
1. Propositional Logic vs First-Order Logic
2. Unsur-unsur FOL dan Kuantor
3. Representasi Pengetahuan dengan FOL
4. Inferensi dengan FOL (Model Checking vs Proof)
5. Unification
6. Forward Chaining vs Backward Chaining
7. Forward Chaining (algoritma + contoh + latihan)
8. Backward Chaining (algoritma + contoh + latihan)
9. Tugas 1 (Investasi IBM)
10. Tugas 2 (Budi & Dewi)
11. Contoh Implementasi: Sistem Pakar Diabetes

**Bagian B — Pertemuan 7**
12. Konsep Logika Fuzzy
13. Teori Dasar (variabel, himpunan, semesta, domain)
14. Fungsi Keanggotaan (+ contoh hitung)
15. Operasi Himpunan Fuzzy
16. Sistem Inferensi Fuzzy
17. Studi Kasus Mesin Cuci: Mamdani, Tsukamoto, Sugeno
18. Perbandingan Tiga Metode
19. Latihan Soal Tambahan + Kunci
20. Cheat Sheet Kuis

---

# BAGIAN A — PERTEMUAN 6

## 1. Propositional Logic vs First-Order Logic

| Aspek | Propositional Logic (PL) | First-Order Logic (FOL) |
|---|---|---|
| Yang dinyatakan | Fakta benar/salah (p, q, r) | Objek, relasi, fungsi, kuantor |
| Struktur internal | Tidak ada (satu kalimat = satu simbol) | Ada (predikat dengan argumen) |
| Contoh | `p` = "John adalah raja" | `King(John)` |
| Pernyataan umum ("semua…") | Tidak bisa ringkas | Bisa: `∀x King(x) ∧ Greedy(x) ⇒ Evil(x)` |

**Intinya:** FOL = PL + **objek** + **relasi (predikat)** + **fungsi** + **kuantor (∀, ∃)**.

---

## 2. Unsur-unsur FOL

| No | Unsur | Arti | Contoh |
|---|---|---|---|
| 1 | Konstanta | Objek spesifik | John, Mary, 2, a |
| 2 | Variabel | Simbol umum | x, y, z |
| 3 | Fungsi | Memetakan objek ke objek | `FatherOf(x)` |
| 4 | Predikat | Sifat atau relasi (hasilnya benar/salah) | `Loves(x,y)`, `GreaterThan(x,y)` |
| 5 | Konektor logika | ¬ (not), ∧ (and), ∨ (or), ⇒ (implikasi), ⇔ (bi-implikasi) | |
| 6 | Kuantor | ∀ (untuk semua), ∃ (ada paling sedikit satu) | |

**Perbedaan fungsi vs predikat** (sering keluar di kuis):
- **Fungsi** menghasilkan **objek** → `FatherOf(John)` = objek (misal Bob).
- **Predikat** menghasilkan **benar/salah** → `Loves(John, Mary)` = True/False.

### Kuantor
- **Universal** `∀x P(x)` → untuk semua x berlaku P.
- **Eksistensial** `∃x P(x)` → ada paling sedikit satu x yang P.

### Aturan emas penerjemahan kalimat → FOL
| Kuantor | Konektor yang dipakai | Bentuk |
|---|---|---|
| `∀` | **⇒** (implikasi) | `∀x (A(x) ⇒ B(x))` |
| `∃` | **∧** (dan) | `∃x (A(x) ∧ B(x))` |

### Contoh soal penerjemahan (dengan langkah)

**Soal 1:** "Semua mahasiswa itu pintar."
1. Kata "semua" → kuantor `∀`.
2. Himpunan yang dibicarakan: mahasiswa → predikat `Mahasiswa(x)`.
3. Sifatnya: pintar → `Pintar(x)`.
4. `∀` dipasangkan dengan `⇒`.
5. **Jawab:** `∀x Mahasiswa(x) ⇒ Pintar(x)`

**Soal 2:** "Ada mahasiswa yang pintar."
1. "Ada" → `∃`.
2. `∃` dipasangkan dengan `∧`.
3. **Jawab:** `∃x Mahasiswa(x) ∧ Pintar(x)`

**Kesalahan umum (jebakan kuis):**
- `∀x Mahasiswa(x) ∧ Pintar(x)` ❌ artinya "semua benda di dunia adalah mahasiswa dan pintar".
- `∃x Mahasiswa(x) ⇒ Pintar(x)` ❌ terlalu lemah: benar kalau ada satu benda yang bukan mahasiswa.

**Soal 3:** "Setiap raja yang serakah adalah jahat."
`∀x King(x) ∧ Greedy(x) ⇒ Evil(x)`

**Soal 4:** "John bersaudara dengan Richard."
`Brother(John, Richard)`

---

## 3. Representasi Pengetahuan dengan FOL

FOL dipakai untuk:
1. **Menyatakan fakta:** `Brother(John, Richard)`
2. **Aturan umum:** `∀x King(x) ∧ Greedy(x) ⇒ Evil(x)`
3. **Menjawab query dengan inferensi.**

---

## 4. Inferensi dengan FOL

Inferensi = menghasilkan **kesimpulan** dari knowledge base (KB). Dua pendekatan:

1. **Model checking:** coba semua kemungkinan interpretasi → **tidak efisien**.
2. **Proof procedures:** pakai algoritma inferensi (unification, forward chaining, backward chaining, dll).

---

## 5. Unification

**Definisi:** proses **mencocokkan ekspresi logika** dengan menemukan **substitusi variabel** agar kedua ekspresi menjadi **sama**.

Notasi substitusi: `{x/Mary}` dibaca "variabel x diganti Mary".

### Algoritma sederhana UNIFY(p, q)
1. Jika p dan q **identik** → berhasil, substitusi kosong `{}`.
2. Jika salah satu **variabel** → ganti variabel dengan pasangannya (cek: variabel tidak boleh muncul di dalam pasangannya sendiri / *occurs check*).
3. Jika keduanya **predikat/fungsi**: nama dan jumlah argumen harus sama, lalu unify argumen **dari kiri ke kanan**, **terapkan substitusi yang sudah ada** ke argumen sisanya.
4. Selain itu → **gagal**.

### Contoh dengan langkah rinci

**Contoh 1 (dari slide):** Query `Knows(John, x)` vs Fakta `Knows(John, Mary)`
1. Predikat sama (`Knows`), jumlah argumen sama (2) ✓.
2. Argumen 1: `John` vs `John` → identik ✓.
3. Argumen 2: `x` vs `Mary` → x variabel → `x/Mary`.
4. **Hasil:** `{x/Mary}` → query terpenuhi.

**Contoh 2:** `Knows(John, x)` vs `Knows(y, Bill)`
1. Arg 1: `John` vs `y` → `y/John`.
2. Arg 2: `x` vs `Bill` → `x/Bill`.
3. **Hasil:** `{y/John, x/Bill}`.

**Contoh 3 (ada fungsi):** `Knows(John, x)` vs `Knows(y, Mother(y))`
1. Arg 1: `John` vs `y` → `y/John`.
2. Terapkan ke arg 2 pasangan: `Mother(y)` → `Mother(John)`.
3. Arg 2: `x` vs `Mother(John)` → `x/Mother(John)`.
4. **Hasil:** `{y/John, x/Mother(John)}`.

**Contoh 4 (gagal):** `Knows(John, x)` vs `Knows(x, Elizabeth)`
1. Arg 1: `John` vs `x` → `x/John`.
2. Arg 2: `x` → `John`, bandingkan `John` vs `Elizabeth` → dua konstanta berbeda → **GAGAL**.
3. Penyebab: dua kalimat memakai nama variabel yang sama padahal tidak berhubungan. Perbaikan: *standardizing apart*, ubah menjadi `Knows(x17, Elizabeth)` → hasil `{x/Elizabeth, x17/John}`.

**Contoh 5 (gagal):** `P(a)` vs `P(b)` → konstanta berbeda → gagal.
**Contoh 6 (gagal):** `P(x)` vs `Q(x)` → nama predikat berbeda → gagal.
**Contoh 7 (occurs check):** `x` vs `F(x)` → x muncul di dalam `F(x)` → gagal.

### Generalized Modus Ponens (unification dalam inferensi)

KB:
- `King(John)`, `Greedy(John)`
- `∀x King(x) ∧ Greedy(x) ⇒ Evil(x)`

Langkah:
1. Unify premis aturan dengan fakta: `King(x)` vs `King(John)` → `{x/John}`.
2. Terapkan ke `Greedy(x)` → `Greedy(John)` → ada di KB ✓.
3. Terapkan `{x/John}` ke kesimpulan `Evil(x)`.
4. **Kesimpulan:** `Evil(John)`.

---

## 6. Forward Chaining vs Backward Chaining

| Aspek | Forward Chaining | Backward Chaining |
|---|---|---|
| Arah | Dari **fakta → kesimpulan** | Dari **goal → fakta** |
| Jenis penalaran | **Data-driven** | **Goal-driven** |
| Mulai dari | Fakta yang diketahui | Query/goal (bagian THEN) |
| Cocok untuk | KB besar dengan banyak fakta | Membuktikan satu hipotesis tertentu |
| Contoh pemakaian | Sistem pakar berbasis aturan (rule-based expert system) | Prolog |
| Struktur bantu | Database fakta baru | **Stack** subgoal |

---

## 7. Forward Chaining

### Konsep
Teknik pencarian yang dimulai dari **fakta yang diketahui**, lalu mencocokkan fakta dengan **bagian IF** dari rule IF-THEN.

### Aturan main (dari slide)
1. Jika fakta cocok dengan bagian IF → rule **dieksekusi**.
2. Rule dieksekusi → **fakta baru (bagian THEN)** ditambahkan ke database.
3. Setiap putaran pencocokan **dimulai dari rule paling atas**.
4. **Setiap rule hanya boleh dieksekusi sekali.**
5. Berhenti bila **tidak ada lagi rule yang bisa dieksekusi** (atau goal sudah terbukti).

### Contoh dari slide (dikerjakan rinci)

**Rules:**
- R1: IF (Y AND D) THEN Z
- R2: IF (X AND B AND E) THEN Y
- R3: IF A THEN X
- R4: IF C THEN L
- R5: IF (L AND M) THEN N

**Fakta awal:** A, B, C, D, E
**Goal:** Apakah Z benar?

**Cara A — aturan "mulai dari rule teratas, eksekusi rule pertama yang bisa":**

| Iterasi | Pengecekan (dari atas) | Rule dieksekusi | Fakta baru |
|---|---|---|---|
| 1 | R1: Y ada? ✗. R2: X ada? ✗. R3: A ada ✓ | **R3** | X |
| 2 | R1: Y ✗. R2: X ✓, B ✓, E ✓ | **R2** | Y |
| 3 | R1: Y ✓, D ✓ | **R1** | Z |

Z terbukti → **Goal BENAR**. (R4 dan R5 tidak diperlukan.)

**Cara B — urutan di slide:** R3 (X) → R4 (L) → R2 (Y) → R1 (Z).
Slide mengeksekusi R4 lebih dulu sebelum R2 dan databasenya menjadi `X, L, Y, Z`. **Hasil akhirnya sama:** Z benar. Rule R5 tidak pernah jalan karena M tidak ada di fakta.

> Tips kuis: kalau soal tidak menyebut urutan, tulis urutan iterasi yang konsisten (top-down) dan sebutkan hasil akhir benar/salahnya. Urutan eksekusi boleh berbeda, kesimpulan tidak boleh berbeda.

### Latihan Forward Chaining (slide 19) — jawaban rinci

**Rules:**
- R1: IF A THEN B
- R2: IF C THEN D
- R3: IF B AND (P OR L) THEN H
- R4: IF H AND M THEN K
- R5: IF L OR M THEN P

**Fakta awal:** A, C, M
**Goal:** Apakah K bernilai TRUE?

| Iter | Pengecekan (dari atas) | Rule dieksekusi | Fakta baru | Database sesudahnya |
|---|---|---|---|---|
| 1 | R1: A ada ✓ | **R1** | B | A, C, M, B |
| 2 | R1 sudah dipakai. R2: C ada ✓ | **R2** | D | A, C, M, B, D |
| 3 | R3: B ✓, (P ✗ OR L ✗) → ✗. R4: H ✗. R5: L ✗ OR M ✓ → ✓ | **R5** | P | A, C, M, B, D, P |
| 4 | R3: B ✓, (P ✓ OR L ✗) → ✓ | **R3** | H | A, C, M, B, D, P, H |
| 5 | R4: H ✓, M ✓ | **R4** | K | A, C, M, B, D, P, H, K |

**Kesimpulan:** K ∈ database → **K bernilai TRUE**.

Poin penting: R3 awalnya gagal (iterasi 3) karena `P OR L` belum terpenuhi. Setelah R5 menghasilkan P, R3 baru bisa jalan.

---

## 8. Backward Chaining

### Konsep
Kebalikan forward chaining: **diawali dari goal** (bagian THEN dari rule), lalu mencari apakah premis (bagian IF) terpenuhi oleh fakta.

### Aturan main (dari slide)
1. Mulai dari **Goal** → cari rule yang bagian **THEN**-nya = goal.
2. Cek premis di bagian **IF**:
   - Premis **cocok dengan fakta** → rule dieksekusi, bagian THEN masuk database sebagai **fakta baru**.
   - Premis **tidak cocok** → simpan premis itu ke **stack sebagai subgoal**, lalu buktikan subgoal itu dengan cara yang sama (rekursif).
3. **Berhenti** jika goal terbukti, atau tidak ada rule yang bisa membuktikan subgoal/goal.

### Contoh dari slide (dikerjakan rinci)

**Rules:**
- R1: IF (Y AND D) THEN Z
- R2: IF (X AND B AND E) THEN Y
- R3: IF A THEN X
- R4: IF C THEN L
- R5: IF (L AND M) THEN N

**Fakta:** A, B, C, D, E  **Goal:** Z?

| Langkah | Stack | Yang dilakukan | Fakta baru |
|---|---|---|---|
| 1 | Z | Cari rule dengan THEN = Z → **R1**. Premis: Y, D. D ada di fakta ✓. Y tidak ada ✗ → push Y | – |
| 2 | Z, Y | Cari rule THEN = Y → **R2**. Premis: X, B, E. B ✓, E ✓. X tidak ada ✗ → push X | – |
| 3 | Z, Y, X | Cari rule THEN = X → **R3**. Premis: A ada ✓ → R3 dieksekusi | **X** |
| 4 | Z, Y | pop X. Kembali ke R2: X ✓, B ✓, E ✓ → R2 dieksekusi | **Y** |
| 5 | Z | pop Y. Kembali ke R1: Y ✓, D ✓ → R1 dieksekusi | **Z** |
| 6 | – | Stack kosong | Goal terbukti |

**Kesimpulan:** Z **BENAR**. Database akhir: A, B, C, D, E + X, Y, Z. (R4 dan R5 tidak disentuh.)

> Perhatikan: pada backward chaining, hanya rule yang **relevan dengan goal** yang diperiksa. Ini yang membuatnya efisien untuk membuktikan satu hipotesis.

### Latihan Backward Chaining (slide 28) — jawaban rinci

**Rules:**
- R1: (A ∧ G) → C
- R2: C → E
- R3: (D ∧ I ∧ G) → F
- R4: (A ∧ D) → G
- R5: ((I ∧ F) ∨ G) → B
- R6: (E ∧ (B ∨ A)) → I
- R7: (C ∧ F) → H

**Fakta awal:** A, D  **Goal:** H

**Peta ketergantungan (dari goal ke bawah):**
```
H  ← R7: C ∧ F
 ├─ C ← R1: A ∧ G
 │    └─ G ← R4: A ∧ D  (A ✓, D ✓ fakta)
 └─ F ← R3: D ∧ I ∧ G
      ├─ D  fakta ✓
      ├─ G  (sudah terbukti)
      └─ I ← R6: E ∧ (B ∨ A)
           ├─ E ← R2: C  (C sudah terbukti)
           └─ (B ∨ A): A fakta ✓ → cukup, B tidak perlu dicari
```

| Langkah | Stack | Yang dilakukan | Fakta baru |
|---|---|---|---|
| 1 | H | Rule THEN=H → **R7**: butuh C dan F. Keduanya bukan fakta → push C lalu F (kerjakan C dulu) | – |
| 2 | H, C | Rule THEN=C → **R1**: butuh A ✓ dan G ✗ → push G | – |
| 3 | H, C, G | Rule THEN=G → **R4**: A ✓, D ✓ → R4 dieksekusi | **G** |
| 4 | H, C | pop G. R1: A ✓, G ✓ → R1 dieksekusi | **C** |
| 5 | H, F | pop C. Sekarang buktikan F. Rule THEN=F → **R3**: butuh D ✓, G ✓ (sudah ada), I ✗ → push I | – |
| 6 | H, F, I | Rule THEN=I → **R6**: butuh E ✗ dan (B ∨ A). Push E | – |
| 7 | H, F, I, E | Rule THEN=E → **R2**: butuh C ✓ (sudah terbukti) → R2 dieksekusi | **E** |
| 8 | H, F, I | pop E. R6: E ✓, (B ∨ A): A ✓ fakta → OR terpenuhi → R6 dieksekusi | **I** |
| 9 | H, F | pop I. R3: D ✓, I ✓, G ✓ → R3 dieksekusi | **F** |
| 10 | H | pop F. R7: C ✓, F ✓ → R7 dieksekusi | **H** |

**Kesimpulan:** H **TERBUKTI (benar)**. Fakta baru berurutan: **G, C, E, I, F, H**. R5 tidak dipakai.

---

## 9. TUGAS 1 — Investasi Saham IBM

**Rules:**
- R1: IF memiliki uang $10.000 AND pendidikan College THEN berinvestasi di **Sekuritas**
- R2: IF pendapatan ≥ $40.000/tahun AND pendidikan College THEN berinvestasi di **Saham Pertumbuhan**
- R3: IF usia < 30 AND investasi di **Sekuritas** THEN sebaiknya **Saham Pertumbuhan**
- R4: IF usia < 30 (atau > 22) THEN pendidikan **College**
- R5: IF ingin **Saham Pertumbuhan** THEN saham yang dipilih = **IBM**

**Database:** punya uang $10.000; usia < 30
**Goal:** Apakah tepat berinvestasi pada saham IBM?

Singkatan: **U** = uang $10.000, **Um** = usia < 30, **Pd** = Pendidikan College, **Sk** = Sekuritas, **Gr** = Saham Pertumbuhan, **Inc** = Pendapatan ≥ $40.000, **IBM** = saham IBM.

### a) Forward Chaining

Fakta awal: **U, Um**

| Iter | Pengecekan (dari atas) | Rule dieksekusi | Fakta baru | Database |
|---|---|---|---|---|
| 1 | R1: U ✓ tapi Pd ✗ → ✗. R2: Inc ✗. R3: Um ✓ tapi Sk ✗ → ✗. R4: Um ✓ | **R4** | Pd | U, Um, Pd |
| 2 | R1: U ✓, Pd ✓ | **R1** | Sk | U, Um, Pd, Sk |
| 3 | R1 sudah. R2: Inc ✗ → ✗. R3: Um ✓, Sk ✓ | **R3** | Gr | U, Um, Pd, Sk, Gr |
| 4 | R4 sudah. R5: Gr ✓ | **R5** | IBM | U, Um, Pd, Sk, Gr, IBM |

**Kesimpulan:** IBM terbukti → **Ya, tepat berinvestasi pada saham IBM.**

### b) Backward Chaining

Goal: **IBM**

| Langkah | Stack | Yang dilakukan | Fakta baru |
|---|---|---|---|
| 1 | IBM | Rule THEN=IBM → **R5**: butuh Gr ✗ → push Gr | – |
| 2 | IBM, Gr | Rule THEN=Gr ada dua: R2 dan R3. Coba **R2** dulu: butuh Inc dan Pd. Inc bukan fakta dan tidak ada rule yang menghasilkan Inc → **R2 gagal** (backtrack) | – |
| 3 | IBM, Gr | Coba **R3**: butuh Um ✓ (fakta) dan Sk ✗ → push Sk | – |
| 4 | IBM, Gr, Sk | Rule THEN=Sk → **R1**: butuh U ✓ dan Pd ✗ → push Pd | – |
| 5 | IBM, Gr, Sk, Pd | Rule THEN=Pd → **R4**: Um ✓ → dieksekusi | **Pd** |
| 6 | IBM, Gr, Sk | pop Pd. R1: U ✓, Pd ✓ → dieksekusi | **Sk** |
| 7 | IBM, Gr | pop Sk. R3: Um ✓, Sk ✓ → dieksekusi | **Gr** |
| 8 | IBM | pop Gr. R5: Gr ✓ → dieksekusi | **IBM** |

**Kesimpulan:** IBM **terbukti** → investasi saham IBM **tepat**.

> Catatan: R4 di slide tertulis "usia < 30 **atau** > 22". Karena fakta kita "usia < 30", syarat OR sudah terpenuhi. (Maksud sebenarnya kemungkinan rentang 22–30 tahun.)
> Jebakan: R2 gagal karena tidak ada fakta pendapatan. Backward chaining harus **kembali (backtrack)** dan mencoba rule lain (R3) untuk goal yang sama.

---

## 10. TUGAS 2 — Budi & Dewi

**Rules:**
- R1: IF (bukan hari minggu) AND (Budi sehat) OR (Dewi tidak kuliah) THEN (Budi kuliah)
- R2: IF (Budi kuliah) THEN (Dewi kuliah)
- R3: IF (bukan hari minggu) AND (Budi sakit) THEN (Dewi di rumah sakit)
- R4: IF (bukan hari minggu) THEN (mahasiswa SD kuliah)
- R5: IF (Budi di rumah sakit) AND (mahasiswa SD kuliah) THEN (Dewi kuliah)
- R6: IF (mahasiswa SD kuliah) AND (Budi sakit) THEN (Dewi tidak kuliah)
- R7: IF (Budi kuliah) AND (Dewi tidak kuliah) THEN (Dewi sakit)
- R8: IF (bukan hari minggu) AND (Dewi sakit) THEN (kuliah tidak libur)
- R9: IF (mahasiswa SD kuliah) THEN (kuliah tidak libur)
- R10: IF (kuliah tidak libur) THEN (Budi belajar di kampus SD)

**Database:** bukan hari minggu, Budi sakit
**Goal:** Dewi sakit **dan** kuliah tidak libur

**Singkatan:** M = bukan hari minggu, BSk = Budi sakit, BSh = Budi sehat, DK = Dewi kuliah, DTK = Dewi tidak kuliah, DRS = Dewi di RS, BRS = Budi di RS, SD = mahasiswa SD kuliah, BK = Budi kuliah, DS = Dewi sakit, KTL = kuliah tidak libur.

**Pembacaan R1:** `(M AND BSh) OR DTK → BK`. AND diproses lebih dulu daripada OR, jadi cukup salah satu: (M dan BSh) atau DTK.

### a) Forward Chaining

Fakta awal: **M, BSk**

| Iter | Pengecekan (dari atas) | Rule dieksekusi | Fakta baru |
|---|---|---|---|
| 1 | R1: (M ✓ ∧ BSh ✗) ∨ DTK ✗ → ✗. R2: BK ✗. R3: M ✓, BSk ✓ | **R3** | DRS |
| 2 | R1 ✗, R2 ✗, R3 sudah. R4: M ✓ | **R4** | SD |
| 3 | R1 ✗, R2 ✗. R5: BRS ✗ (yang ada *Dewi* di RS, bukan *Budi*) → ✗. R6: SD ✓, BSk ✓ | **R6** | DTK |
| 4 | R1: DTK ✓ → bagian OR terpenuhi | **R1** | BK |
| 5 | R2: BK ✓ | **R2** | DK |
| 6 | R5 ✗. R6 sudah. R7: BK ✓, DTK ✓ | **R7** | **DS** ✅ |
| 7 | R8: M ✓, DS ✓ | **R8** | **KTL** ✅ |

Database akhir: M, BSk, DRS, SD, DTK, BK, DK, DS, KTL.
**Kesimpulan:** Dewi sakit ✓ dan kuliah tidak libur ✓ → **kedua goal TERBUKTI.**

⚠️ Catatan: iterasi 5 menghasilkan DK (Dewi kuliah), padahal iterasi 3 menghasilkan DTK (Dewi tidak kuliah). Soal ini memang mengandung pertentangan. Forward chaining mekanis tidak memeriksa konsistensi, hanya menambah fakta. Untuk kuis, ikuti prosedur, lalu sebutkan hasil akhirnya.
Jebakan lain: R3 menghasilkan "**Dewi** di RS", sedangkan R5 mensyaratkan "**Budi** di RS" → R5 tidak pernah terpenuhi.

### b) Backward Chaining

Ada **dua goal**: kerjakan satu per satu.

**Goal 1: DS (Dewi sakit)**

| Langkah | Stack | Yang dilakukan | Fakta baru |
|---|---|---|---|
| 1 | DS | Rule THEN=DS → **R7**: butuh BK ✗ dan DTK ✗ → push BK, DTK | – |
| 2 | DS, BK | Rule THEN=BK → **R1**: `(M ∧ BSh) ∨ DTK`. M ✓ tapi BSh ✗ (tidak ada fakta dan tidak ada rule yang menghasilkan) → bagian pertama gagal. Coba bagian OR kedua: **DTK** ✗ → push DTK | – |
| 3 | DS, BK, DTK | Rule THEN=DTK → **R6**: butuh SD ✗ dan BSk ✓ → push SD | – |
| 4 | DS, BK, DTK, SD | Rule THEN=SD → **R4**: M ✓ → dieksekusi | **SD** |
| 5 | DS, BK, DTK | pop SD. R6: SD ✓, BSk ✓ → dieksekusi | **DTK** |
| 6 | DS, BK | pop DTK. R1: DTK ✓ → dieksekusi | **BK** |
| 7 | DS | pop BK. Kembali ke R7: BK ✓, DTK ✓ (sudah terbukti) → dieksekusi | **DS** ✅ |

**Goal 2: KTL (kuliah tidak libur)**

| Langkah | Stack | Yang dilakukan | Fakta baru |
|---|---|---|---|
| 1 | KTL | Rule THEN=KTL: **R8** (M ∧ DS) dicoba dulu. M ✓, DS ✓ (sudah terbukti di Goal 1) → dieksekusi | **KTL** ✅ |

(Alternatif: **R9** juga bisa, karena SD sudah terbukti.)

**Kesimpulan:** Dewi sakit ✓ dan kuliah tidak libur ✓ → **kedua goal TERBUKTI.**

---

## 11. Contoh Implementasi: Sistem Pakar Diabetes

### Data gejala
| Kode | Gejala |
|---|---|
| G1 | Sering merasa haus dan lapar |
| G2 | Penurunan berat badan tiba-tiba |
| G3 | Sering buang air kecil |
| G4 | Kelelahan berlebihan |
| G5 | Penglihatan kabur |
| G6 | Luka sulit sembuh |
| G7 | Mual atau muntah |
| G8 | Kenaikan berat badan tidak biasa |

### Tabel pengetahuan
| Tipe | G1 | G2 | G3 | G4 | G5 | G6 | G7 | G8 |
|---|---|---|---|---|---|---|---|---|
| Tipe 1 | T | T | T | T | T | F | F | F |
| Tipe 2 | T | F | T | T | T | T | F | F |
| Gestasional | T | F | T | T | F | F | T | T |

### Diubah menjadi rule (untuk forward/backward chaining)
*(Rule di bawah adalah penerjemahan tabel di atas oleh saya, bukan tertulis di slide.)*
- R1: IF G1 AND G2 AND G3 AND G4 AND G5 THEN **Diabetes Tipe 1**
- R2: IF G1 AND G3 AND G4 AND G5 AND G6 THEN **Diabetes Tipe 2**
- R3: IF G1 AND G3 AND G4 AND G7 AND G8 THEN **Diabetes Gestasional**

### Contoh soal
Pasien memiliki gejala **G1, G3, G4, G5, G6**. Diagnosis?

**Forward chaining:**
| Iter | Pengecekan | Rule | Fakta baru |
|---|---|---|---|
| 1 | R1: G2 ✗ → ✗. R2: G1 ✓, G3 ✓, G4 ✓, G5 ✓, G6 ✓ | **R2** | Diabetes Tipe 2 |
| 2 | R3: G7 ✗ → ✗. Tidak ada rule lain | – | selesai |

**Kesimpulan:** **Diabetes Tipe 2**.

**Backward chaining (goal: Gestasional?):** R3 butuh G1 ✓, G3 ✓, G4 ✓, G7 ✗ (tidak ada, tidak ada rule penghasilnya) → **gagal** → bukan Gestasional.

Poin: G1, G3, G4 adalah **gejala bersama** semua tipe. Pembeda: G2 → Tipe 1; G6 → Tipe 2; G7 + G8 → Gestasional.

---

# BAGIAN B — PERTEMUAN 7

## 12. Konsep Logika Fuzzy

**Definisi:** logika yang merepresentasikan nilai **samar**, **ketidakpastian**, **kebenaran sebagian** atau **"degree of truth"**.

**Mengapa dibutuhkan?** Banyak permasalahan nyata mengandung **ketidakpastian**:
- "Cepat" itu berapa km/jam? 100? 50? 10? (tergantung konteks)
- Jika TINGGI ≥ 170 cm, Ani (169 cm) pendek atau tinggi? Beda 1 cm langsung "pendek" terasa tidak masuk akal.
- Jika suhu < 60° nyalakan pemanas: batas ini sama untuk semua orang? Musim panas/dingin? Siang/malam? → nilai optimal menjadi **tidak pasti / samar / fuzzy**.

### Boolean vs Fuzzy
| | Logika Boolean (klasik / crisp) | Logika Fuzzy |
|---|---|---|
| Nilai | Hanya **0 atau 1** | **Antara 0 dan 1** |
| Notasi | **{0, 1}** | **[0, 1]** |
| Sifat | Tegas (*precise*) | Samar (*imprecise*) |
| Ungkapan | benar/salah, ya/tidak, hitam/putih | "agak", "sedikit", "lumayan", "sangat" |

### Fakta sejarah
Dikembangkan pertama kali oleh **Prof. Lotfi A. Zadeh** melalui tulisannya tahun **1965** tentang **Teori Himpunan Fuzzy**.

### Penerapan
Rice cooker, AC, mesin cuci, kereta (subway), dan perangkat elektronik lain.

---

## 13. Teori Dasar Logika Fuzzy

1. **Variabel fuzzy:** variabel yang dibahas dalam sistem fuzzy. Contoh: umur, kecepatan, suhu, tinggi badan, penghasilan.
2. **Himpunan fuzzy:** kelompok yang mewakili suatu keadaan tertentu dalam variabel fuzzy. Punya dua atribut:
   - **Linguistik:** nama kelompok dengan bahasa alami. Contoh: suhu = *panas, sedang, dingin*; tinggi badan = *pendek, sedang, tinggi*.
   - **Numerik:** ukuran nilai variabel berupa angka. Contoh: 10, 30, 50.
3. **Semesta pembicaraan:** seluruh nilai yang boleh dioperasikan pada variabel fuzzy. Contoh: umur = [1, ∞]; suhu = [0, 100].
4. **Domain himpunan fuzzy:** seluruh nilai yang diizinkan dalam semesta pembicaraan dan boleh dioperasikan dalam suatu himpunan fuzzy.

**Contoh gabungan:** Variabel **Suhu** → semesta [0, 100] → himpunan fuzzy **Dingin, Sedang, Panas** (linguistik) → tiap himpunan punya domain angka (numerik).

---

## 14. Fungsi Keanggotaan

- Grafik yang mewakili besar **derajat keanggotaan** tiap nilai input, di interval **0 sampai 1**.
- Derajat keanggotaan variabel x dilambangkan **μ(x)**.
- Jenis kurva: **linear (naik/turun)**, **segitiga**, **trapesium**, **S (sigmoid)**, dll.

### 14.1 Kurva Linear Naik
```
        0                 ; x ≤ a
μ(x) =  (x − a)/(b − a)   ; a ≤ x ≤ b
        1                 ; x ≥ b
```
(Linear **turun**: `μ(x) = (b − x)/(b − a)` untuk a ≤ x ≤ b; 1 untuk x ≤ a; 0 untuk x ≥ b.)

**Soal (slide):** a = 20, b = 80. Berapa μ(24)?
1. Cek daerah: 20 ≤ 24 ≤ 80 → pakai rumus tengah.
2. μ(24) = (24 − 20)/(80 − 20) = 4/60.
3. **μ(24) = 0,0667** (≈ 1/15).

Tambahan latihan: μ(50) = 30/60 = **0,5**; μ(15) = **0** (x ≤ a); μ(90) = **1** (x ≥ b).

### 14.2 Kurva Segitiga
```
        0                 ; x ≤ a  atau  x ≥ c
μ(x) =  (x − a)/(b − a)   ; a ≤ x ≤ b
        (c − x)/(c − b)   ; b ≤ x ≤ c
```

**Soal (slide):** a = 10, b = 25, c = 40. Berapa μ(24)?
1. 24 berada di antara a (10) dan b (25) → sisi **naik**.
2. μ(24) = (24 − 10)/(25 − 10) = 14/15.
3. **μ(24) = 0,9333**.

Tambahan: μ(30) → sisi turun: (40 − 30)/(40 − 25) = 10/15 = **0,6667**. μ(25) = **1** (puncak). μ(45) = **0**.

### 14.3 Kurva Trapesium
```
        0                 ; x ≤ a  atau  x ≥ d
μ(x) =  (x − a)/(b − a)   ; a ≤ x ≤ b
        1                 ; b ≤ x ≤ c
        (d − x)/(d − c)   ; c ≤ x ≤ d
```

**Soal (slide):** a = 10, b = 25, c = 35, d = 50. Berapa μ(24)?
1. 24 berada di antara a (10) dan b (25) → sisi **naik**.
2. μ(24) = (24 − 10)/(25 − 10) = 14/15.
3. **μ(24) = 0,9333**.

Tambahan: μ(30) → di antara b dan c → **1**. μ(40) → sisi turun: (50 − 40)/(50 − 35) = 10/15 = **0,6667**.

**Cara cepat memilih rumus:** tentukan dulu x berada di bagian grafik mana (kiri-nol / naik / datar / turun / kanan-nol), baru pakai rumus bagian itu.

---

## 15. Operasi Himpunan Fuzzy

- Dibutuhkan untuk proses **penalaran/inferensi**.
- Operasi dilakukan terhadap **derajat keanggotaan μ(x)**.
- Derajat keanggotaan hasil operasi dua himpunan fuzzy disebut **α-predikat**.

| Operasi | Operator | Nama | Rumus |
|---|---|---|---|
| Gabungan (Union) | **OR** | operasi **max** | μ(A∪B) = max(μA(x), μB(x)) |
| Irisan (Intersection) | **AND** | operasi **min** | μ(A∩B) = min(μA(x), μB(x)) |
| Komplemen | **NOT** | | μ(Aᶜ) = 1 − μA(x) |

**Contoh hitung:** μA(x) = 0,7 dan μB(x) = 0,4
1. OR: max(0,7; 0,4) = **0,7**
2. AND: min(0,7; 0,4) = **0,4** ← inilah α-predikat untuk "A dan B"
3. NOT A: 1 − 0,7 = **0,3**

---

## 16. Sistem Inferensi Fuzzy (FIS)

Cara **memetakan ruang input ke ruang output** memakai logika fuzzy.

```
INPUT (crisp) → [Fuzzifikasi] → [Inferensi] → [Defuzzifikasi] → OUTPUT (crisp)
                       ↑              ↑              ↑
                       └──────── Basis Pengetahuan ──┘
```

| Tahap | Fungsi |
|---|---|
| **Basis pengetahuan** | Kumpulan aturan **IF-THEN** buatan pakar (plus fungsi keanggotaan) |
| **Fuzzifikasi** | Mengubah input **tegas (crisp)** menjadi **variabel linguistik (fuzzy)** memakai fungsi keanggotaan |
| **Inferensi** | Mengubah input fuzzy menjadi output fuzzy dengan mengikuti aturan IF-THEN |
| **Defuzzifikasi** | Mengubah hasil inferensi menjadi output **tegas (crisp)** |

---

## 17. Studi Kasus Mesin Cuci (Tsukamoto, Mamdani, Sugeno)

### Soal
Mesin cuci otomatis berbasis fuzzy mengatur **kecepatan putar** berdasarkan **banyaknya pakaian** dan **tingkat kekotoran**.

- Kecepatan putar: min **500 rpm** (lambat), maks **1200 rpm** (cepat).
- Banyak pakaian: skala 0–100. ≤ 40 = **sedikit**, ≥ 80 = **banyak**.
- Tingkat kekotoran: skala 0–100. 0–40 = **rendah**, 50 = **sedang**, 60–100 = **tinggi**.

**Rules:**
| Rule | Pakaian | Kekotoran | Putaran |
|---|---|---|---|
| R1 | sedikit | rendah | lambat |
| R2 | sedikit | sedang | lambat |
| R3 | sedikit | tinggi | cepat |
| R4 | banyak | rendah | lambat |
| R5 | banyak | sedang | cepat |
| R6 | banyak | tinggi | cepat |

**Pertanyaan:** berapa rpm jika **pakaian = 50** dan **kekotoran = 58**?

---

### LANGKAH 1 — Fuzzifikasi (sama untuk ketiga metode)

**Fungsi keanggotaan** (bentuk yang konsisten dengan nilai α di slide):

*Pakaian (0–100):*
- Sedikit (linear turun): 1 jika x ≤ 40; (80 − x)/40 jika 40 ≤ x ≤ 80; 0 jika x ≥ 80
- Banyak (linear naik): 0 jika x ≤ 40; (x − 40)/40 jika 40 ≤ x ≤ 80; 1 jika x ≥ 80

*Kekotoran (0–100):*
- Rendah (linear turun): 1 jika x ≤ 40; (50 − x)/10 jika 40 ≤ x ≤ 50; 0 jika x ≥ 50
- Sedang (segitiga 40–50–60): (x − 40)/10 jika 40 ≤ x ≤ 50; (60 − x)/10 jika 50 ≤ x ≤ 60
- Tinggi (linear naik): 0 jika x ≤ 50; (x − 50)/10 jika 50 ≤ x ≤ 60; 1 jika x ≥ 60

**Hitung untuk Pakaian = 50:**
- μSedikit(50) = (80 − 50)/40 = 30/40 = **0,75**
- μBanyak(50) = (50 − 40)/40 = 10/40 = **0,25**

**Hitung untuk Kekotoran = 58:**
- μRendah(58) = **0** (58 ≥ 50)
- μSedang(58) = (60 − 58)/10 = **0,2**
- μTinggi(58) = (58 − 50)/10 = **0,8**

### LANGKAH 2 — α-predikat tiap rule (operator AND = **min**)

| Rule | Perhitungan | α |
|---|---|---|
| R1 (sedikit ∧ rendah) | min(0,75; 0) | **0** |
| R2 (sedikit ∧ sedang) | min(0,75; 0,2) | **0,2** |
| R3 (sedikit ∧ tinggi) | min(0,75; 0,8) | **0,75** |
| R4 (banyak ∧ rendah) | min(0,25; 0) | **0** |
| R5 (banyak ∧ sedang) | min(0,25; 0,2) | **0,2** |
| R6 (banyak ∧ tinggi) | min(0,25; 0,8) | **0,25** |

Hasil ini sama dengan grafik Mamdani di slide (R2 = 0,20 lambat; R3 = 0,75 cepat; R5 = 0,20 cepat; R6 = 0,25 cepat; R1 & R4 tidak aktif).

**Fungsi keanggotaan output (rpm):**
- Lambat (turun): μ(z) = 1 jika z ≤ 500; (1200 − z)/700 jika 500 ≤ z ≤ 1200
- Cepat (naik): μ(z) = (z − 500)/700 jika 500 ≤ z ≤ 1200; 1 jika z ≥ 1200

---

### A) METODE MAMDANI (potong output, defuzzifikasi **centroid**)

**Langkah 3 — Komposisi (agregasi) dengan MAX.** Tiap rule "memotong" himpunan outputnya pada tinggi α:
- R2: Lambat dipotong di 0,2
- R3: Cepat dipotong di 0,75
- R5: Cepat dipotong di 0,2
- R6: Cepat dipotong di 0,25

Ambil **max** semuanya → R5 dan R6 "tertutup" oleh R3 (karena Cepat dipotong 0,75 paling tinggi).

**Langkah 4 — Cari titik potong t₁ dan t₂.**
- t₁: garis Cepat memotong tinggi 0,2 → (t₁ − 500)/700 = 0,2 → t₁ = 0,2·700 + 500 = **640**
- t₂: garis Cepat mencapai tinggi 0,75 → (t₂ − 500)/700 = 0,75 → t₂ = 0,75·700 + 500 = **1025**

**Langkah 5 — Fungsi hasil agregasi:**
```
         0,2                 ; z ≤ 640
μ(z) =   (z − 500)/700       ; 640 ≤ z ≤ 1025
         0,75                ; z ≥ 1025
```
Daerahnya: D₁ (0–640), D₂ (640–1025), D₃ (1025–1200).

**Langkah 6 — Defuzzifikasi centroid:**

z* = ∫ μ(z)·z dz / ∫ μ(z) dz = (M₁+M₂+M₃)/(A₁+A₂+A₃)

*Momen (M):*
- M₁ = ∫₀⁶⁴⁰ 0,2z dz = 0,1·z² |₀⁶⁴⁰ = 0,1 · 409.600 = **40.960**
- M₂ = ∫₆₄⁰¹⁰²⁵ z(z − 500)/700 dz = **151.273,013** (nilai di slide)
- M₃ = ∫₁₀₂₅¹²⁰⁰ 0,75z dz = 0,375·(1200² − 1025²) = 0,375 · 389.375 = **146.015,625**

*Luas (A):*
- A₁ = 0,2 · 640 = **128**
- A₂ = ∫₆₄⁰¹⁰²⁵ (z − 500)/700 dz = **173,712** (nilai di slide)
- A₃ = 0,75 · (1200 − 1025) = 0,75 · 175 = **131,25**

**Langkah 7 — Hasil:**
z* = (40.960 + 151.273,013 + 146.015,625) / (128 + 173,712 + 131,25)
= 338.248,638 / 432,962 = **781,243 rpm** (jawaban di slide)

⚠️ **Catatan ketelitian:** slide memakai koefisien yang dibulatkan (0,0014z − 0,7143). Jika dihitung eksak dengan 1/700: A₂ = (525² − 140²)/1400 = 182,875 dan M₂ ≈ 159.037,08, sehingga z* ≈ 346.012,7 / 442,125 ≈ **782,6 rpm**. Selisihnya hanya karena pembulatan. Untuk kuis, gunakan angka sesuai yang diajarkan (781,243) dan pahami caranya.

---

### B) METODE TSUKAMOTO

Ciri: output tiap rule berupa **nilai tegas (z)** yang dicari dari **α-predikat** memakai kurva output yang **monoton** (naik/turun). Defuzzifikasi = **rata-rata terbobot**.

**Langkah 3 — Cari z tiap rule dari α:**
- Untuk output **Lambat** (turun): α = (1200 − z)/700 → **z = 1200 − 700α**
- Untuk output **Cepat** (naik): α = (z − 500)/700 → **z = 500 + 700α**

| Rule | α | Output | z |
|---|---|---|---|
| R1 | 0 | Lambat | 1200 − 0 = 1200 *(α = 0, tidak berpengaruh)* |
| R2 | 0,2 | Lambat | 1200 − 700·0,2 = **1060** |
| R3 | 0,75 | Cepat | 500 + 700·0,75 = **1025** |
| R4 | 0 | Lambat | 1200 *(α = 0)* |
| R5 | 0,2 | Cepat | 500 + 700·0,2 = **640** |
| R6 | 0,25 | Cepat | 500 + 700·0,25 = **675** |

**Langkah 4 — Rata-rata terbobot:**

Z* = Σ(αᵢ · zᵢ) / Σ(αᵢ)

- Pembilang = (0,2·1060) + (0,75·1025) + (0,2·640) + (0,25·675)
  = 212 + 768,75 + 128 + 168,75 = **1277,5**
- Penyebut = 0,2 + 0,75 + 0,2 + 0,25 = **1,4**
- **Z\* = 1277,5 / 1,4 = 912,5 rpm**

⚠️ **Slide menuliskan 1061,25.** Perhitungan saya dari α-predikat yang sama dengan Mamdani menghasilkan **912,5**. Saya tidak bisa merekonstruksi 1061,25 dari data di slide. Kemungkinan itu salah ketik atau memakai data lain. **Pahami caranya** (α → z → rata-rata terbobot). Jika dosen menanyakan angka, konfirmasikan ke beliau.

---

### C) METODE SUGENO (orde-1)

Ciri: output tiap rule berupa **persamaan/konstanta**, bukan himpunan fuzzy. Defuzzifikasi = **rata-rata terbobot** (rumus sama dengan Tsukamoto, hanya zᵢ berbeda cara memperolehnya).

**Rules Sugeno (slide):**
- R1: putaran = 500
- R2: putaran = 10·kekotoran + 100
- R3: putaran = 10·kekotoran + 100
- R4: putaran = 5·(…) + 2·kekotoran
- R5: putaran = 5·(…) + 4·kekotoran
- R6: putaran = 5·(…) + 2·kekotoran + 300

*(Di slide suku pertama R4–R6 tertulis "kekotoran" dua kali; kemungkinan besar maksudnya suku pertama = **pakaian**. Saya pakai tafsiran itu.)*

**Langkah 3 — Hitung z tiap rule** (pakaian = 50, kekotoran = 58):
| Rule | α | Perhitungan z | z |
|---|---|---|---|
| R1 | 0 | 500 | 500 *(α = 0)* |
| R2 | 0,2 | 10·58 + 100 | **680** |
| R3 | 0,75 | 10·58 + 100 | **680** |
| R4 | 0 | 5·50 + 2·58 = 250 + 116 | 366 *(α = 0)* |
| R5 | 0,2 | 5·50 + 4·58 = 250 + 232 | **482** |
| R6 | 0,25 | 5·50 + 2·58 + 300 = 250 + 116 + 300 | **666** |

**Langkah 4 — Rata-rata terbobot:**
- Pembilang = (0,2·680) + (0,75·680) + (0,2·482) + (0,25·666)
  = 136 + 510 + 96,4 + 166,5 = **908,9**
- Penyebut = **1,4**
- **Z\* = 908,9 / 1,4 ≈ 649,21 rpm**

⚠️ **Slide menuliskan 733,85.** Saya tidak bisa merekonstruksinya dari rule yang tertulis (jika "kekotoran" dibaca literal untuk semua suku, hasilnya ≈ 662,07). Soal ini ambigu dan angka slide tampaknya tidak konsisten. Yang diuji kemungkinan besar **prosedurnya**.

---

## 18. Perbandingan Tiga Metode

| Aspek | Mamdani | Tsukamoto | Sugeno |
|---|---|---|---|
| Output rule | Himpunan fuzzy (dipotong pada α) | Himpunan fuzzy **monoton** → dicari nilai z tegas | **Konstanta / persamaan linear** |
| Komposisi | **MAX** (agregasi daerah) | Tidak ada agregasi daerah | Tidak ada agregasi daerah |
| Defuzzifikasi | **Centroid** (integral) | **Rata-rata terbobot** | **Rata-rata terbobot** |
| Rumus | z* = ∫μ(z)z dz / ∫μ(z) dz | Z* = Σαᵢzᵢ / Σαᵢ | Z* = Σαᵢzᵢ / Σαᵢ |
| Beban hitung | Paling berat (integral) | Ringan | Ringan |
| Operator AND | min | min | min |

**Urutan kerja umum:** Fuzzifikasi → hitung α-predikat tiap rule (AND = min, OR = max) → (Mamdani: potong + max agregasi / Tsukamoto: cari z tiap rule / Sugeno: hitung persamaan) → defuzzifikasi.

---

## 19. Latihan Soal Tambahan + Kunci

**Soal 1 (FOL).** Terjemahkan: "Tidak ada mahasiswa yang malas."
- Jawab: `¬∃x Mahasiswa(x) ∧ Malas(x)`, ekuivalen `∀x Mahasiswa(x) ⇒ ¬Malas(x)`.

**Soal 2 (Unification).** Unify `Loves(x, Ani)` dengan `Loves(Budi, y)`.
- Arg 1: x vs Budi → `x/Budi`. Arg 2: Ani vs y → `y/Ani`. **Jawab:** `{x/Budi, y/Ani}`.

**Soal 3 (Unification).** Apakah `Knows(x, x)` bisa di-unify dengan `Knows(John, Mary)`?
- Arg 1: `x/John`. Arg 2: x (= John) vs Mary → John ≠ Mary → **gagal**.

**Soal 4 (Forward).** Rules: R1: IF P AND Q THEN R; R2: IF R THEN S; R3: IF S OR T THEN U. Fakta: P, Q. Goal: U?
- Iter 1: R1: P ✓, Q ✓ → **R** . Iter 2: R2: R ✓ → **S**. Iter 3: R3: S ✓ → **U**. **Jawab:** U benar.

**Soal 5 (Backward).** Rules yang sama, fakta P, Q, goal U.
- Stack: U → R3 butuh (S ∨ T) → coba S → R2 butuh R → R1 butuh P ✓ Q ✓ → R ✓ → S ✓ → U ✓. (T tidak perlu dicek karena OR sudah terpenuhi.)

**Soal 6 (Fuzzy).** Segitiga a = 0, b = 50, c = 100. Hitung μ(20) dan μ(80).
- μ(20) = (20 − 0)/(50 − 0) = **0,4**. μ(80) = (100 − 80)/(100 − 50) = 20/50 = **0,4**.

**Soal 7 (Operasi).** μA = 0,3 dan μB = 0,9. Hitung A OR B, A AND B, NOT B.
- OR = max = **0,9**; AND = min = **0,3**; NOT B = 1 − 0,9 = **0,1**.

**Soal 8 (Tsukamoto mini).** Dua rule aktif: α₁ = 0,5 dengan z₁ = 800, dan α₂ = 0,25 dengan z₂ = 600. Hitung Z*.
- Pembilang = 0,5·800 + 0,25·600 = 400 + 150 = 550. Penyebut = 0,75. **Z\* = 733,33**.

**Soal 9 (Pilihan ganda konsep).**
a. Pendekatan inferensi yang "mencoba semua interpretasi" = **model checking** (tidak efisien).
b. Forward chaining disebut **data-driven**, backward chaining **goal-driven**.
c. Prolog memakai **backward chaining**.
d. Rule-based expert system umumnya memakai **forward chaining**.
e. Hasil operasi dua himpunan fuzzy disebut **α-predikat**.
f. Derajat keanggotaan dilambangkan **μ**.
g. Proses mengubah crisp → fuzzy = **fuzzifikasi**; fuzzy → crisp = **defuzzifikasi**.
h. Union = **max (OR)**; Intersection = **min (AND)**; Complement = **1 − μ (NOT)**.

---

## 20. Cheat Sheet Kuis

**FOL**
- `∀` ↔ `⇒` ; `∃` ↔ `∧`.
- Fungsi → objek; predikat → benar/salah.
- Unification gagal jika: konstanta berbeda, nama predikat/fungsi berbeda, jumlah argumen berbeda, atau variabel muncul di dalam pasangannya (occurs check).

**Forward chaining**
1. Mulai dari fakta. 2. Cek rule dari atas. 3. Rule yang premisnya terpenuhi → eksekusi, tambah fakta baru. 4. Tiap rule maksimal sekali. 5. Ulangi dari atas sampai goal tercapai atau tidak ada rule baru.

**Backward chaining**
1. Mulai dari goal. 2. Cari rule yang THEN = goal. 3. Premis tidak ada di fakta → push ke stack sebagai subgoal. 4. Buktikan subgoal paling atas; setelah terbukti, tambah ke fakta dan pop. 5. Jika satu rule gagal → backtrack ke rule lain untuk goal yang sama.

**Fuzzy**
- Boolean {0,1}; Fuzzy [0,1].
- Linear naik: (x−a)/(b−a). Segitiga: naik (x−a)/(b−a), turun (c−x)/(c−b). Trapesium: ditambah bagian datar = 1.
- AND = min, OR = max, NOT = 1 − μ.
- FIS: Fuzzifikasi → Inferensi → Defuzzifikasi (dengan Basis Pengetahuan).
- Mamdani: potong + max + **centroid**. Tsukamoto & Sugeno: **rata-rata terbobot** Σαz/Σα.
- Tsukamoto: z dari α (Lambat: z = 1200 − 700α; Cepat: z = 500 + 700α pada contoh mesin cuci). Sugeno: z dari persamaan rule.

**Jawaban akhir soal-soal slide**
| Soal | Jawaban |
|---|---|
| FC contoh (Z) | Z benar (R3 → R2 → R1) |
| BC contoh (Z) | Z benar (X → Y → Z) |
| FC latihan (K) | K benar (B, D, P, H, K) |
| BC latihan (H) | H benar (G, C, E, I, F, H) |
| Tugas 1 (IBM) | Ya, tepat investasi di IBM |
| Tugas 2 | Dewi sakit ✓ dan kuliah tidak libur ✓ |
| Mesin cuci Mamdani | ≈ 781,243 rpm (slide) / ≈ 782,6 (eksak) |
| Mesin cuci Tsukamoto | 912,5 rpm (hitung ulang) / slide 1061,25 ⚠️ |
| Mesin cuci Sugeno | ≈ 649,21 rpm (hitung ulang) / slide 733,85 ⚠️ |

**Semoga sukses kuisnya! 🍀**
