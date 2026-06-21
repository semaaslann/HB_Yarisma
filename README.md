# HB_Yarisma
#  HepsiJET Lojistik Anahat Optimizasyonu

## Sonuçlar
| Metrik | Değer |
|--------|-------|
| **Toplam Maliyet** | **8,314,272 TL** |
| Tahmin Modeli | Baseline (Ağırlıksız Ortalama) |
| Tahmin WMAPE | %25.6 |
| Toplam Araç Ataması | 473 |
| Kiralık Araç | 98 |
| Spot Araç | 375 |

---

##  Problem Tanımı
11-17 Mayıs 2026 haftası için:
1. Transfer merkezleri arası **desi talebini tahmin etmek**
2. Tahmin edilen talebi **minimum maliyetle** kiralık ve spot araçlara atamak

---

---

##  Tahmin Modeli
**Yöntem:** Baseline — Son 4 haftanın aynı gününün ortalaması

**Neden Baseline?**
- LightGBM ve XGBoost denendi (MAE: 5.800+)
- Baseline daha iyi performans gösterdi (MAE: 3.891)
- 89 rota × 130 gün = 10.770 satır → ML için yetersiz veri

**Metrikler:**
| Metrik | Değer |
|--------|-------|
| MAE | 3.891 desi |
| MAPE | %62.8 |
| WMAPE | %25.6 |

**Önemli Bulgu:** Güçlü haftalık pattern tespit edildi.
Pazartesi talebi Pazar'ın 7 katı — `day_of_week` en kritik özellik.

---

## ⚙️ Optimizasyon Algoritması
**Yöntem:** Greedy + Kombinasyon Tabanlı Spot Seçimi + Uğrama Optimizasyonu

### Adımlar:
**1. Kiralık Araç Ataması (Zorunlu)**
- Her gün, her kiralık rota için araçlar plana dahil edildi
- Dolu olsun boş olsun maliyet yazıldı (yarışma kuralı)
- Maliyet = Günlük Kira + (km × Km Başına Maliyet)

**2. Spot Araç Optimizasyonu**
- Kiralık kapasite yetersiz kalan rotalar için spot araç seçildi
- Tüm araç kombinasyonları denendi (Kamyonet/Hafif Kamyon/Kamyon/TIR)
- En ucuz kombinasyon seçildi
- %10 doluluk kuralı uygulandı

**3. Uğrama Optimizasyonu**
- Aynı çıkış noktasından aynı gün çıkan spot araçlar birleştirildi
- Coğrafi filtre: Uğramalı yol, direkt yoldan max %20 uzun olabilir
- 1.276 uğrama kabul edildi, 51 coğrafi açıdan mantıksız uğrama elendi

### Versiyon Karşılaştırması:
| Versiyon | Açıklama | Maliyet |
|----------|----------|---------|
| V1 | Basit Greedy | 10,089,310 TL |
| V2 | Kombinasyon tabanlı spot | 10,031,689 TL |
| V3 | Kiralık zorunlu kural eklendi | 10,061,463 TL |
| V4 | Uğrama eklendi (filtresiz) | 8,313,828 TL |
| **V5** | **Uğrama + %20 coğrafi filtre** | **8,314,272 TL** |

---

## 📏 Kısıtlar ve Varsayımlar
- ✅ Kiralık araçlar öncelikli ve zorunlu
- ✅ Minimum %10 araç doluluk oranı (sadece spot)
- ✅ Mesafe: Haversine kuş uçuşu
- ✅ Araçlar dönüş yapmaz
- ✅ Her gün bağımsız değerlendirilir
- ✅ Şoför molası ve yolculuk süresi hesaba katılmadı
- ✅ Sınırsız spot araç erişimi varsayıldı

---

## 🛠️ Kullanılan Teknolojiler
- Python 3.x
- Pandas, NumPy
- LightGBM, XGBoost (denendi)
- Scikit-learn
- Matplotlib
- Itertools (kombinasyon)
