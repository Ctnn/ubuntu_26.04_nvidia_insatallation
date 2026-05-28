# Ubuntu 26.04 LTS (Resolute Raccoon) — Kurulum Rehberi

## Sistem Bilgisi

| Bileşen | Detay |
|---|---|
| OS | Ubuntu 26.04 LTS (Resolute Raccoon) |
| GPU | NVIDIA GeForce RTX 5070 Ti Mobile |
| iGPU | AMD Radeon Graphics (Granite Ridge) |
| WiFi | MediaTek MT7922 |
| Ethernet | Realtek RTL8125 2.5GbE |

---

## 1. Node.js ve Claude Code

### nvm Kurulumu

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
```

### Node.js (LTS) Kurulumu

```bash
nvm install --lts
node --version   # v24.16.0
npm --version    # 11.13.0
```

### Claude Code Kurulumu

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

> nvm, gerekli PATH satırlarını otomatik olarak `~/.bashrc`'ye ekler.

---

## 2. NVIDIA Driver

Driver zaten Ubuntu 26.04 ile birlikte geldi. Kontrol:

```bash
nvidia-smi
# Driver Version: 595.71.05 | CUDA Version: 13.2
```

Manuel kurulum gerekirse:

```bash
sudo ubuntu-drivers install
# veya belirli bir versiyon:
sudo apt install nvidia-driver-595
```

---

## 3. CUDA Toolkit

### Repo Ekleme (Ubuntu 26.04)

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2604/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
```

> **Not:** NVIDIA'nın ubuntu2604 reposu henüz yoksa ubuntu2404 paketi kullanılabilir:
> ```bash
> wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
> ```

### CUDA Kurulumu

```bash
sudo apt install cuda-toolkit-13-2
# veya en güncel için:
sudo apt install cuda-toolkit
```

### PATH Ayarı (`~/.bashrc`'ye ekle)

```bash
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

```bash
source ~/.bashrc
nvcc --version
```

---

## 4. cuDNN

### NVIDIA Repo'dan Kurulum (CUDA repo eklendikten sonra)

```bash
sudo apt install cudnn
# veya spesifik versiyon:
sudo apt install cudnn9-cuda-13
```

### Doğrulama

```bash
dpkg -l | grep cudnn
```

---

## 5. TensorRT

### Repo'dan Kurulum

```bash
sudo apt install tensorrt
```

### Python Binding'leri (isteğe bağlı)

```bash
sudo apt install python3-libnvinfer python3-libnvinfer-dev
```

### Doğrulama

```bash
dpkg -l | grep -i tensorrt
python3 -c "import tensorrt; print(tensorrt.__version__)"
```

---

## 6. Sürücü Durumu Özeti

Tüm sürücüler kutu açık çalışıyor:

| Bileşen | Driver | Durum |
|---|---|---|
| RTX 5070 Ti Mobile | nvidia 595.71.05 | ✅ |
| AMD Radeon iGPU | amdgpu (kernel) | ✅ |
| WiFi MT7922 | mt7921e (kernel) | ✅ |
| Ethernet RTL8125 | r8169 (kernel) | ✅ |
| Dahili Ses | snd_hda_intel / SN6140 | ✅ |
| Razer Barracuda X 2.4 | USB Audio | ✅ |

---

## 7. Faydalı Komutlar

```bash
# GPU durumu
nvidia-smi

# CUDA versiyonu
nvcc --version

# Kurulu paket kontrolü
dpkg -l | grep -E "cuda|cudnn|tensorrt"

# Önerilen driver listesi
ubuntu-drivers list

# Kernel modülleri
lsmod | grep nvidia
```
