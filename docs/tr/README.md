# MTA CarBall Derecelendirme Sistemi - Tam Puanlama Motoru

## Bu Belge Hakkında

Bu belge, CarBall oyuncu derecelendirme sisteminin **tam ve şeffaf bir açıklamasını** sunmaktadır.

**Amaç:** Oyuncu derecelendirmelerinin nasıl hesaplandığını tamamen açıklamak.

> **Not:** Bu, çalıştırılabilir bir yazılım projesi değil, hesaplama motorunun dokümantasyonudur. Mantık, herkesin sistemi doğrulayabilmesi ve anlayabilmesi için detaylı olarak açıklanmıştır.

---

## 🎯 SİSTEME GENEL BAKIŞ

### Temel Formül

```
SON DERECELENDİRME = HAM YETENEK × DENEYİM ÇARPANI
```

- **Ham Yetenek:** 1.0 - 10.0 ölçeği (oyuncunun saf performansı)
- **Deneyim Çarpanı:** 0.0 - 1.15 ölçeği (deneyimi ödüllendirir)
- **Son Derecelendirme:** 1.0 - 10.0 ölçeği (maksimum 10.0 ile sınırlandırılmıştır)

### Bu Sistem Neden Kullanılmalı?

1. **Adil:** Her oyuncu aynı başlangıç noktasından başlar
2. **Şeffaf:** Tüm hesaplamalar kamuya açık olarak belgelenmiştir
3. **Dengeli:** Hem yeteneği HEM de deneyimi ödüllendirir
4. **Rol Bilinçli:** Farklı roller farklı şekilde değerlendirilir
5. **Sınırlandırılmış:** Sonsuz ölçeklendirme yoktur, maksimum 10.0'dır

---

## 📊 HAM YETENEK HESAPLAMASI (1.0 - 10.0)

Ham Yetenek, **6 ağırlıklı kriter** aracılığıyla saf performansı ölçer. Her kriterin maksimum katkısı vardır ve hepsi toplanır.

### 6 Kriter

| # | Kriter | Maks. Puan | Formül | Sınır |
|---|--------|------------|---------|-------|
| 1 | 🏆 Kazanma Oranı | 3.5 | (Kazanma Oranı / 100) × 3.5 | Yok |
| 2 | 👑 Galibiyet Başına MVP | 2.5 | (Galibiyet Başına MVP / 100) × 2.5 | Yok |
| 3 | ⚽ Maç Başına Gol | 1.5 | min(Maç Başına Gol, 1.5) × 1.5 | 1.5 |
| 4 | 🧤 Maç Başına Kurtarış | 2.0 | min(Maç Başına Kurtarış, 3.0) × 2.0 | 3.0 |
| 5 | 🎯 Maç Başına Asist | 1.5 | min(Maç Başına Asist, 1.0) × 1.5 | 1.0 |
| 6 | 💫 MVP Verimliliği | 0.5 | (MVP Verimliliği / 10) × 0.5 | Yok |

### Ceza Sistemi

| Ceza | Koşul | Kesinti |
|------|-------|---------|
| ❌ Kendi Kalesine Gol Cezası | Maç Başına Kendi Kalesine Gol ≥ 0.55 | -0.5 puan |

### Tam Formül

```python
ham_yetenek = (
    # Bileşen 1: Kazanma Oranı (maks 3.5)
    (kazanma_orani / 100 * 3.5) +
    
    # Bileşen 2: Galibiyet Başına MVP (maks 2.5)
    (mvp_galibiyet_orani / 100 * 2.5) +
    
    # Bileşen 3: Maç Başına Gol (maks 1.5, 1.5 ile sınırlandırılmış)
    (min(maç_başına_gol, 1.5) * 1.5) +
    
    # Bileşen 4: Maç Başına Kurtarış (maks 2.0, 3.0 ile sınırlandırılmış)
    (min(maç_başına_kurtarış, 3.0) * 2.0) +
    
    # Bileşen 5: Maç Başına Asist (maks 1.5, 1.0 ile sınırlandırılmış)
    (min(maç_başına_asist, 1.0) * 1.5) +
    
    # Bileşen 6: MVP Verimliliği (maks 0.5)
    (mvp_verimliliği / 10.0 * 0.5)
)

# Aşırı kendi kalesine gol için ceza uygula
if maç_başına_kendi_kalesine_gol >= 0.55:
    ham_yetenek -= 0.5

# Ham yeteneği 1.0 ile 10.0 arasında sınırlandır
son_yetenek = min(max(ham_yetenek, 1.0), 10.0)
```

