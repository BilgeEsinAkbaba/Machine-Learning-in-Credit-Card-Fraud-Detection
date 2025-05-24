# 💳 Credit Card Fraud Detection – Makine Öğrenmesi ile Dolandırıcılık Tespiti

Bu proje, **Akbank Makine Öğrenmesi Bootcamp** kapsamında gerçekleştirilmiştir. Projenin temel amacı, kredi kartı işlemleri arasındaki **dolandırıcılık vakalarını (fraud transactions)** tespit edebilen etkili ve güvenilir bir makine öğrenmesi modeli geliştirmektir. Gerçek hayatta kredi kartı dolandırıcılığı finansal kurumlar için ciddi zararlar doğurabileceğinden, bu tür işlemlerin hızlı ve doğru bir şekilde tespit edilmesi büyük önem taşımaktadır.

Veri seti, **Kaggle** üzerinden sağlanan ve Avrupa’daki anonimleştirilmiş bir banka tarafından sağlanmış **284,807 adet kredi kartı işlemini** içeren, etiketli bir veri setidir. İşlemlerin yalnızca 492’si dolandırıcılık olup, geri kalanı normal işlemdir. Bu durum, projeyi klasik sınıflandırma probleminden ayıran önemli bir zorluk barındırmaktadır: **şiddetli sınıf dengesizliği**.

---

## 📁 Proje İçeriği ve Adımları

### 1. Veri İncelemesi ve Keşifsel Veri Analizi (EDA)

Veri seti, her biri gizlenmiş (anonymized) olmak üzere 28 sayısal öznitelik (`V1` - `V28`), `Time` ve `Amount` adlı iki ek sütun ve hedef değişken olan `Class` sütunundan oluşmaktadır.

- `Class = 0`: normal işlem
- `Class = 1`: dolandırıcılık işlemi

Sınıflar arası büyük dengesizlik nedeniyle veri doğrudan makine öğrenmesi modellerine verilmeden önce, bu sorunu gidermek için **ön işleme** yapılması gerekmektedir.

---

### 2. Sınıf Dengesizliğinin Giderilmesi (Downsampling)

Veri setindeki `Class=1` yani dolandırıcılık vakalarının sayısı yalnızca 492'dir. Bu az sayıdaki örnek, modelin öğrenmesini zorlaştırır çünkü model çoğunlukla `Class=0`'ı görerek öğrenir. Bu problemi çözmek amacıyla:

- `Class=0` (normal işlemler) sınıfından rastgele 492 örnek seçilmiştir
- Bu örnekler, `Class=1` ile birleştirilerek dengelenmiş yeni bir veri seti oluşturulmuştur

Bu sayede model, her iki sınıfı da eşit şekilde öğrenme fırsatı bulur ve azınlık sınıfını ihmal etmeden genelleme yapabilir.

---

### 3. Veri Hazırlığı ve Ölçeklendirme

Model eğitimi için veriler:
- Özellikler (`X`) ve hedef (`y`) olmak üzere ayrılmıştır
- Eğitim ve test kümelerine **%80 - %20** oranında ayrılmıştır (stratified)
- `Amount` ve `Time` sütunları **StandardScaler** ile normalize edilmiştir

Veri hazırlık süreci, modelin öğrenme sürecini hızlandırmak ve daha sağlıklı sonuçlar almak adına oldukça kritik bir adımdır.

---

### 4. Modelleme ve Değerlendirme

#### Logistic Regression

İlk olarak, baseline performansı görmek adına **Logistic Regression** modeli eğitilmiştir. Oldukça basit ve yorumlanabilir bir model olmasına rağmen dengelenmiş veri sayesinde oldukça başarılı sonuçlar vermiştir.

#### Random Forest (Grid Search ile)

Ardından, daha kompleks bir algoritma olan **Random Forest** modeli, **GridSearchCV** kullanılarak hiperparametre optimizasyonu ile eğitilmiştir. Bu sayede modelin:
- Ağaç sayısı
- Derinlik sınırı
- Minimum örnek bölme sayısı
- Sınıf ağırlıkları gibi parametreleri optimize edilmiştir.

Her iki model için:
- Precision, Recall, F1-Score gibi sınıflandırma metrikleri
- ROC AUC skoru
- Karışıklık Matrisi (Confusion Matrix)
- Precision-Recall eğrisi grafik olarak sunulmuştur

---

### 5. Özellik Önem Dereceleri

Random Forest modeli aynı zamanda her bir özelliğin karar sürecine ne kadar katkı sağladığını da belirlemiştir. Bu analizde, özellikle `V14`, `V10`, `V12` gibi özniteliklerin dolandırıcılığı belirlemede çok daha etkili olduğu görülmüştür. Bu tür bilgi, hem modelin yorumlanabilirliğini artırmakta hem de ilerideki veri toplama stratejilerine yön vermektedir.

---

### 6. BONUS: DBSCAN ile Gözetimsiz Anomali Tespiti

Veri etiketlenmemiş olsaydı bu problemi nasıl çözebilirdik?

Bu sorunun cevabı için **DBSCAN** (Density-Based Spatial Clustering of Applications with Noise) algoritması kullanılarak **gözetimsiz (unsupervised)** bir anomali tespiti denemesi yapılmıştır. Yüksek boyutlu veri olduğu için önce **PCA** ile 2 boyuta indirgenmiş ve ardından kümeler tespit edilmiştir.

