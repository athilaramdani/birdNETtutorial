# BirdNET‑Analyzer — Panduan Singkat (Bahasa Indonesia)

> **Tujuan**: Membantu dosen/penilai menjalankan *inference* **dan** *training* cepat menggunakan BirdNET‑Analyzer di Windows (CMD/PowerShell).

---

## 1. Struktur Folder dan Dataset
1. Buat satu folder kerja, mis. `F:\projek_dosen`.
2. **Unduh** dataset contoh di Google Drive:
   [https://drive.google.com/drive/u/0/folders/1ITfVDgiuiDOgSIhek\_XktSx0CvGN1mXr](https://drive.google.com/drive/u/0/folders/1ITfVDgiuiDOgSIhek_XktSx0CvGN1mXr)
3. **Ekstrak** arsip ke dalam folder kerja sehingga struktur akhirnya seperti ini:

```
F:\projek_dosen\09mei2025\
├── train_grouped\   # folder audio untuk pelatihan model
├── test_grouped\    # folder audio untuk pengujian model
└── output\          # folder kosong, hasil training dan analisis akan disimpan di sini
```

### Isi `train_grouped` dan `test_grouped`

Setiap folder spesies memiliki file audio `.mp3`:

```
F:\projek_dosen\09mei2025\train_grouped\
├── cinnyris jugularis\
│   ├── 31411.mp3
│   └── 33037.mp3
├── dicrurus paradiseus\
│   ├── 19055.mp3
│   └── 29957.mp3
└── ...

F:\projek_dosen\09mei2025\test_grouped\
├── cinnyris jugularis\
│   └── ...
├── dicrurus paradiseus\
│   └── ...
└── ...
```

### Isi Folder `output`

Folder ini akan berisi hasil training dan analisis:

```
F:\projek_dosen\09mei2025\output\
├── analyze1\        # hasil analisis pertama
├── analyze2\        # hasil analisis kedua
├── train1\          # hasil training model pertama
└── train2\          # hasil training model berikutnya
```

---

## 2. Kloning BirdNET‑Analyzer

```powershell
cd F:\projek_dosen
git clone https://github.com/birdnet-team/BirdNET-Analyzer.git
cd BirdNET-Analyzer
```

set library:
> *catatan* : numpy harus 1.24.4, tensorflow harus 2.12.0 biar tidak error

```powershell
py -m pip install numpy==1.24.4 scipy pandas soundfile librosa matplotlib tqdm resampy lazy_loader tensorflow==2.12.0
```

---

## 3. Analisis Cepat (*Inference*)

Analisis semua file di `test_grouped` dan simpan hasil ke `output\analyze1`:

```powershell
py -m birdnet_analyzer.analyze "F:\projek_dosen\09mei2025\test_grouped" -o "F:\projek_dosen\09mei2025\output\analyze1"
```

### Opsi Penting `analyze`

| Opsi                | Fungsi                              | Default      |
| ------------------- | ----------------------------------- | ------------ |
| `-o / --output`     | Folder output                       | input path   |
| `--min_conf`        | Ambang kepercayaan deteksi          | **0.25**     |
| `--fmin` / `--fmax` | Batas frekuensi band‑pass (Hz)      | 0 / 15000    |
| `--threads`         | Jumlah CPU thread                   | **2**        |
| `--classifier`      | Model kustom (tflite)               | model bawaan |
| `--rtype`           | Format hasil (`table`, `csv`, dll.) | `table`      |
| `--combine_results` | Gabungkan semua hasil ke satu file  | `False`      |

Contoh lengkap:

```powershell
py -m birdnet_analyzer.analyze "F:\projek_dosen\09mei2025\test_grouped" -o "F:\projek_dosen\09mei2025\output\analyze2" --min_conf 0.30 --threads 4 --rtype csv table --classifier checkpoints\custom\Custom_Classifier.tflite
```

---

## 4. Training Model Kustom

1. Siapkan folder data misalkan di folder tadi yaitu `test_grouped`:

```
test_grouped\
├── Poecile_atricapillus_Black‑capped_Chickadee\
├── Noise\
└── ...
```

2. Jalankan training:

```powershell
py -m birdnet_analyzer.train "F:\projek_dosen\09mei2025\train_grouped" -o "F:\projek_dosen\09mei2025\output\train1"
```

contoh output akan seperti ini
```powershell
...
 - loading 'pnoepyga pusilla': 100%|████████████████████████████████| 127/127 [00:07<00:00, 16.52f/s] 
 - loading 'psilopogon duvaucelii':   0%|                                     | 0/120 [00:00<?, ?f/s]
 ...
 Note: Hit end of (available) data during resync.
 - loading 'tanysiptera galatea': 100%|█████████████████████████████| 106/106 [00:23<00:00,  4.48f/s]
...Done. Loaded 2179 training samples and 12 labels.
Normalizing embeddings...
Building model...
...Done.
Training model...
Training on 1738 samples, validating on 441 samples.
Epoch 1/50
55/55 [==============================] - 1s 8ms/step - loss: 9.1147 - AUPRC: 0.0644 - AUROC: 0.5384 - val_loss: 8.2518 - val_AUPRC: 0.0624 - val_AUROC: 0.5356 - lr: 2.0000e-05
Epoch 2/50
55/55 [==============================] - 0s 3ms/step - loss: 8.8906 - AUPRC: 0.0776 - AUROC: 0.5858 - val_loss: 8.1158 - val_AUPRC: 0.0758 - val_AUROC: 0.5835 - lr: 4.0000e-05
Epoch 3/50
55/55 [==============================] - 0s 2ms/step - loss: 8.5710 - AUPRC: 0.1075 - AUROC: 0.6456 - val_loss: 7.9121 - val_AUPRC: 0.1063 - val_AUROC: 0.6484 - lr: 6.0000e-05
Epoch 4/50
55/55 [==============================]
...
Epoch 50/50
55/55 [==============================] - 0s 2ms/step - loss: 0.7779 - AUPRC: 0.9293 - AUROC: 0.9904 - val_loss: 1.0185 - val_AUPRC: 0.8253 - val_AUROC: 0.9644 - lr: 1.0110e-05
...Done.
Saving model...
...
ing (showing 5 of 78). These functions will not be directly callable after loading.

No separate test data provided for evaluation. Using validation metrics.
...Done. Best AUPRC: 0.8253326416015625, Best AUROC: 0.9644060134887695, Best Loss: 1.0184696912765503 (epoch 50/50)
```

nanti di folder `train 1`

### Opsi Penting `train`

| Opsi              | Fungsi             | Default                                |
| ----------------- | ------------------ | -------------------------------------- |
| `-o / --output`   | Lokasi model hasil | `checkpoints/custom/Custom_Classifier` |
| `--epochs`        | Jumlah epoch       | **50**                                 |
| `--batch_size`    | Ukuran batch       | **32**                                 |
| `--val_split`     | Rasio validasi     | **0.2**                                |
| `--learning_rate` | Learning rate awal | **0.0001**                             |
| `--threads`       | Jumlah CPU thread  | **2**                                  |
| `--focal-loss`    | Gunakan focal loss | `False`                                |

---

##

## 6. Tips & Catatan

* Gunakan `--lat`, `--lon`, dan `--week` untuk mengatur konteks geografis
* Folder `Noise` / `Background` bisa digunakan untuk contoh negatif
* Nama folder bisa mengandung multi-label dipisahkan koma
* Training akan lebih cepat jika GPU tersedia
* Model hasil training bersifat non-komersial: CC BY-NC-SA 4.0

Selamat mencoba! 🚀