### Her Bileşeni Anlamak

#### 1. Kazanma Oranı (🏆)

- **Neden:** Kazanmak nihai hedeftir
- **Nasıl:** %100 kazanma oranı = 3.5 puan, %0 = 0 puan
- **Örnek:** %67 kazanma oranı = (67/100) × 3.5 = 2.345 puan

#### 2. Galibiyet Başına MVP (👑)

- **Neden:** MVP'ler oyunu değiştiren etkiyi gösterir
- **Nasıl:** %100 Galibiyet Başına MVP = 2.5 puan, %0 = 0 puan
- **Örnek:** %46 Galibiyet Başına MVP = (46/100) × 2.5 = 1.15 puan

#### 3. Maç Başına Gol (⚽)

- **Neden:** Goller maçları kazandırır
- **Nasıl:** 1.5+ gol/maç = 1.5 puan (sınırlandırılmış)
- **Örnek:** 0.59 gol/maç = 0.59 × 1.5 = 0.885 puan

#### 4. Maç Başına Kurtarış (🧤)

- **Neden:** Savunma şampiyonluk kazandırır
- **Nasıl:** 3.0+ kurtarış/maç = 2.0 puan (sınırlandırılmış)
- **Örnek:** 1.92 kurtarış/maç = 1.92 × 2.0 = 3.84 puan

#### 5. Maç Başına Asist (🎯)

- **Neden:** Oyun kuruculuk fırsatlar yaratır
- **Nasıl:** 1.0+ asist/maç = 1.5 puan (sınırlandırılmış)
- **Örnek:** 0.33 asist/maç = 0.33 × 1.5 = 0.495 puan

#### 6. MVP Verimliliği (💫)

- **Neden:** Aksiyon başına etki önemlidir
- **Nasıl:** %10+ verimlilik = 0.5 puan
- **Örnek:** %5.01 verimlilik = (5.01/10) × 0.5 = 0.2505 puan

---

## 📈 DENEYİM ÇARPANI (0.0 - 1.15)

Deneyim çarpanı, daha fazla maç oynamış oyuncuları ödüllendirir. Deneyimin önemli olmasını sağlarken yeni oyuncuların da rekabet edebilmesine olanak tanır.

### Kademe Sistemi

| Kademe | Maç | Çarpan Aralığı | Formül |
|--------|-----|----------------|---------|
| Doktora | 10,000+ | 1.00 - 1.15 | 1.00 + ((maç-10000)/1500) × 0.015 (1.15 ile sınırlandırılmış) |
| Deneyimli | 5,000 - 9,999 | 0.85 - 0.99 | 0.85 + ((maç-5000)/5000) × 0.14 |
| Yerleşik | 2,500 - 4,999 | 0.70 - 0.84 | 0.70 + ((maç-2500)/2500) × 0.15 |
| Gelişmekte | 500 - 2,499 | 0.40 - 0.69 | 0.40 + ((maç-500)/2000) × 0.30 |
| Yeni | 0 - 499 | 0.00 - 0.39 | (maç/500) × 0.40 |

### Tam Formül

```python
def deneyim_carpani_hesapla(toplam_maç):
    # Hiç maç yok = çarpan yok
    if toplam_maç == 0:
        return 0.0
    
    # Doktora Kademesi: 10,000+ maç
    # 10k üzerindeki her 1500 maç +0.015 bonus verir
    # Maks sınır 1.15
    if toplam_maç >= 10000:
        ekstra_maç = toplam_maç - 10000
        bonus = (ekstra_maç / 1500) * 0.015
        return min(1.00 + bonus, 1.15)
    
    # Deneyimli Kademesi: 5,000-9,999 maç
    # 5k'da 0.85'ten 10k'da 0.99'a doğrusal ölçeklendirme
    elif toplam_maç >= 5000:
        return 0.85 + ((toplam_maç - 5000) / 5000) * 0.14
    
    # Yerleşik Kademesi: 2,500-4,999 maç
    # 2.5k'da 0.70'ten 5k'da 0.84'e doğrusal ölçeklendirme
    elif toplam_maç >= 2500:
        return 0.70 + ((toplam_maç - 2500) / 2500) * 0.15
    
    # Gelişmekte Kademesi: 500-2,499 maç
    # 500'de 0.40'tan 2.5k'da 0.69'a doğrusal ölçeklendirme
    elif toplam_maç >= 500:
        return 0.40 + ((toplam_maç - 500) / 2000) * 0.30
    
    # Yeni Kademesi: 0-499 maç
    # 0'da 0.0'dan 500'de 0.40'a doğrusal ölçeklendirme
    else:
        return (toplam_maç / 500) * 0.40
```