`Cluster = -1` olan örnekler, DBSCAN’e göre “anormal” olarak işaretlenmiştir. Sonuçlar çarpıcıdır:
- Toplam 270 anomali tespit edilmiştir
- Bunlardan **257’si gerçekten dolandırıcılık işlemi çıkmıştır** (%95 doğruluk)

---

## 📈 Sonuçlar

| Model             | Precision | Recall | F1-Score | ROC AUC |
|------------------|-----------|--------|----------|----------|
| Logistic Regression | 0.98 / 0.93 | 0.93 / 0.98 | 0.95 / 0.96 | 0.978 |
| Random Forest       | 0.96 / 0.93 | 0.93 / 0.96 | 0.94 / 0.95 | 0.981 |

> Her iki model de dengelenmiş veri sayesinde azınlık sınıfı olan dolandırıcılık vakalarını başarıyla öğrenmiştir.  
> Random Forest modeli daha yüksek ROC AUC skoru ile üstün bir performans göstermiştir.  
> DBSCAN ise etiketlenmemiş veriler üzerinde dahi yüksek doğrulukla anomali tespiti yapabileceğini göstermiştir.

---

## 🔬 Gerçek Hayatta Kullanım Senaryoları

- Finans kuruluşlarında **gerçek zamanlı dolandırıcılık tespiti**
- Etiketlenmemiş verilerde anomali tespiti için gözetimsiz öğrenme sistemleri
- Kullanıcı profillemesi ve davranış temelli alarm sistemleri
- Otomatik sahte işlem tespiti ve müşteri uyarı mekanizmaları

---

## 🚀 Geliştirme Önerileri

- SMOTE, ADASYN gibi farklı oversampling teknikleriyle performans artırımı
- XGBoost, LightGBM gibi gradient boosting modelleri ile karşılaştırmalı çalışmalar
- Modelin bulut tabanlı bir ortamda web arayüzü ile entegrasyonu (deployment)
- Stream veriler üzerinde gerçek zamanlı karar alma sistemleri kurulması

---
## 🧠 Proje Üzerine Sözel Değerlendirme

### 1. Hangi algoritmayı seçtiniz?

Bu projede hem **Logistic Regression** hem de **Random Forest** algoritmaları uygulanmıştır. Logistic Regression, temel bir doğrusal sınıflandırıcı olarak projeye referans (baseline) sağlamak amacıyla kullanılmıştır. Random Forest ise daha güçlü ve esnek bir algoritma olup, hiperparametre optimizasyonu ile nihai model olarak değerlendirilmiştir. Her iki model de başarılı sonuçlar vermiş olsa da, ROC AUC ve f1-score metrikleri açısından daha üstün performans sergileyen **Random Forest modeli** tercih edilmiştir.

### 2. Geliştirdiğiniz proje, veri seti üzerinde hangi problemi çözdü?

Bu proje, kredi kartı işlemleri içindeki **nadir görülen dolandırıcılık vakalarının doğru şekilde tespit edilmesi** problemini çözmeyi amaçlamaktadır. Veri setinde %0.17 gibi oldukça düşük oranla temsil edilen bu dolandırıcılık işlemlerinin, sınıf dengesizliği giderildikten sonra makine öğrenmesi modelleriyle tespit edilmesi hedeflenmiştir. Projede uygulanan dengeleme teknikleri ve sınıflandırma algoritmaları sayesinde, dolandırıcılık sınıfı %95'e varan başarı oranlarıyla tanınabilmiştir.

### 3. Bu proje gerçek hayatta nasıl işimize yarayabilir?

Gerçek dünyada bankalar ve finansal kuruluşlar her gün milyonlarca işlem gerçekleştirir. Bu işlemler arasında gerçekleşen dolandırıcılık vakalarının tespiti hem zaman alıcıdır hem de yüksek finansal zararlara yol açabilir.  
Bu proje sayesinde geliştirilen model:

- **Gerçek zamanlı** olarak sahte işlemleri tespit edebilir  
- İnsan müdahalesini en aza indirerek otomatik uyarı sistemleriyle entegre olabilir  
- Şirketlerin müşteri güvenliğini artırmasına ve zararı minimize etmesine olanak sağlar

Ayrıca, gözetimsiz öğrenme kullanılarak etiketlenmemiş yeni verilerde potansiyel dolandırıcılık işlemleri önceden tespit edilebilir.

### 4. Bu proje daha da geliştirilmek istense nasıl geliştirilebilir?

Bu projenin bir sonraki aşamada geliştirilebilmesi için aşağıdaki adımlar önerilmektedir:

- Daha büyük ve güncel veri setleriyle model yeniden eğitilerek genelleme gücü artırılabilir
- **SMOTE, ADASYN, BorderlineSMOTE** gibi gelişmiş yeniden örnekleme teknikleri denenebilir
- **XGBoost, LightGBM, CatBoost** gibi daha güçlü modellerle karşılaştırmalı analizler yapılabilir
- Model bir **web arayüzüne entegre edilerek** son kullanıcıya açık hale getirilebilir
- Stream data (akış verisi) üzerinden gerçek zamanlı tahminler yapılması için model optimize edilebilir

Bu geliştirmeler, modeli ticari kullanım için daha uygun hâle getirir ve finansal teknolojiler alanında gerçek uygulamalara dönüştürülebilir.

---
## 📂 Proje Linkleri

- 📓 Kaggle Notebook: https://www.kaggle.com/code/bilgeesinakbaba/machine-learning-in-credit-card-fraud-detection

---
