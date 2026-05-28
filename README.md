# Ubuntu 26.04 LTS (Resolute Raccoon) — NVIDIA AI Stack Kurulum Rehberi

Kapsam: NVIDIA driver, CUDA, cuDNN ve TensorRT kurulumu. RTX 5070 Ti Mobile üzerinde test edilmiştir.

## Sistem Bilgisi

| Bileşen | Detay |
|---|---|
| OS | Ubuntu 26.04 LTS (Resolute Raccoon) |
| Kernel | Linux 7.x |
| GPU | NVIDIA GeForce RTX 5070 Ti Mobile (GB205M) |
| iGPU | AMD Radeon Graphics (Granite Ridge) |
| CPU | AMD Ryzen (Raphael/Granite Ridge) |
| WiFi | MediaTek MT7922 802.11ax |
| Ethernet | Realtek RTL8125 2.5GbE |

---

## 1. Temel Araçlar

### git

```bash
sudo apt install -y git
git config --global user.name "Kullanici Adi"
git config --global user.email "email@example.com"
```

### GitHub CLI ve SSH

```bash
sudo apt install -y gh
```

SSH key oluştur ve GitHub'a ekle:

```bash
ssh-keygen -t ed25519 -C "email@example.com" -f ~/.ssh/id_ed25519 -N ""
ssh-keyscan github.com >> ~/.ssh/known_hosts
cat ~/.ssh/id_ed25519.pub   # Bu çıktıyı GitHub > Settings > SSH keys'e ekle
```

Bağlantıyı test et:

```bash
ssh -T git@github.com
# Hi <kullanici>! You've successfully authenticated...
```

### Node.js (nvm ile)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install --lts
node --version   # v24.16.0
```

---

## 2. NVIDIA Driver

Ubuntu 26.04, modern NVIDIA GPU'ları için driver'ı kutudan çıkar çıkmaz yükler.

Durumu kontrol et:

```bash
nvidia-smi
# Driver Version: 595.71.05 | CUDA Version: 13.2 (maks. desteklenen)
```

Eğer driver eksikse:

```bash
ubuntu-drivers list              # Önerilen driver'ları listele
sudo ubuntu-drivers install      # Önerilen driver'ı otomatik kur
# veya belirli bir versiyon:
sudo apt install nvidia-driver-595
sudo reboot
```

---

## 3. CUDA Toolkit

### NVIDIA Repo Ekleme

Ubuntu 26.04 için resmi NVIDIA CUDA reposu mevcut:

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2604/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
```

### Driver-CUDA Versiyon Uyumu

`nvidia-smi` çıktısındaki `CUDA Version` değeri, driver'ın desteklediği **maksimum** CUDA versiyonudur. Daha yüksek bir CUDA toolkit kurulmamalı.

| Driver Serisi | Maks. CUDA |
|---|---|
| 595.x | 13.2 |
| 570.x | 12.8 |
| 550.x | 12.4 |

### Kurulum

```bash
# Uyumlu versiyonu kur (driver 595 → maks. CUDA 13.2 → cuda-toolkit-13-1 güvenli)
sudo apt install -y cuda-toolkit-13-1
```

### PATH Ayarı

`~/.bashrc` dosyasına ekle:

```bash
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

```bash
source ~/.bashrc
nvcc --version
# Cuda compilation tools, release 13.1
```

---

## 4. cuDNN

CUDA repo eklendikten sonra apt ile kurulur:

```bash
sudo apt install -y nvidia-cudnn
```

Kurulumu doğrula:

```bash
dpkg -l | grep cudnn
# nvidia-cudnn  9.0.0.312~cuda12build1
```

---

## 5. TensorRT

### Önemli Not: Python Versiyon Kısıtlaması

TensorRT'nin PyPI paketi yalnızca **Python 3.8–3.12** destekler. Ubuntu 26.04 varsayılan Python'u 3.14 olduğundan Python 3.12 ayrıca kurulmalıdır.

### Python 3.12 Kurulumu (deadsnakes PPA)

```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install -y python3.12 python3.12-venv python3.12-dev
python3.12 --version   # Python 3.12.x
```

### TensorRT Sanal Ortamı

```bash
python3.12 -m venv ~/venvs/tensorrt
source ~/venvs/tensorrt/bin/activate
pip install tensorrt==10.16.1.11
```

### LD_LIBRARY_PATH Ayarı

TensorRT'nin kütüphaneleri venv içinde bulunur, sisteme tanıtılması gerekir. `~/.bashrc` dosyasına ekle:

```bash
export LD_LIBRARY_PATH=/home/$USER/venvs/tensorrt/lib/python3.12/site-packages/tensorrt_libs:$LD_LIBRARY_PATH
```

### Doğrulama

```bash
source ~/venvs/tensorrt/bin/activate
python -c "import tensorrt; print('TensorRT:', tensorrt.__version__)"
# TensorRT: 10.16.1.11
```

---

## 6. Sürücü ve Donanım Durumu

Tüm bileşenler Ubuntu 26.04'te kutu açık çalışıyor:

| Bileşen | Driver | Modül |
|---|---|---|
| RTX 5070 Ti Mobile | nvidia 595.71.05 | nvidia, nvidia_drm |
| AMD Radeon iGPU | kernel built-in | amdgpu |
| WiFi MT7922 | kernel built-in | mt7921e |
| Ethernet RTL8125 | kernel built-in | r8169 |
| Dahili Ses (SN6140) | kernel built-in | snd_hda_intel |
| HDMI Ses | kernel built-in | snd_hda_intel |

---

## 7. Kurulum Özeti ve Versiyon Tablosu

| Paket | Versiyon | Yöntem |
|---|---|---|
| NVIDIA Driver | 595.71.05 | ubuntu-drivers / apt |
| CUDA Toolkit | 13.1 | apt (NVIDIA repo) |
| cuDNN | 9.0.0 | apt (NVIDIA repo) |
| TensorRT | 10.16.1.11 | pip (Python 3.12 venv) |
| Python (AI stack) | 3.12.13 | deadsnakes PPA |

---

## 8. Faydalı Komutlar

```bash
# GPU ve driver durumu
nvidia-smi

# CUDA derleyici versiyonu
nvcc --version

# Kurulu AI paketleri
dpkg -l | grep -E "cuda|cudnn|nvidia"

# TensorRT versiyon kontrolü
source ~/venvs/tensorrt/bin/activate && python -c "import tensorrt; print(tensorrt.__version__)"

# Kernel modülleri
lsmod | grep -E "nvidia|amdgpu"

# Aktif ses cihazları
aplay -l

# Network durumu
nmcli device status
```

---

## 9. Bilinen Sorunlar ve Çözümler

**TensorRT import hatası: `No module named 'tensorrt'`**
Venv'in Python 3.12 ile oluşturulduğundan emin ol. Python 3.14 ile oluşturulmuş venv'lerde binding paketi kurulmaz.

```bash
rm -rf ~/venvs/tensorrt
python3.12 -m venv ~/venvs/tensorrt
```

**`nvcc: command not found`**
CUDA PATH'i `~/.bashrc`'ye eklenmemiş. Kontrol et:

```bash
echo $PATH | grep cuda
export PATH=/usr/local/cuda/bin:$PATH
```

**SSH bağlantısı: `Host key verification failed`**
GitHub'un host key'ini known_hosts'a ekle:

```bash
ssh-keyscan github.com >> ~/.ssh/known_hosts
```