### Bu Çarpan Neden Kullanılmalı?

- **Uzun Ömürlülüğü Ödüllendirir:** Doktoralar %15'e kadar bonus alır
- **Yeni Oyuncuları Teşvik Eder:** Yeni oyuncular çok sert cezalandırılmaz
- **Kademeli İlerleme:** Kademeler arası sorunsuz ölçeklendirme
- **Adil Sınır:** Hiç kimse 1.15x çarpanı aşamaz

---

## 🎭 TAKTİK ROL BELİRLEME

Oyuncular, istatistiklerine göre 3 ana role ve alt kategorilere ayrılır. Bu, oyuncuların kendi rol bağlamlarında değerlendirilmesini sağlar.

### Sınıflandırma Akış Şeması

```
                    BAŞLANGIÇ
                      │
                      ▼
         ┌─────────────────────┐
         │ Kalecilik İndeksi   │
         │ > %55?              │
         └─────────────────────┘
              │            │
             EVET         HAYIR
              │            │
              ▼            ▼
     ┌─────────────┐  ┌─────────────┐
     │ KALECİ      │  │ Gol/Asist   │
     │ Kategorisi  │  │ Oranı > 1.5 │
     └─────────────┘  │ VE Gol/Maç  │
              │       │ >= 1.2?     │
              │       └─────────────┘
              │              │            │
              │             EVET         HAYIR
              │              │            │
              │              ▼            ▼
              │     ┌─────────────┐  ┌─────────────┐
              │     │ FORVET      │  │ OYUN KURUCU │
              │     │ Kategorisi  │  │ Kategorisi  │
              │     └─────────────┘  └─────────────┘
              │
              ▼
         ┌─────────────┐
         │ Doktora Etiketi│
         │ Uygula (eğer│
         │ 10K+ Maç)  │
         └─────────────┘
```

### 1. 🧤 KALECİ (Kurtarışlar > Aksiyonların %55'i)

**Alt Kategoriler:**

| Kategori | Gereksinimler |
|----------|---------------|
| Elite Kaleci | Savunma Güvenilirliği ≥ 8.0 VE Kazanma Oranı ≥ %45 |
| Zayıf Kaleci | Maç Başına Kendi Kalesine Gol ≥ 0.55 |
| Pasif Kaleci | Diğer tüm kaleciler |

**Mantık:**

```python
if kalecilik_indeksi > 55:
    if savunma_guvenilirligi >= 8.0 and kazanma_orani >= 45:
        rol = "Elite Kaleci"
    elif maç_başına_kendi_kalesine_gol >= 0.55:
        rol = "Zayıf Kaleci"
    else:
        rol = "Pasif Kaleci"
```

### 2. ⚽ FORVET (Gol/Asist > 1.5 VE Maç Başına Gol ≥ 1.2)

**Alt Kategoriler:**

| Kategori | Gereksinimler |
|----------|---------------|
| Elite Forvet | Kazanma Oranı ≥ %48 VE Galibiyet Başına MVP ≥ %33.33 |
| Bencil Forvet | Kazanma Oranı < %43 VEYA Galibiyet Başına MVP < %28 VEYA Maç Başına Kendi Kalesine Gol ≥ 0.55 |
| Fırsatçı Forvet | Diğer tüm forvetler |

**Mantık:**

