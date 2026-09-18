# Diabetes_130-US_Hospitals_for_Years_1999-2008


## 1. Problem Framing
* **Target Definition:** `readmitted` hədəf dəyişəni binar təsnifat (binary classification) şəkilinə salınmışdır:
* **1 (Pozitiv Sinif):** Xəstəxanadan çıxdıqdan sonra 30 gün daxilində təkrar daxil olanlar (`<30`).
* **0 (Neqativ Sinif):** 30 gündən sonra qayıdanlar (`>30`) və ya təkrar yatışı olmayanlar (`NO`).
* **Klinik və Biznes Əsaslandırma:** 30 günlük təzədən yatış müddəti klinik keyfiyyət standartı (CMS KPI) hesab olunur. Erkən qayıdışlar adətən müalicə planının natamamlığı, dərman rejiminə riayət edilməməsi və ya erkən evə buraxılma ilə bağlıdır.

---

## 2. Trustworthy Data Handling & Leakage Mitigation
* **Patient-Level Leakage:** Dataset-də 101,766 müraciət olsa da, unikal pasiyent sayı 71,518-dir. Eyni pasiyentin fərqli müraciətlərinin Train və Test setlərinə bölünərək modelin pasiyenti əzbərləməsinin qarşısını almaq üçün `patient_nbr` üzrə yalnız **ilk müraciət** (`keep='first'`) saxlanılmışdır.
* **Expired / Hospice Leakage (Vəfat və Hospis Sızması):** Xəstəxanada vəfat edən və ya hospisə köçürülən pasiyentlərin (`discharge_disposition_id` $\in [11, 13, 14, 19, 20, 21]$) 30 gün daxilində fiziki olaraq qayıtması mümkünsüzdür. Bu 2,423 sətir məlumat sızmasının və modeli aldatmasının qarşısını almaq üçün silinmişdir.
* **İdentifikatorlar və Lazımsız Sütunlar:** `encounter_id`, `patient_nbr` kimi verilənlər bazası indeksləri, 97% boş olan `weight` və sığorta kodu olan `payer_code` modelə daxil edilməmişdir.
* **Yüksək Kardinallıq (High-Cardinality ICD-9 Mapping):** `diag_1`, `diag_2`, və `diag_3` sütunlarındakı 700-dən çox ICD-9 xəstəlik kodu klinik standartlara uyğun 9 əsas anatomik/klinik qrupa (Circulatory, Respiratory, Digestive, Diabetes, Injury, Musculoskeletal, Genitourinary, Neoplasms, Other) xəritələnmişdir.

---

## 3. Class Imbalance Handling
* **Asimmetrik Sinif Paylanması:** Təmizlikdən sonra pozitiv sinif (`<30`) ümumi datanın təxminən ~8.98%-ni təşkil edir.
* **Metodoloji Həll:** Data sızmasının qarşısını almaq üçün **SMOTE** (Synthetic Minority Over-sampling Technique) strictly yalnız `train_test_split`-dən sonra **Train setinə** tətbiq edilmişdir. Alternativ olaraq XGBoost daxilində `scale_pos_weight` ($\approx 10.1$) parametri ilə cərimələmə balansı yaradılmışdır.

---

## 4. Baseline Comparison & Evaluation Decisions

* **Baza Model (Baseline):** Ən sadə referans nöqtəsi kimi `DummyClassifier(strategy='most_frequent')` və sadə `LogisticRegression` modeli qurulmuşdur.
* **Model Müqayisəsi:** Kompleks `XGBoost` modelinin baza modellərdən üstünlüyü ölçülmüşdür.
* **Metrikaların Seçilməsi:** Balanssız datada `Accuracy` yanıldıcı olduğu üçün qiymətləndirmə əsasən aşağıdakı metrikalara əsaslanır:
* **ROC-AUC & PR-AUC:** Modelin sinifləri ayırma qabiliyyətini ümumi qiymətləndirmək üçün.
* **Recall (Sensitivity):** Ötürülmüş hər bir erkən qayıdış xəstəxanaya əlavə xərc və risk yaratdığı üçün pozitiv sinfi tapmaq (False Negatives-i azaltmaq) əsas hədəfdir.

---

## 5. Model Interpretability & Critical Audit (SHAP Analysis)

* **Global Interpretability:** SHAP summary plot analizinə əsasən risk proqnozuna ən çox təsir edən faktolar:
1. `number_inpatient` (Əvvəlki xəstəxanaya yatış sayı)
2. `discharge_disposition_id` (Xəstəxanadan buraxılma forması)
3. `number_diagnoses` və `num_medications` (Polifarmasiya və ağır xəstəlik göstəricisi)


* **Local Interpretability:** Yüksək riskli və aşağı riskli fərdi pasiyentlər üçün SHAP Waterfall qrafikləri çıxarılaraq həkimlər üçün "bu xəstənin riski nə üçün yüksəkdir" sualı izah olunmuşdur.
* **Critical Insight & Trust Conclusion:** Modelin SHAP nəticələri təhlil edildikdə, alqoritmin ID və ya inzibati təsadüfi dəyişənlərə deyil, klinik baxımdan tamamilə əsaslı olan göstəricilərə (əvvəlki yatış tarixçəsi, qəbul edilən dərmanların sayı və diabet göstəriciləri) əsaslandığı sübut olunmuşdur. Model klinik istifadə üçün etibarlı (trustworthy) hesab edilə bilər.
