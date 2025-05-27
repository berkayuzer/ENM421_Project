# Ruh Sağlığı Verileriyle Depresyon Tahmini

Bu proje, bireylerin yaşam tarzı, demografik bilgileri ve çeşitli stres faktörlerine dayanarak depresyon riskini tahmin etmeyi amaçlayan bir makine öğrenmesi modelini içermektedir. Erken teşhisin öneminden yola çıkarak, veri bilimi teknikleriyle depresyon olasılığını belirlemeye yönelik bir model geliştirilmiştir.

## Proje Açıklaması

Depresyon, dünya genelinde yaklaşık 300 milyon insanı etkileyen ciddi bir sağlık sorunudur (WHO, 2021). Bu projenin temel amacı, sağlanan veri setindeki özellikleri kullanarak bir bireyin depresyonda olup olmadığını sınıflandıran bir model oluşturmaktır. Modelin, özellikle depresyon vakalarını (azınlık sınıfı) yüksek bir duyarlılıkla (recall) tespit etmesi hedeflenmiştir.

## Veri Seti

Bu çalışmada kullanılan veri seti, bireylerin aşağıdaki gibi çeşitli bilgilerini içermektedir:

*   Demografik bilgiler (Yaş, Cinsiyet, Şehir)
*   Çalışma/Öğrenim durumu ve meslek
*   Akademik ve iş baskısı seviyeleri
*   Not ortalaması (CGPA)
*   Ders ve iş memnuniyeti
*   Uyku düzeni ve yeme alışkanlıkları
*   Eğitim seviyesi (Degree)
*   İntihar düşünceleri olup olmadığı
*   Çalışma/ders saatleri
*   Finansal stres
*   Ailede ruhsal hastalık öyküsü
*   **Hedef Değişken:** Depresyon (Var/Yok)

## Kullanılan Yöntemler

Proje aşağıdaki temel adımları içermektedir:

1.  **Keşifsel Veri Analizi (EDA):**
    *   Veri setinin genel yapısının incelenmesi.
    *   Özellikler arasındaki ilişkilerin ve hedef değişkenle olan korelasyonların analizi.
    *   Sınıf dengesizliği gibi potansiyel sorunların tespiti.
    *   Önemli bulguların görselleştirilmesi (örneğin, cinsiyete göre depresyon dağılımı, yaş ve depresyon ilişkisi).

2.  **Veri Temizleme ve Ön İşleme:**
    *   **Eksik Veri Yönetimi:**
        *   Sayısal özellikler için eksik değerler medyan ile dolduruldu (`SimpleImputer`).
        *   Kategorik özellikler için eksik değerler en sık görülen değer ile dolduruldu (`SimpleImputer`).
    *   **Kategorik Veri Düzeltme:**
        *   "Dietary Habits" sütunundaki kategoriler, "Healthy", "Unhealthy", "Moderate" ve "NaN" olarak; "Sleep Duration" sütunundaki kategoriler ise “<5 saat”, “5-6 saat”, “6-7 saat”, “7-8 saat”, “>8 saat” ve “NaN” olarak yeniden haritalandırıldı.
        *   "Working Professional or Student" bilgisinden faydalanılarak "Profession", "Academic Pressure" ve "Work Pressure" sütunları mantıksal olarak düzenlendi.
        *   "Study Satisfaction" ve "Job Satisfaction" sütunları birleştirilerek tek bir memnuniyet ölçütü oluşturuldu.
        *   Profession, “City ve “Degree" sütunlarında yer alan 5'ten az olan değerler "Other" olarak sınıflandırıldı.
    *   **Özellik Mühendisliği:**
        *   Mesleklere dayalı olarak ortalama "Salary" (Maaş) özelliği oluşturuldu.
        *   Mesleklerden yola çıkarak "Work Environment" (Çalışma Ortamı) kategorik özelliği türetildi.
        *   "Hourly Salary" (Saatlik Maaş) ve "Income by Age" (Yaşa Göre Gelir) gibi yeni sayısal özellikler eklendi.
    *   **Kategorik Veri Kodlaması:** Tüm kategorik özellikler, makine öğrenmesi modellerinin işleyebilmesi için `OrdinalEncoder` kullanılarak sayısal formata dönüştürüldü.
    *   **Veri Dengesizliğiyle Başa Çıkma:** Eğitim setindeki sınıf dengesizliğini, depresyon vakalarının azınlıkta olmasını, gidermek için **SMOTE (Synthetic Minority Over-sampling Technique)** kullanıldı.