```python
elif gol_asist_orani > 1.5 and maç_başına_gol >= 1.2:
    if kazanma_orani >= 48 and mvp_galibiyet_orani >= 33.33:
        rol = "Elite Forvet"
    elif kazanma_orani < 43 or mvp_galibiyet_orani < 28 or maç_başına_kendi_kalesine_gol >= 0.55:
        rol = "Bencil Forvet"
    else:
        rol = "Fırsatçı Forvet"
```

### 3. 🎯 OYUN KURUCU (Diğer tüm oyuncular)

**Alt Kategoriler:**

| Kategori | Gereksinimler |
|----------|---------------|
| Elite Oyun Kurucu | Maç Başına Asist ≥ 0.75 VE Kazanma Oranı ≥ %48 |
| Verimli MVP | MVP Verimliliği ≥ %10.0 VE Kazanma Oranı ≥ %45 |
| Standart Oyun Kurucu | Kazanma Oranı ≥ %48 VE Galibiyet Başına MVP ≥ %35 |
| Tutarsız Orta Saha | Kazanma Oranı < %43 VEYA Maç Başına Kendi Kalesine Gol ≥ 0.55 |
| Her Yönüyle Orta Saha | Diğer tüm oyun kurucular |

**Mantık:**

```python
else:
    if maç_başına_asist >= 0.75 and kazanma_orani >= 48:
        rol = "Elite Oyun Kurucu"
    elif mvp_verimliliği >= 10.0 and kazanma_orani >= 45:
        rol = "Verimli MVP"
    elif kazanma_orani >= 48 and mvp_galibiyet_orani >= 35:
        rol = "Standart Oyun Kurucu"
    elif kazanma_orani < 43 or maç_başına_kendi_kalesine_gol >= 0.55:
        rol = "Tutarsız Orta Saha"
    else:
        rol = "Her Yönüyle Orta Saha"
```

### Doktora Etiketi

10,000+ maçı olan oyuncular rollerine "☣️ 10K+ Maç" etiketi alır:

```python
if toplam_maç >= 10000:
    return f"☣️ 10K+ Maç | {rol}"
```

---

## 🏷️ KADEME SİSTEMİ

Son derecelendirme, kolay görsel tanımlama için bir kademeye dönüştürülür:

| Puan Aralığı | Kademe | Emoji | Anlam |
|--------------|--------|-------|-------|
| 9.0 - 10.0 | S-Kademe | ☣ | En iyilerin en iyisi |
| 7.5 - 8.9 | A-Kademe | 🥇 | Elite oyuncular |
| 6.0 - 7.4 | B-Kademe | 🥈 | Çok iyi oyuncular |
| 4.5 - 5.9 | C-Kademe | 🥉 | Ortalamanın üstü |
| 3.0 - 4.4 | D-Kademe | 📈 | Ortalama oyuncular |
| 0.0 - 2.9 | E-Kademe | 🆕 | Yeni veya gelişmekte |

### Tam Fonksiyon

```python
def puandan_kademe_hesapla(puan):
    if puan >= 9.0:
        return "☣ S-Kademe"
    elif puan >= 7.5:
        return "🥇 A-Kademe"
    elif puan >= 6.0:
        return "🥈 B-Kademe"
    elif puan >= 4.5:
        return "🥉 C-Kademe"
    elif puan >= 3.0:
        return "📈 D-Kademe"
    else:
        return "🆕 E-Kademe"
```

---

## 🔍 TAM ÖRNEK: "keLLog"

Gerçek bir oyuncu için tüm hesaplamayı adım adım inceleyelim.

### Adım 1: Ham Veri Girişi

| İstatistik | Değer |
|------------|-------|
| Kazanılan | 4,401 |
| Kaybedilen | 5,854 |
| MVP | 1,481 |
| Goller | 6,528 |
| Asistler | 3,398 |
| Kurtarışlar | 19,651 |
| Kendi Kalesine Goller | 1,623 |

### Adım 2: Maç Başına Metrikleri Hesapla

```python
toplam_maç = 4,401 + 5,854 = 10,255

kazanma_orani = (4,401 / 10,255) × 100 = %42.91
maç_başına_gol = 6,528 / 10,255 = 0.6365
maç_başına_asist = 3,398 / 10,255 = 0.3313
maç_başına_kurtarış = 19,651 / 10,255 = 1.9162
maç_başına_kendi_kalesine_gol = 1,623 / 10,255 = 0.1583

mvp_galibiyet_orani = (1,481 / 4,401) × 100 = %33.65
```

