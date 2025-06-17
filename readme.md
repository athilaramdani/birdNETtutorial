# BirdNET‑Analyzer — Panduan Singkat (Bahasa Indonesia)

> **Tujuan**: Membantu dosen/penilai menjalankan *inference* **dan** *training* cepat menggunakan BirdNET‑Analyzer di Windows (CMD/PowerShell).

---

## 1  Persiapan Folder & Dataset

1. Buat satu folder kerja, mis. `F:\projek_dosen`.
2. **Unduh** dataset contoh di Google Drive:
   [https://drive.google.com/drive/u/0/folders/1ITfVDgiuiDOgSIhek\_XktSx0CvGN1mXr](https://drive.google.com/drive/u/0/folders/1ITfVDgiuiDOgSIhek_XktSx0CvGN1mXr)
3. **Ekstrak** arsip ke dalam folder kerja sehingga struktur akhirnya seperti ini:

```
F:\projek_dosen\09mei2025\
├── train_grouped\   # audio untuk training
├── test_grouped\    # audio untuk pengujian
└── output\          # folder kosong (hasil)
```

## 2  Kloning BirdNET‑Analyzer

```powershell
cd F:\projek_dosen
git clone https://github.com/birdnet-team/BirdNET-Analyzer.git
cd BirdNET-Analyzer
```

jika ada error library tidak ada, taruh ini diterminal
```powershell
 py -m pip install numpy scipy tensorflow pandas soundfile librosa matplotlib tqdm
```

> **Catatan**: kalau masih tidak bisa, pakai script ini (`pip install numpy scipy tensorflow pandas soundfile librosa matplotlib tqdm`) diperlukan sekali saja.
---

## 3  Analisis Cepat (*Inference*)

Analisis semua file di `test_grouped` dan simpan hasil ke `output\o1`:

```powershell
py -m birdnet_analyzer.analyze "F:\projek_dosen\09mei2025\test_grouped" -o "F:\projek_dosen\09mei2025\output\o1"
```

### Opsi Penting `analyze` (ringkas)

| Opsi                | Fungsi                               | Default      |
| ------------------- | ------------------------------------ | ------------ |
| `-o / --output`     | Folder output                        | input path   |
| `--min_conf`        | Ambang kepercayaan deteksi           | **0.25**     |
| `--fmin` / `--fmax` | Batas frekuensi band‑pass (Hz)       | 0 / 15000    |
| `--threads`         | Jumlah CPU thread                    | **2**        |
| `--classifier`      | Model kustom (tflite)                | model bawaan |
| `--rtype`           | Format hasil (`table`, `csv`, dll.)  | `table`      |
| `--combine_results` | Gabungkan semua hasil ke satu berkas | `False`      |

#### Contoh Lengkap

```powershell
py -m birdnet_analyzer.analyze "F:\projek_dosen\09mai2025\test_grouped" -o "F:\projek_dosen\09mai2025\output\o_full" --min_conf 0.30 --threads 4 --rtype csv table --classifier checkpoints\custom\Custom_Classifier.tflite
```

---

## 4  Training Klasifier Kustom

1. Siapkan folder data berlabel per spesies:

```
train_data\
├── Poecile_atricapillus_Black‑capped_Chickadee\
├── Noise\                 # non‑event
└── …...
```

2. Jalankan:

```powershell
py -m birdnet_analyzer.train train_data/ -o checkpoints/custom/Custom_Classifier.tflite
```

### Opsi Penting `train` (ringkas)

| Opsi              | Fungsi                               | Default                                |
| ----------------- | ------------------------------------ | -------------------------------------- |
| `-o / --output`   | Lokasi model hasil                   | `checkpoints/custom/Custom_Classifier` |
| `--epochs`        | Jumlah epoch                         | **50**                                 |
| `--batch_size`    | Ukuran batch                         | **32**                                 |
| `--val_split`     | Rasio validasi (jika tanpa test set) | **0.2**                                |
| `--learning_rate` | LR awal                              | **0.0001**                             |
| `--threads`       | CPU thread                           | **2**                                  |
| `--focal-loss`    | Aktifkan focal‑loss                  | `False`                                |

#### Contoh Lengkap

```powershell
py -m birdnet_analyzer.train train_data/ -o checkpoints/custom/IndoMY_Birds.tflite --epochs 75 --batch_size 16 --focal-loss --learning_rate 5e-4 --threads 4 --val_split 0.15
```

Setelah training selesai, gunakan model kustom saat analisis:

```powershell
py -m birdnet_analyzer.analyze input_folder/ --classifier checkpoints/custom/IndoMY_Birds.tflite
```

---

## 5  Tips & Catatan

* **Lokasi/Lat/Lon**: Jika memakai model bawaan, Anda bisa menyetel `--lat`, `--lon`, `--week` agar daftar spesies difilter per lokasi.
* **Negative Samples**: Tambahkan folder berawalan `Noise`, `Background`, atau awalan `-` untuk contoh negatif.
* **Multi‑label**: Pisahkan label dengan koma di nama folder.
* **GPU**: Untuk training lebih cepat aktifkan GPU via TensorFlow jika tersedia.
* **License**: Model kustom yang dilatih dengan BirdNET‑Analyzer berada di bawah CC BY‑NC‑SA 4.0.

Selamat mencoba! 🚀