3.  **Model Geliştirme ve Değerlendirme:**
    *   **Kullanılan Algoritmalar:**
        *   LightGBM
        *   CatBoost
        *   XGBoost
    *   **Hiperparametre Optimizasyonu:** Her bir model için en iyi hiperparametreler **Optuna** kütüphanesi kullanılarak bulundu. Optimizasyon sırasında temel değerlendirme metriği olarak 'accuracy' (doğruluk) kullanıldı.
    *   Optimizasyon sürecinde, model performansının daha güvenilir bir şekilde değerlendirilmesi ve veri setindeki sınıf dağılımının her katlamada (fold) korunması için **K-Fold Çapraz Doğrulama (Cross-Validation)** tekniği (k=5 olarak) uygulandı.
    *   **Sınıf Dengesizliği Problemi:**
        *   LightGBM: `class_weight="balanced"`
        *   CatBoost: `auto_class_weights="Balanced"`
        *   XGBoost: `scale_pos_weight` (eğitim verisindeki sınıf oranlarına göre hesaplandı)
    *   **Ensemble Learning (Topluluk Öğrenmesi):**
        *   Üç modelin tahminlerini birleştirmek için **Voting Classifier** (Oylama Sınıflandırıcısı) kullanıldı.
        *   `voting='soft'` stratejisi seçilerek modellerin olasılık tahminlerinin ortalaması alındı.
    *   **Tahmin Eşiği Ayarlaması:**
        *   Modelin depresyon sınıfını daha iyi yakalaması için, validasyon seti üzerindeki F1 skoru maksimize edilerek optimum tahmin eşiği belirlendi.
    *   **Değerlendirme Metrikleri:** Accuracy, Precision, Recall ve F1 Score.

## Özet Sonuçlar

SMOTE ile dengelenmiş eğitim verisi üzerinde eğitilen ve tahmin eşiği optimize edilen ensemble modelimizin **test seti** üzerindeki performansı aşağıdaki gibidir:

*   **Accuracy:** \[%91.41]
*   **Precision:** \[%74.20]
*   **Recall:** \[%80.83]
*   **F1 Score:** \[%77.38]

**Sınıflandırma Raporu:**

Aşağıdaki tablo, modelimizin her sınıf (Depresyon=0 ve Depresyon=1) için precision, recall ve F1-score değerlerini özetlemektedir:

| Sınıf             | Precision | Recall  | F1-Score | Destek (Support) |
| :---------------- | :-------- | :-----  | :------- | :--------------- |
| **Depresyon=0**   | \[0.96]   | \[0.94] | \[0.95]  | \[23027]         |
| **Depresyon=1**   | \[0.74]   | \[0.81] | \[0.77]  | \[5113]          |
|                   |           |         |          |                  |
| _Accuracy_        |           |         | \[0.91]  | \[28140]         |
| _Macro Avg_       | \[0.85]   | \[0.87] | \[0.86]  | \[28140]         |
| _Weighted Avg_    | \[0.92]   | \[0.91] | \[0.92]  | \[28140]         |

**Karmaşıklık Matrisi:**

|                   | Tahmin: Depresyon Yok | Tahmin: Depresyon Var |
| :---------------- | :-------------------- | :-------------------- |
| **Gerçek: Depresyon Yok** | TN: \[21590]          | FP: \[1437]   |
| **Gerçek: Depresyon Var** | FN: \[980]            | TP: \[4133]   |

**Yorum:**
Modelimiz, özellikle sınıf dengesizliğiyle başa çıkma teknikleri (SMOTE ve model içi ağırlıklandırmalar) ve tahmin eşiği optimizasyonu sayesinde, depresyon vakalarını tespit etmede başarısını artırmıştır. Recall değerindeki belirgin artış (%80.8), modelimizin gerçek depresyon vakalarının büyük bir çoğunluğunu yakalayabildiğini göstermektedir. F1 skoru da (%77.4) modelin genel performansının dengeli olduğunu göstermektedir.

**Geliştiriciler:**
*   \[Berkay ÜZER]
*   \[Görkem YILDIZ]

## Kurulum ve Çalıştırma

```bash
# Gerekli kütüphaneleri yükleyin
pip install pandas numpy matplotlib seaborn scikit-learn xgboost catboost lightgbm optuna squarify imblearn missingno

# Notebook'u çalıştırın
jupyter notebook ENM421_Project.ipynb