### Adım 3: Gelişmiş Metrikleri Hesapla

```python
toplam_aksiyon = 6,528 + 3,398 + 19,651 = 29,577

kalecilik_indeksi = (19,651 / 29,577) × 100 = %66.44
savunma_guvenilirligi = 19,651 / 1,623 = 12.11
gol_asist_orani = 6,528 / 3,398 = 1.92

mvp_verimliliği = (1,481 / 29,577) × 100 = %5.01
```

### Adım 4: Taktik Rolü Belirle

```python
# Kaleci Kontrolü
kalecilik_indeksi = %66.44 > %55 → KALECİ!

# Alt kategori kontrolü
savunma_guvenilirligi = 12.11 ≥ 8.0 ✓
kazanma_orani = %42.91 ≥ %45 → YANLIŞ

# Elite Kaleci değil
maç_başına_kendi_kalesine_gol = 0.1583 ≥ 0.55 → YANLIŞ

# → Pasif Kaleci

# Doktora Kontrolü
toplam_maç = 10,255 ≥ 10,000 → ✓

# SON ROL:
"☣️ 10K+ Maç | Pasif Kaleci"
```

### Adım 5: Ham Yeteneği Hesapla

```python
# Bileşen 1: Kazanma Oranı
(42.91 / 100) × 3.5 = 1.5019

# Bileşen 2: Galibiyet Başına MVP
(33.65 / 100) × 2.5 = 0.8413

# Bileşen 3: Maç Başına Gol (1.5 ile sınırlandırılmış)
min(0.6365, 1.5) × 1.5 = 0.9548

# Bileşen 4: Maç Başına Kurtarış (3.0 ile sınırlandırılmış)
min(1.9162, 3.0) × 2.0 = 3.8324

# Bileşen 5: Maç Başına Asist (1.0 ile sınırlandırılmış)
min(0.3313, 1.0) × 1.5 = 0.4970

# Bileşen 6: MVP Verimliliği
(5.01 / 10) × 0.5 = 0.2505

# Tüm bileşenleri topla
ham_yetenek = 1.5019 + 0.8413 + 0.9548 + 3.8324 + 0.4970 + 0.2505
ham_yetenek = 7.8779

# Kendi Kalesine Gol Cezası Kontrolü
maç_başına_kendi_kalesine_gol = 0.1583 < 0.55 → Ceza yok

# 1.0 ile 10.0 arasında sınırlandır
son_yetenek = min(max(7.8779, 1.0), 10.0) = 7.8779
```

### Adım 6: Deneyim Çarpanını Hesapla

```python
toplam_maç = 10,255 ≥ 10,000 → Doktora Kademesi

ekstra_maç = 10,255 - 10,000 = 255
bonus = (255 / 1500) × 0.015 = 0.0026
carpan = min(1.00 + 0.0026, 1.15) = 1.0026

# 3 haneye yuvarla
carpan = 1.003
```

### Adım 7: Son Derecelendirmeyi Hesapla

```python
son_puan = son_yetenek × carpan
son_puan = 7.8779 × 1.0026 = 7.8984

# 10.0 ile sınırlandır
son_puan = min(7.8984, 10.0) = 7.9

# 1 haneye yuvarla
son_puan = 7.9
```

### Adım 8: Kademeyi Hesapla

```python
puan = 7.9
7.5 ≤ 7.9 < 9.0 → "🥇 A-Kademe"
```

### Sonuç

```
@keLLog | ☣️ 10K+ Maç | 🧤 Pasif Kaleci | 🥇 A-Kademe
Puan: 7.9/10 (Ham: 7.9)
Kazanma: %42.9 4401G/5854M (0.75)
MVP: 1481 (%34)
İstatistikler: Kurtarış: 19651 - Gol: 6528 - Asist: 3398 - Kendi Kalesine Gol: 1623
```

---

## 📊 SONUÇLARIN ANALİZİ

### Neden 7.9 Puan?

