<div align="center">

# 📡 Telco Müşteri Kaybı (Churn) Tahmini

**Hangi müşteri aboneliğini iptal edecek? Makine öğrenmesi ile erken tespit.**

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=for-the-badge&logo=scikitlearn&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![Google Colab](https://img.shields.io/badge/Colab-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)

</div>

---

## 🎯 Problem

Telekom şirketleri için yeni müşteri kazanmak, mevcut müşteriyi elde tutmaktan çok daha pahalıdır.
Bu çalışmada, **ayrılma riski taşıyan müşterileri önceden tespit eden** bir sınıflandırma modeli geliştirildi.
Böylece şirket bu müşterilere gitmeden önce kampanya veya indirim sunabilir.

## 📊 Veri Seti

| Özellik | Değer |
|---|---|
| **Kaynak** | [IBM Telco Customer Churn](https://github.com/IBM/telco-customer-churn-on-icp4d) |
| **Gözlem sayısı** | 7.043 müşteri |
| **Sütun sayısı** | 21 (demografik bilgiler, hizmetler, sözleşme, ödeme) |
| **Hedef değişken** | `Churn` – müşteri ayrıldı mı? (Yes / No) |
| **Sınıf dağılımı** | %73,5 kaldı · %26,5 ayrıldı → **dengesiz veri** ⚖️ |

## 🔄 Proje Akışı

```
Veri Yükleme → Veri Temizleme → Kodlama → Bölme (70/15/15) → Ölçekleme → Modelleme → Değerlendirme
```

### 🧹 1. Veri Temizleme
- `customerID` sütunu kaldırıldı (öğrenilecek bir kalıp taşımıyor)
- `TotalCharges` sütununda **gizli eksik değerler** bulundu: 11 satırda sayı yerine boşluk karakteri vardı.
  Bu müşterilerin abonelik süresi 0 olduğu için (yeni müşteriler) değerler **0** ile dolduruldu.

### 🔢 2. Kategorik Kodlama
- Hedef değişken: `Yes → 1`, `No → 0`
- Diğer kategorik sütunlar: **One-Hot Encoding** (`get_dummies`), sahte sıralama oluşmasın diye
- Sonuç: 30 özellik

### ✂️ 3. Veri Bölme
| Küme | Oran | Amaç |
|---|---|---|
| Train | %70 | Modelin öğrenmesi |
| Validation | %15 | Hiperparametre ve model seçimi |
| Test | %15 | Son ve tarafsız değerlendirme |

`stratify` ile her kümede churn oranı korundu.

### 📏 4. Ölçekleme
`MinMaxScaler` **yalnızca train verisine** fit edildi; validation ve test kümelerine sadece transform uygulandı (**veri sızıntısı önlendi**).

### 🤖 5. Modelleme

| Model | Ayarlanan Parametre | Not |
|---|---|---|
| k-NN | `k` ∈ {1, 3, 5, 11, 21, 51} | Mesafe temelli → ölçekli veri kullanıldı |
| Karar Ağacı | `max_depth` ∈ {1, 3, 5, 8, 15, None} | Eşik temelli → ölçekleme gerekmedi |
| Random Forest | 200 ağaç, `max_depth=8` | Topluluk (ensemble) yöntemi |

**Gözlemler:**
- 🔴 `k=1` ve `max_depth=None` → **Overfitting** (eğitimde ~%99,8, doğrulamada ~%73)
- 🟡 `max_depth=1` → **Underfitting** (herkese "kalır" diyen modelle aynı sonuç)
- 🟢 `k=21` ve `max_depth=5` → **İyi uyum**

## 🏆 Sonuçlar (Validation)

| Model | Accuracy | Precision | Recall | F1 |
|---|:---:|:---:|:---:|:---:|
| Taban çizgisi (herkese "kalır") | 0.735 | – | 0.000 | – |
| k-NN (k=21) | 0.796 | 0.636 | 0.585 | 0.610 |
| **Karar Ağacı (max_depth=5)** ⭐ | **0.811** | 0.654 | **0.645** | **0.649** |
| Random Forest | 0.803 | **0.687** | 0.505 | 0.582 |

### 💡 Neden Recall ve F1?

| Hata | Anlamı | İş etkisi |
|---|---|---|
| **FP** (yalancı alarm) | Kalacak müşteriye "gidecek" denir | Boşuna indirim verilir → *küçük maliyet* |
| **FN** (kaçırılan müşteri) | Gidecek müşteriye "kalacak" denir | Müşteri kaybedilir → **büyük maliyet** |

FN daha maliyetli olduğu için model **Recall ve F1**'e göre seçildi.
Dikkat: Hiçbir şey öğrenmeyen taban çizgisi modeli bile **%73,5 accuracy** alıyor. Bu, dengesiz veride accuracy'nin tek başına yanıltıcı olduğunu gösteriyor.

### 🧩 Karmaşıklık Matrisi (Test)

<img src="karmasiklik_matrisi.png" width="400">

## 🚀 Çalıştırma

1. Repoyu indir veya notebook'u Colab'da aç
2. `Telco-Customer-Churn.csv` dosyasını notebook ile aynı klasöre koy
3. Tüm hücreleri sırayla çalıştır

```bash
pip install pandas numpy scikit-learn matplotlib seaborn
```

## 🛠️ Kullanılan Araçlar

`Python` · `pandas` · `NumPy` · `scikit-learn` · `matplotlib` · `seaborn` · `Google Colab`

---

<div align="center">

👩‍💻 **Beyza Nur Dinçer** · Yönetim Bilişim Sistemleri, KTÜ
[GitHub](https://github.com/BeyzaaNuur)

</div>