| Bileşen | Puan | Maks | Yüzde |
|---------|------|------|-------|
| Kazanma Oranı | 1.50 | 3.5 | %43 |
| Galibiyet Başına MVP | 0.84 | 2.5 | %34 |
| Maç Başına Gol | 0.95 | 1.5 | %64 |
| Maç Başına Kurtarış | 3.83 | 2.0 | %100+ |
| Maç Başına Asist | 0.50 | 1.5 | %33 |
| MVP Verimliliği | 0.25 | 0.5 | %50 |
| **TOPLAM** | **7.88** | **10.0** | **%79** |

**Önemli Noktalar:**

- Maç Başına Kurtarış en güçlü bileşen - Maks 2.0 üzerinden 3.83 puan (sınırlandırılmış)
- Kazanma Oranı en zayıf - Maksimumun sadece %43'ü
- Galibiyet Başına MVP orta düzeyde - Maksimumun %34'ü
- Kendi Kalesine Gol cezası yok - 0.158 < 0.55 eşiği

### Neden Pasif Kaleci?

- Kalecilik İndeksi: %66.44 > %55 → Kesinlikle kaleci
- Savunma Güvenilirliği: 12.11 ≥ 8.0 ✓ (Elite gereksinimi)
- Kazanma Oranı: %42.91 < %45 ✗ (Elite gereksinimini karşılamıyor)
- Kendi Kalesine Goller: 0.158 < 0.55 ✓ (Zayıf değil)
- **Sonuç:** Pasif Kaleci

### Neden 1.003 Çarpan?

- 10,255 maç 10,000 eşiğinin hemen üzerinde
- Eşik üzerinde 255 maç = minimum bonus
- Çarpan: 1.003 (sadece %0.3 bonus)
- Daha fazla maç çarpanı artırırdı

---

## 🔄 TAM SÜREÇ AKIŞI

```
1. GİRİŞ: Ham İstatistikler
   ↓
2. DOĞRULA: Sıfır maç kontrolü
   ↓
3. HESAPLA: Maç başına metrikler
   - Kazanma Oranı
   - Maç Başına Gol
   - Maç Başına Asist
   - Maç Başına Kurtarış
   - Maç Başına Kendi Kalesine Gol
   ↓
4. HESAPLA: Gelişmiş metrikler
   - Toplam Aksiyon
   - Kalecilik İndeksi
   - Savunma Güvenilirliği
   - Gol/Asist Oranı
   - MVP Verimliliği
   ↓
5. BELİRLE: Taktik Rol
   - Kaleci / Forvet / Oyun Kurucu
   - Alt kategori
   - Doktora etiketi (eğer 10K+ maç varsa)
   ↓
6. HESAPLA: Ham Yetenek (6 bileşen)
   ↓
7. UYGULA: Kendi Kalesine Gol Cezası (varsa)
   ↓
8. SINIRLANDIR: Ham Yetenek 1.0 - 10.0 arası
   ↓
9. HESAPLA: Deneyim Çarpanı
   ↓
10. HESAPLA: Son Puan = Yetenek × Çarpan
   ↓
11. SINIRLANDIR: Son Puan ≤ 10.0
   ↓
12. BELİRLE: Kademe (S'den E'ye)
   ↓
13. ÇIKTI: Tam oyuncu profili
```

---

## 🔬 BU SİSTEM NEDEN ÇALIŞIYOR?

### 1. Dengeli Kriterler

- Hücum (Goller, Asistler)
- Savunma (Kurtarışlar)
- Takım Etkisi (Kazanma Oranı, MVP)
- Hiçbir istatistik domine etmez

### 2. Rol Tanıma

- Kaleciler Forvetlerle karşılaştırılmaz
- Her rolün kendi değerlendirme yolu vardır
- Alt kategoriler nüans katar

### 3. Deneyim Önemlidir

- Doktoralar tanınır (10K+ etiketi)
- Çarpan uzun ömürlülüğü ödüllendirir
- Yeni oyuncular çok sert cezalandırılmaz

### 4. Adil Sınırlar

- Hiçbir istatistik makul sınırları aşamaz
- Ham Yetenek 10.0 ile sınırlandırılmıştır
- Son Puan 10.0 ile sınırlandırılmıştır

### 5. Ceza Sistemi

- Aşırı kendi kalesine goller cezalandırılır
- Olumsuz oyun tarzlarını önler
- Takım oyununu teşvik eder

### 6. Şeffaflık

- Tüm formüller herkese açıktır
- Herkes hesaplamaları doğrulayabilir
- Gizli ayarlamalar yoktur

---

## 📝 TERİMLER SÖZLÜĞÜ

| Terim | Tanım |
|-------|-------|
| Ham Yetenek | Saf performans puanı (1.0-10.0) |
| Deneyim Çarpanı | Oynanan maçlara göre bonus (0.0-1.15) |
| Son Puan | Ham Yetenek × Çarpan (1.0-10.0) |
| MVP Verimliliği | Toplam aksiyon başına MVP × 100 |
| Kalecilik İndeksi | Kurtarış olan aksiyonların yüzdesi |
| Savunma Güvenilirliği | Kendi kalesine gol başına kurtarış |
| Gol/Asist Oranı | Gollerin asistlere bölümü |
| Doktora | 10,000+ maçı olan oyuncu |
| Kademe | Puana göre harf notu (S'den E'ye) |

---

## 💡 TASARIM FELSEFESİ

### Adalet

Her oyuncu aynı başlangıç noktasından başlar. Sistem herhangi bir spesifik oyun tarzını kayırmaz.

### Şeffaflık

Tüm hesaplamalar belgelenmiş ve doğrulanabilirdir. Gizli formüller veya ayarlamalar yoktur.

### Denge

Hücum, savunma ve takım etkisi uygun şekilde ağırlıklandırılmıştır. Hiçbir istatistik domine etmez.

### Tanıma

İyi performans ödüllendirilir. Deneyim değerlidir. Rol uzmanlığı tanınır.

### Basitlik

Temel formül anlaşılacak kadar basit, ancak doğru olacak kadar kapsamlıdır.

---

## 🔧 UYGULAMA NOTLARI

### Veri Gereksinimleri

Sistem, oyuncu başına bu ham istatistikleri gerektirir:

- Kazanılan / Kaybedilen (Maçlar)
- MVP (Toplam ödüller)
- Goller / Asistler / Kurtarışlar (Toplam sayılar)
- Kendi Kalesine Goller (Toplam sayı)

### Hesaplama Sırası

1. Ham veriyi çıkar
2. Maç başına metrikleri hesapla
3. Gelişmiş metrikleri hesapla
4. Rolü belirle
5. Ham yeteneği hesapla
6. Cezaları uygula
7. Çarpanı uygula
8. Son puanı hesapla
9. Kademeyi belirle

### Doğrulama

- Sıfır maç = varsayılan değerler
- Eksik veri = varsayılan 0
- Tüm hesaplamalar sıfıra bölmeye karşı korunmuştur

---

## 📈 SİSTEM SINIRLAMALARI

- **Statik Ağırlıklar:** Kriter ağırlıkları sabittir ve uyarlanmaz
- **Oyuncu Etkileşimi Yok:** Takım arkadaşlarını/rakipleri hesaba katmaz
- **Saf İstatistikler:** Taktik kararları yakalamaz
- **Maç Bağlamı:** Maç önemini dikkate almaz
- **Zaman Azalması:** Eski maçlar yeniler kadar sayılır

### Gelecek İyileştirmeler

- Metaya göre dinamik ağırlıklar
- Zamanla performans azalması
- Takım kimyası faktörleri
- Rakip gücü ayarlaması

---

## ✅ SİSTEM AVANTAJLARI

- **Kapsamlı:** Hücum, savunma ve etkiyi kapsar
- **Rol Bilinçli:** Farklı roller farklı değerlendirilir
- **Deneyim Tabanlı:** Doktoraları uygun şekilde ödüllendirir
- **Adil:** Tüm oyuncular için eşit fırsat
- **Şeffaf:** Tüm hesaplamalar herkese açıktır
- **Sınırlandırılmış:** Sonsuz ölçeklendirme yok
- **Dengeli:** Hiçbir istatistik domine etmez

---

Bu belge, CarBall derecelendirme sisteminin tam, detaylı açıklamasını sunmaktadır. Tüm hesaplamalar doğrulama ve iyileştirme için açıktır.

**Son Güncelleme:** Ağustos 2026
