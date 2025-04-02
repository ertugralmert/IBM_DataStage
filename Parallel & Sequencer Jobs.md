# Parallel & Sequencer Jobs

# Parameter Setleri

Tekrar kullanılabilir değişkenleri gruplayıp saklamamızı sağlayan önemli bir özelliktir. 

1. Parametreleri Gruplama: İlişkili parametreleri örneğin veritabanı bilgileri, dosya yolları bir arada saklayabiliriz.
2. Tekrar Kullanılabilirlik: Bir kez oluşturup birden fazla işte kullanabiliriz.
3. Merkezi Yönetim: Değerleri tek bir yerden değiştirebiliriz.

Örneğin bir Oracle bağlantısı için parametre seti oluşturduğumuzda, içinde sunu, kullanıcı adı ve şifre gibi değerleri saklayabiliriz. Benzer şekildee, çevre değişkenleri için veya dosya yolları için ayrı parametre setleri oluşturabiliriz.

## Row Generator (Satır Üreteci)

Row Generator, test amaçlı sahte veri oluşturmak için kullanılan bir kaynaktır.

- Sadece Kaynak olarak kullanılır: Bir işin ilk aşaması olarak kullanılır, hedef olamaz
- Sütun Tanımlama: İstediğiniz veri yapısını tanımlayabilirsiniz. Sütun adları, veri tipleri, uzunluk
- Algortima Seçimi: Her sütun için veri üretme algoritması seçebilirsiniz.:
* Cycle: Belirli değerleri döngüsel olarak tekrarlar
* Random: Belirtilen aralıkta rastgele değerler üretir
* Numeric : Başlangıç değeri, artış mikkarı ve limit değer ile sayılar üretir
* Alphabet: Rastgele karakterler üretir.

Örneğin hesap numarası için 111'den başlayıp her seferinde 121 artarak giden sayısal değerler, hesap adı için rastgele karakterler, hesap türü iiçin "tasarruf","kredi kartı", "DMAT" gibi döngüsel değerler tanımlayabiiriz.

## Parametre Kullanımı

İşlerde sabit değerler yerine parametreler kullanmak büyük avataj sağlar:

- Esneklik: İşleri tekrar derlemeden parametreleri değiştirebiliriz.
- Çalışma Zamanı Yapılandırması: Örneğin kaç kayıt üretileceğini, çalışma zamanında değiştirebiliriz.

## Peak Stage

Verileri incelemek için kullanılan özel bir araçtır. 

- Dosya Oluşturmadan İnceleme: Veriler için ayrı dosya oluşturmadan, doğrudan günlükte görüntüler
- Kolay Kullanım: Herhangi bir yapılandırma gerektirmeden çalışabilir
- Seçici Görüntüleme: Belirli bölümlere(partitions) göre filtreleme yapılabilir, tüm sütunlar yerine sadece belirli sütunları görüntüleyebiliriz, kayıt saysını sınırlayabiliriz.

## Bölümlendirme - Partitioning

DS'de işler genellikle işleme için bölümlere ayrılır.

- Round Robin: Kayıtları sırayla bölümler arasında dağıtır. 1. kayıt → 1. bölüm → 2. kayıt → 2. bölüm gibi
- Hash: Belirli bir sütuna göre benzer değerleri aynı bölüme gönderir
- Auto: Sistem için en uygun bölümlemeyi seçer

Row Generator varsayılan olarak Round Robin bölümlendirme kullanır. bu nedenle 10 kayıt 2 bölüm arasında eşit dağılır. her bölüme 5 kayıt

Diğer Test Aşamaları:

- Column Generator: Mevcut verilere yeni sütunlar ekler.
- Head: Her bölümden belirli sayıda ilk kaydı gösterir
- Tail: Her bölümden belirli sayıda son kaydı gösterir
- Sample: erileri belirli oranlarda örnekleme. yapar.

Bu araçlar, gerçek veri olmadan veya hassas verileri kullanmadan iş tasarımı ve test süreçlerini kolaylaştırır. bu yaklaşımla, developer veya prod ortamlarda güvenli ve verimli bir şekilde jobları test edebilir ve hata ayıklayabilirsiniz.

# Database İşleri ve Bağlantılar

## Oracle Veritabanı bağlantısı kurma

Connector ekleriz, sunucu, kullanıcı adı ve şifre bilgilerini parametre seti üzerinden bağlarız. Bağlantı test edilir.

Bir veritabanı kaynağı ile çalışırken sütun bilgilerini elle girmek yerine otomatik olarak import etmenin daha iyi olduğunu söyleyebiliriz.

Import → Table definitions → Oracle veya DB2 → Bağlantı bilgileri → Şema veya tablo filtreleme seç → DataStage aktar

Veritabanından veri çekerken 

1. Manual sorgu yazma: SELECT * FROM training.ablum gibi sorguları doğrudan yazabiliriz
2. Otomatik Sorgu oluşturma: "Generate SQL at run time" seçeneği ile DataStage'in sorguyu otomatik oluşturmasını sağlayabiliriz. sadece tablo adı belirtmemiz yeterli 

## Runtime Column Propagation - RCP

Sorgu sonucunda gelen tüm sütunların, biz özellikle tanımlamasak bile otomatik olarak akışa dahil edilmesini sağlar.

RCP etkinleştirildiğinde, tabloda olmaya sütunları tanımlamak zorunda kalmayız

RCP ile sorgu kaç sütun döndürürse döndürsün iş başarıyla çalışır

## Veritabanı Yazma İşlemleri

Farklı modlar vardır.

1. insert: yeni kayıtlar ekler
2. update: mevcut kayıtları günceller
3. Insert then Update : Önce eklemeyi dener sonra günceller
4. Update then Insert: Önce güncellemeyi dener, sonra ekler
5. Delete then Insert: Önce mevcut kayıtları siler, sonra yenilerini ekler

### Performans Ayarları

Veritransferinde performansı etkileyen iki önemli ayar:

1. Record Count: Bir seferde kaç kayıt çekileceğini belirler örn: 2000
2. Array size : Verilerin tutulacağı bellek alanını belirler: örn 2000 blok

Eğer kayıtlar büyükse (çok sütunlu) aynı blok sayısına daha az kayıt sığacağından performans etkilenebilir.

### Genel İş Tasarımı

Parametre olarak "scheme" ve "table_name" değişkenleri var

Oracle veya DB2 bağlayıcısında ve hedef dosya adında parametreler kullanılır. Böylece aynı iş farklı tablolardan veri çekmek için tek bir tasarımla kullanılır.

Sadece çalışma zamanında parametre değerlerini değiştirmek yeterli olur.

---

# Sequencer İş ve Döngü Kullanımı

Yukarıda parametre değiştirerek veriçekme işlemi yapıyorduk ancak gerçek projelerde 10 veya 100 tabloyu tek tek elle çalıştırmak verimsiz olur.

Çözüm olarak bir "Sequencer" iş kullanarak otomatikleştirme yaklaşımı yapabiliriz. Bu aynı işi farklı tablo isimleri için otomatik olarak çalıştırabilmeyi sağlar.

Sequencer İş Oluşturma :

Yeni bir "Sequence Job" oluşturalım  ve "Job Activity" bileşeni ekleyelim. doğrudan veya sürükleyip bırakabilirz. 

Execution Action - Çalışma Eylemi

Sequencer içinde bir job çalıştırmak için 4 seçeneceğimiz var.

Run: Sadece çalıştır

Reset if required, then run: Gerekirse sıfırla sonra çalıştır (önerilen)

Validate only: sadece doğrula, çalıştırma

Reset Only: Sadece sıfırla

İşin herhangi bir neden ötürü iptal (abort) edilmesi durumunda "Reset if required, then run" seçeneği otomatik olarak işi sıfırlayıp yeniden çalıştıracağın en iyi seçenek olarak düşünebiliriz.

### Multiple Instance Desteği - çoklu örnek

İşleri daha iyi takip edebilmek için " allow multiple instances" seçeneğini etkinleştiririz. Bu sayede

her tablo için işin ayı bir örneği oluşturulur örn J_DB_9.album, J_DB_9.artist gibi

Hata ayıklama ve takio daha kolay hale gelir

Hangi tablonun başarıyla çalıştığı, hangisinin hata verdiği daha net görülebiliyor.

### Loop Kullanımı

Tabloların tümü için otomatik çalıştırma amacıyla döngü ekleriz

"Start Loop Activity" ve "End Loop Activity" bileşenleri eklenir. Bunları birbirine bağlarız

İki tür loop seçeneği vardır.

1. Numeric Loop: Sayılsal döngü - belirli sayıda tekrar için
2. List Loop: Liste döngüsü - bir liste içindeki her öğre için tekrar

Veritabanı tablolarını otomatik işlemek için "List Loop" kullanırız. Virgülle ayrılmış tablo isimleri loop değeri olarak ayarlanabilir. album,artist,customer,employee gibi

### Loop Değerinin Parametreleştirilmesi

Farklı tablo listeleri için işi yeniden derlemeden çalıştıraibliriz.

Çalışma zamanında hangi tabloların işleneceğini belirtebiliriz

Tam anlamıyla jenerik bir job elde etmiş oluruz.

# Farklı Veritabanı Bağlayıcıları

Oracle,DB2,Teradata, JDBC, ODBC benzer yapıdadır. Bir tanesini öğrenmemiz diğerlerini de kolayca bağlamamıza yarar. 

Bağlantı ayaları benzer şekilde ayarlanır.

Generate SQL seçenekleri aynı şekilde çalışır

Write Mode seçenekleri - Insert, Update, Delete, vb. aynıdır

Performans parametreleri benzerdir. record count, array size gibi

Bu sayede migration projeleri gibi durumlarda tek bir jenerik job tasarlayarak onlarca veya yüzlerce tabloyu otomatik olarak işleyebiliriz.

---

# Sequential File İşlemleri

Sıralı Dosya aşaması olarak geçer. En temel ve önemli bileşenlerden biridir. Hem kaynak hem de hedef olarak kullanılabilir. Bu aşamada CSV veya TXT gibi yapılandırılmış düz metin dosyalarını okuyabilir ve yazabiliriz.

## Sequential File Stage'in Temel Özellikleri

**Kaynak olarak Sequential File Stage:** 

**Dosya yolu ve ismi** : dosyanın tam yolunu ve ismini belirtmeniz gerekir.

**Sütun bilgisi: Runtime**  Column Propagation yerine sütun metadata'sını belirtmemiz önerilir.

**First Line is Column Name**: Eğer "true" seçilirse, ilk satır kolon başlığı olarak kabul edilir ve veri olarak okunmaz 

**Okuma Yöntemleri:**

**Specific File** : Tek bir dosyayı okumak için kullanılır.

**File Pattern ( Dosya Deseni):** Wild karakterler örn: *.csv kullanarak birden fazla dosyayı okumak için.

Hedef Olarak Sequential File Stage:

**File Update Mode:**
      Append: Mevcut dosyanın sonuna veri ekler
     Override: Mevcut dosyayı siler ve yeniden oluşturur
      Create: Dosya yoksa oluşturur , varsa hata verir
**Generate Multiple Files:**
     Key Column: Belirtilen kolon değerine göre farklı dosyalar oluşturur. örneğin tarih kolonuna                  göre günlük dosyalar.

     Partitioning: Job konfigürasyonundaki bölümleme sayısına göre ayrı dosyalar oluşturur.

Önemli Parametreler:

Reject Mode: Veri tipi uyuşmazlığı veya diğer hatalar olduğunda davranışı belirler.

Continue: Hatalı kayıtları atlayarak diğer kayıtlarla devam eder.

Fail: Hata olduğunda job'ı durdurur

Output: Hatalı kayıtları ayrı bir output link'e yönlendirir (reject link)

Missing File Mode: Kaynak dosya bulunamadığında davranışı belirler.

Error: Hata verir ve job u durdurur

OK: Dosya olmasa bile job devam eder 0 kayıt.

Depends: Job yapısına göre karar verir. (Varsayılan seçenek)

Filter: UNIX komutlarını kullnarak kaynak dosyadan veri filtrelemenize olanak tanır.

head -5: İlk 5 kaydı alır.

grep ABC: "ABC" içeren kayıtları alır.

Row Number ve File Name: 

Row Number Column : her kayda benzersiz bir numara atar

File Name Column: Her kaydın hangi dosyadan geldiğini gösteren bir sütun ekler.

Reject Reason Column: Reddetme nedeni için bir sütun ekler

Performans İyileştirme Seçenekleri:

Number of Reader per Node: Her Node üzerinde birden fazla okuyucu oluşturur.

Read from Multiple Node: Aynı dosyayı birden fazla node dan okur.

Read First Rows: Belirtilen sayıda kaydı okur. Filter'dan farklı olarak reddedilen kayıtları da sayar.

 Cleanup on Failure: Job başarısız olduğunda kısmen oluşturulmuş çıktı dosyalarını temizler. Her zaman "true" olarak ayarlanmalıdır.

Örnekler:

        1. Veri tipi uyuşmazlığı: Double veri tipine bir metin dğer gelirse:
Continue: Kaydı atlar ve diğerleriyle devam eder
Fail: Job'u durdurur
Output: Hatalı kaydı reject link'e yönlendirir
        2. Multiple Files:
Key column temelli: Her benzersiz değer için ayı bir dosya örn: tarih veya müşteri ID
Partition Temelli: Job'ın çalıştığı node sayısını göre dosyalar. part0, part1 ...

# DataStage'de Data Set ve File Set İşlemleri

DataStage'de Sequential File Stage dışında iki önemli dosya işleme aşaması daha vardır. Data Set ve File Set. Bu iki dosya türü, DataStage'in kendi özel formatları olup, sadece DataStage içinde oluşturulabilir ve okunabilir. Dışarıdaki programlar tarafından oluşturulamaz veya okunamazlar.

### Data Set ve File Set'in Temel Özellikleri

1. Dosya Yapısı ve Uzantıları
* Data Set: .DS uzantılı dosyalar
* File Set: .FS uzantılı dosyalar
2. Dosya Tipleri
Her iki format da iki farklı dosya oluşturur.
* Descriptor File (Tanımlayıcı Dosya): Şema bilgisi ve veri dosyasının yolunu içerir
* Data File : Gerçek verileri içerir ve farklı bir konumda saklanır.
3. Şifreleme ve Güvenlik
* Data Set: Hem tanımlayıcı dosya hem de veri dosyası şifrelidir. Standart metin editörleriyle okunamazlar
* File Set: Şifrelenmez, düz metin editörleriyle okunabilir.
4. Dosya Yönetimi
* Data Set : Manual olarak silinemez, sadece DataStage'deki "Dat Set Management Utility" ile yönetilebilir.
* File Set: Manual olarak silinebilir ve düzenlenebilir. ancak dikkatli yapmak gereklidir.

## Data Set ve File Set'in Kullanım Amaçları

1. Karmaşık İş Akışlarını Bölme: Büyük ve karmaşık DataStage işlerini daha küçük ve yönetilebilir parçalara bölmek için kullanılır.
2. Performans İyileştirme: DataStage işlerninin performansını artırmak için, çok fazla sayıda aşama yerine ara aşamalarda data set/file set kullanmak daha çok verimlidir.
3. Bölümleme(Partitioning Bilgisini Koruma: Sequntial File'dan farklı olarak bölümleme bilgilerini korurlar. Bir job'un oluşturduğu bölümleme yapısı, diğer joblarda da korunur.
4. Güvenlik: Özellikle Data Set ler şifreli olduundan, hassas verileri işlendiği bankacılık gibi alanlarda tercih edilir.

## Data Set/File Set Yapılandırma Seçenekleri

**Hedef olarak kullanırken**

- Update Policy
**Append:** Var olan kayıtlara ekler
**Create:** Yeni dosya oluşturur
**Override:** Var olan dosyayı tamamen değiştirir
**Use existing discard records**: Tanımlayıcı dosyayı değiştirmeden, sadece veri dosyasını günceller
**Use existing discard records and schema:** Hem tanımlayıcı dosyayı hem ve veri dosyasını günceller
- Partitioning (Bölümleme): Verilerin nasıl bölümleneeceğini belirler özel bir kolon seçilebilir

Kaynak Olarak Kullanırken

- Missing Column Mode: Uyumsuz sütun sayısı durumunda davranışı belirler
- Runtime Column Propagation: Etkinleştirilirse, tanımlayıcı dosyadaki şema bilgisinden sütunları otomatik alır.

File Set'in Özel Özellikleri

- Her 2 GB'da otomatik olarak yeni veri dosyası oluşturur. Büyük veri setleri için avantaj
- Data Set'ler ise tek bir büyük veri dosyası oluşturur
- Büyük miktarda veri işlenirken Filet Set biraz daha iyi performans gösterebilir.

Data Set Nasıl Yönetilir

- Data Set Management Utility : Data Set dosyalarını görünlemek, değiştirmek veya silmek için kullanılır.
- Bu araç ile
Dosyanın kaç node'da oluşturuulduğunu görebiliriz
Veri içeriğini görüntüleyebiliriz
Şema bilgilerini inceleyebiliriz
Data Set'i güvenli bir şekilde silebiliriz

Data Set/File Set seçimi 

Seçim yaparken dikkat edilmesi gereken farktörler:

1. Güvenlik ihtiyacı: Yüksek güvenlik gerektirren projeler için Data Set daha uygundur
2. Veri büyüklüğü: Çok büyük veri setleri için File Set daha iyi performans gösterir
3. Verilerin dış paylaşımı: Eğer verilerin iş birimleriyle paylaışması gerekiyorsa, Sequential File tercih edilmelidir.

Data Set ve File Set, DataStage'in güçlü ancak dikkatli kullanılması gereken özelliklerindendir. Dışarıdan erişilemez olmaları, bölümleme bilgisini korumaları ve performans avantajları ile karmaşık ETL işlerinde büyük değer sağlarlar. Ancak Data Set'lerin manuel silinememesi ve dış sistemlerle paylaşılamaması gibi kısıtlamaları da göz önünde bulundurulmalıdır.

# DataStage Copy Stage

Copy Stage (Kopyalama Aşaması), DataStage'in Processing (İşleme) kategorisindeki en basit ama en kullanışlı aşamalarından biridir. Bu aşamanın temel amacı, kaynak verilerini birden fazla çıkış akışına kopyalamaktır.

## Copy Stage'in Temel Özellikleri ve Kullanım Amaçları

### 1. Tek Kaynaktan Çoklu Hedeflere Veri Dağıtımı

- Aynı veri setini birden fazla hedef aşamaya tek bir okuma işlemiyle göndermek için kullanılır
- Kaynak verileri tekrar tekrar okumak yerine, bir kez okuyup birden çok işlem için kullanma imkanı sağlar

### 2. Sütun Seçme ve Yeniden İsimlendirme

- Her çıkış bağlantısı için farklı sütun setleri seçilebilir
- Sütunlar yeniden isimlendirilebilir (örnek: "title" sütunu "title_1" olarak yeniden adlandırılabilir)
- İstenmeyen sütunlar çıkartılabilir, böylece her çıkış bağlantısı sadece gerekli sütunları alır

### 3. Performans Avantajı

- Aynı kaynağı birden fazla kez okumak yerine, tek okuma işlemiyle veriyi farklı işlem akışlarına dağıtır
- Özellikle büyük veri setleriyle çalışırken performans iyileştirmesi sağlar

## Copy Stage Yapılandırması

### Runtime Column Propagation (RCP) Etkinken

- RCP etkinleştirildiğinde, Copy Stage'de herhangi bir özel sütun yapılandırması yapmaya gerek kalmaz
- Giriş akışındaki tüm sütunlar otomatik olarak çıkış akışlarına aktarılır

### Runtime Column Propagation (RCP) Etkin Değilken

- RCP devre dışı bırakıldığında, her çıkış akışı için hangi sütunların iletileceğini manuel olarak belirtmek gerekir
- Sütunlar "Output" sekmesinden doğru çıkış bağlantılarına sürüklenip bırakılarak atanır
- Bu yöntem, farklı çıkış bağlantıları için farklı sütun setleri seçmeye olanak tanır

## Pratik Kullanım Örnekleri

1. **Aynı Veri Üzerinde Farklı Hesaplamalar**:
    - Bir veri seti üzerinde hem toplam hesaplama (sum) hem de sayım (count) gibi farklı agregasyon işlemleri yapılması gerektiğinde
1. **Farklı İşlem Akışları**:
    - Aynı verinin hem bir join (birleştirme) işlemi hem de bir filtreleme işlemi için kullanılması gerektiğinde
1. **Veri Kalitesi Kontrolü**:
    - Bir kopya veriyi orijinal formatta iletirken, diğer kopyayı veri kalitesi kontrolü için kullanmak

## Özet

Copy Stage, tek bir kaynak akışından birden fazla çıkış akışı oluşturma ihtiyacı olduğunda kullanılan basit ama güçlü bir aşamadır. Veriyi tekrar tekrar okumak yerine, tek okuma işlemiyle çoklu hedeflere dağıtarak performans iyileştirmesi sağlar. Çıkış akışlarındaki sütunları seçme ve yeniden isimlendirme esnekliği sunar. Özellikle karmaşık ETL işlerinde, aynı veri üzerinde farklı işlem akışları gerçekleştirilmesi gerektiğinde çok kullanışlıdır.



# DataStage'de Agregasyon, Join ve Lookup İşlemleri

Aggregator (Toplayıcı), Join (Birleştirme) ve Lookup (Arama) aşamaları.

## 1. Aggregator (Toplayıcı) Stage

Aggregator Stage, verileri belirli bir sütuna göre gruplamak ve özet bilgiler (toplam, sayım, ortalama vb.) elde etmek için kullanılır.

### Temel Özellikler:

- **Grup Anahtarı (Group Key)**: Verilerin gruplandırılacağı sütun
- **Null İşleme**: Null değerlerin nasıl ele alınacağı
- **Toplama İşlemleri**:
    - **Count Rows**: Kayıt sayısını hesaplar
    - **Calculation**: Toplam, ortalama, minimum, maksimum gibi hesaplamalar yapar
    - **Recalculation**: Önceki hesaplamaları tekrar işler

### Örnek Kullanım:

- Web sayfası ziyaretlerini şehir bazında gruplandırarak her şehirden kaç ziyaret olduğunu hesaplama
- Bölge bazında maksimum ziyaret zamanını bulma
- Ürün kategorilerine göre toplam satış tutarlarını hesaplama

## 2. Join (Birleştirme) Stage

Join Stage, iki veya daha fazla veri akışını ortak bir sütun üzerinden birleştirmek için kullanılır.

### Temel Özellikler:

- **Birleştirme Türleri**:
    - **Inner Join**: Yalnızca her iki kaynakta da eşleşen kayıtları getirir
    - **Left Outer Join**: Sol kaynaktaki tüm kayıtları ve sağ kaynaktaki eşleşenleri getirir
    - **Right Outer Join**: Sağ kaynaktaki tüm kayıtları ve sol kaynaktaki eşleşenleri getirir
    - **Full Outer Join**: Her iki kaynaktaki tüm kayıtları getirir
- **Link Ordering**: Hangi bağlantının sol, hangi bağlantının sağ olduğunu belirler

### Sınırlamalar:

- Birden fazla birleştirme yapmak için çoklu Join aşamaları kullanmanız gerekir
- Her bir Join Stage sadece iki bağlantı arasında birleştirme yapabilir

## 3. Lookup (Arama) Stage

Lookup Stage, bir ana veri akışı ile bir veya daha fazla referans veri akışını birleştirmek için kullanılır. Join aşamasına benzer, ancak daha esnek ve güçlüdür.

### Temel Özellikler:

- **Ana Bağlantı**: Ana veri akışı (düz çizgi ile gösterilir)
- **Referans Bağlantıları**: Referans veri akışları (kesik çizgilerle gösterilir)
- **Çoklu Arama**: Tek bir aşamada birden fazla referans ile eşleştirme yapabilme
- **Lookup Failure Options**:
    - **Continue**: Eşleşmeyen kayıtlar için null değerlerle devam eder (Left Outer Join gibi)
    - **Drop**: Eşleşmeyen kayıtları atar (Inner Join gibi)
    - **Fail**: Eşleşmeyen kayıt bulunursa işi durdurur
    - **Reject**: Eşleşmeyen kayıtları ayrı bir reject bağlantısına yönlendirir

### Önemli Noktalar:

- Ana bağlantıdaki tüm sütunlar çıkışa aktarılmalıdır
- Referans bağlantılarından sadece ek sütunlar çıkışa aktarılmalıdır
- Reject seçeneği, yeni kayıtları (insert) ve güncellenen kayıtları (update) ayırmak için kullanılabilir

## Örnek Senaryo: Web Sayfası Ziyaretlerini Analiz Etme

Senaryoda kullanılan veri, web sayfası ziyaretleri hakkında bilgiler içerir:

- Event (Olay): Sayfa görüntüleme, bilgi isteme
- Timestamp (Zaman Damgası): Tıklama zamanı
- Distinct ID: Her tıklama için benzersiz kimlik
- City (Şehir): Tıklamanın geldiği şehir
- OS (İşletim Sistemi): Kullanıcının işletim sistemi
- Region (Bölge): Tıklamanın geldiği bölge

Bu veriyi kullanarak:

1. Her şehirden kaç ziyaretçi geldiğini hesapladık
2. Her bölgeden kaç ziyaretçi geldiğini hesapladık
3. Copy Stage kullanarak tek veri kaynağını iki farklı Aggregator aşamasına yönlendirdik
4. Join ve Lookup aşamalarını kullanarak bu bilgileri orijinal verilerle birleştirdik
5. Lookup'ın farklı hata seçeneklerini (fail, drop, continue, reject) test ettik

## Pratik Tavsiyeler

1. **Agregasyon Yaparken**:
    - Group Key'in dikkatli seçilmesi veri kalitesini etkiler
    - Tüm sütunlar değil, sadece grup anahtarı ve hesaplanan değerler çıktıya geçer
1. **Join ve Lookup Kullanırken**:
    - Birden fazla birleştirme gerekiyorsa Lookup tercih edilmelidir
    - Ana veri akışından gelen sütunların eşlenmesine dikkat edilmelidir
    - Referans bağlantılarından sadece ek bilgileri eşleyin
1. **Veri Filtreleme**:
    - Filter Stage kullanarak verileri filtreleyip farklı davranışları test edebilirsiniz
    - Lookup Reject seçeneği, insert ve update kayıtlarını ayırmak için kullanılabilir

Bu işleme aşamaları, DataStage'de karmaşık veri dönüşümleri ve analizleri gerçekleştirmenin temel yapı taşlarıdır. İyi anlaşıldığında, veri ambarı ve ETL süreçlerinde çok güçlü çözümler sağlarlar.



# DataStage Merge Stage

Merge Stage (Birleştirme Aşaması), DataStage'in Join ve Lookup aşamalarının özelliklerini birleştiren, daha esnek bir veri birleştirme aracıdır. Bu aşama, verilerin nasıl eşleştiğini ve eşleşmediğini izlemeniz gereken karmaşık durumlarda kullanılır.

## Merge Stage'in Temel Özellikleri

### 1. Bağlantı Yapısı

- **Master Link (Ana Bağlantı)**: Ana veri akışı (düz çizgi ile gösterilir)
- **Update Links (Güncelleme Bağlantıları)**: Referans veri akışları (kesik çizgilerle gösterilir)
- **Reject Links (Reddetme Bağlantıları)**: Eşleşmeyen kayıtlar için çıkış bağlantıları

### 2. Bağlantı Sayısı

- **Tek Master Link**: Sadece bir ana veri kaynağı olabilir
- **Çoklu Update Links**: Birden fazla referans kaynağı olabilir
- **Çoklu Reject Links**: Her Update Link için bir Reject Link olabilir

Bu, Merge Stage'i hem Join hem de Lookup'tan farklı kılar:

- **Join**: Sadece iki bağlantı, reject link yok
- **Lookup**: Çoklu referans bağlantısı ancak sadece bir reject link
- **Merge**: Çoklu referans ve her biri için ayrı reject link

### 3. Anahtar Sütun (Key Column) Gereksinimleri

- Tüm bağlantılardaki anahtar sütunlar aynı olmalıdır
- Lookup'tan farklı olarak, farklı isimli sütunları eşleştiremezsiniz

## Merge Stage'in Önemli Özellikleri

### 1. Unmatched Masters Mode (Eşleşmeyen Ana Kayıtlar Modu)

- **Keep**: Ana bağlantıdan gelen ancak referans bağlantılarında eşleşmeyen kayıtlar korunur
- **Drop**: Ana bağlantıdan gelen ancak referans bağlantılarında eşleşmeyen kayıtlar atılır

### 2. Warn on Reject Updates

- True/False: Referans bağlantılarından gelen ve ana bağlantıda eşleşmeyen kayıtlar için uyarı verilip verilmeyeceği

### 3. Warn on Unmatched Masters

- True/False: Ana bağlantıdan gelen ve referans bağlantılarında eşleşmeyen kayıtlar için uyarı verilip verilmeyeceği

### 4. Link Ordering (Bağlantı Sıralaması)

- Hangi bağlantının master (ana) olduğunu belirler
- Hangi bağlantıların update (güncelleme) olduğunu belirler
- Her update bağlantısına karşılık gelen reject bağlantısını belirler

## Merge Stage'in Kullanım Senaryoları

1. **Kapsamlı Veri İzleme**:
    - Eşleşen ve eşleşmeyen tüm kayıtları detaylı olarak izlemek istediğinizde
    - Her referans kaynağından gelen eşleşmeyen kayıtları ayrı ayrı analiz etmek istediğinizde
1. **Karmaşık Veri Entegrasyonu**:
    - Birden fazla kaynaktaki verileri birleştirirken, her kaynaktan gelen eksik verileri ayrı ayrı yönetmek istediğinizde
1. **Veri Kalitesi Analizi**:
    - Birden fazla referans kaynağı ile ana kaynağı karşılaştırıp, hangi kaynaklarda eşleşmeyen veriler olduğunu tespit etmek istediğinizde

## Merge vs Lookup vs Join Karşılaştırması

| **Özellik**           | **Merge**           | **Lookup**                | **Join**                  |
| --------------------- | ------------------- | ------------------------- | ------------------------- |
| Ana Bağlantı          | 1                   | 1                         | 2 (sol/sağ)               |
| Referans Bağlantıları | Çoklu               | Çoklu                     | Yok                       |
| Reject Bağlantıları   | Her referans için 1 | Toplam 1                  | Yok                       |
| Anahtar Sütun         | Aynı isimli olmalı  | Farklı isimli olabilir    | Aynı veya farklı olabilir |
| Eşleşmeyen Kayıtlar   | Seçeneğe bağlı      | Continue/Drop/Fail/Reject | Join türüne bağlı         |

## Pratik Kullanım Tavsiyeleri

1. **Bağlantı Sayısı**: Referans bağlantı sayısı ve reject bağlantı sayısı eşit olmalıdır
2. **Çıkış Sütunları**: Ana bağlantı ve referans bağlantılarından gelen sütunları doğru şekilde eşleştirmeye dikkat edin
3. **Performans**: Çok sayıda bağlantı kullanıldığında, Merge Stage'in performansı diğer aşamalardan düşük olabilir
4. **Kullanım Durumu**: Tüm eşleşmeyen kayıtları ayrıntılı olarak izlemeniz gerektiğinde kullanın, standart birleştirme işlemleri için Join veya Lookup genellikle yeterlidir

Merge Stage, DataStage'in en esnek veri birleştirme aşamalarından biri olmasına rağmen, karmaşıklığı nedeniyle yaygın olarak kullanılmaz. Ancak, kapsamlı veri kalitesi kontrolü ve karmaşık veri entegrasyonu gerektiren durumlarda çok değerli bir araçtır.

# DataStage'de Filter, Funnel ve Sort İşlemleri

 Filter (Filtre), Funnel (Huni) ve Sort (Sıralama) aşamaları. Bu aşamalar, ETL süreçlerinde veri manipülasyonu için temel yapı taşlarıdır.

## 1. Filter Stage (Filtre Aşaması)

Filter Stage, gelen verileri belirli koşullara göre farklı çıkış akışlarına yönlendirmek için kullanılır.

### Temel Özellikler:

- **Çoklu Çıkış Bağlantıları**: Farklı koşulları karşılayan verileri farklı çıkışlara yönlendirebilirsiniz
- **Where Clause (Koşul İfadesi)**: Her çıkış için belirli bir filtre koşulu tanımlayabilirsiniz (örn. `city = 'Mumbai'`)
- **Output Link (Çıkış Bağlantı Numarası)**: Her koşulun hangi çıkış bağlantısına yönlendirileceğini belirtir (0, 1, 2, vb.)

### Önemli Seçenekler:

- **Output Reject (Reddetme Çıkışı)**: `true` olarak ayarlandığında, hiçbir koşula uymayan kayıtlar reject bağlantısına yönlendirilir
- **Output Row Only Once (Satırı Yalnızca Bir Kez Çıkart)**:
    - `false`: Bir kayıt birden fazla koşula uyuyorsa, tüm ilgili çıkışlara gönderilir (çoğaltma)
    - `true`: Bir kayıt yalnızca karşılaştığı ilk koşulun çıkışına gönderilir (koşul sırası önemlidir)

## 2. Funnel Stage (Huni Aşaması)

Funnel Stage, birden çok girişten gelen verileri tek bir çıkışta birleştirmek için kullanılır (SQL'deki UNION operasyonu gibi).

### Temel Özellikler:

- **Çoklu Giriş Bağlantıları**: Birden fazla veri kaynağını tek bir akışta birleştirir
- **Sütun Uyumluluğu**: Tüm giriş bağlantılarındaki sütun adları ve veri tipleri uyumlu olmalıdır

### Toplama Algoritmaları:

- **Continuous Funnel (Sürekli Huni)** (Varsayılan): En verimli seçenektir, kayıtlar geldiği sırayla işlenir
- **Sequence Funnel (Sıralı Huni)**: Her bağlantıdan gelen tüm kayıtları sırayla işler (bağlantı 1'in tüm kayıtları → bağlantı 2'nin tüm kayıtları)
- **Sort Funnel (Sıralama Hunisi)**: Sıralanmış giriş akışlarının sıralama düzenini korur

### Önemli Noktalar:

- **Continuous Funnel**: En yüksek performansı sunar, ancak sıralamayı korumaz
- **Sequence Funnel**: Pipeline işleme yapısını bozar, genellikle önerilmez
- **Sort Funnel**: Önceden sıralanmış verilerin sıralama düzenini korumak için kullanılır

## 3. Sort Stage (Sıralama Aşaması)

Sort Stage, verileri belirli sütunlara göre sıralamak için kullanılır.

### Temel Özellikler:

- **Sort Keys (Sıralama Anahtarları)**: Verilerin sıralanacağı bir veya daha fazla sütun
- **Sort Order (Sıralama Düzeni)**: Artan (ascending) veya azalan (descending) düzen
- **Allow Duplicates (Çoğaltmalara İzin Ver)**: Çoğaltmaları korur veya kaldırır

### Önemli Seçenekler:

- **Key Change Column (Anahtar Değişim Sütunu)**: Benzersiz kayıtları (1) ve çoğaltmaları (0) belirleyen yeni bir sütun ekler
- **Cluster Key Change Column**: Key Change Column'a benzer işlev görür
- **Sort Utility (Sıralama Yardımcı Programı)**:
    - **DataStage**: Tüm ortamlarda çalışır (önerilen)
    - **Unix**: Yalnızca Unix tabanlı sistemlerde çalışır
- **Link Sort (Bağlantı Sıralama)**: Herhangi bir aşamada sütunları sıralamak için kullanılabilir

### Sıralama Modları:

- **Sort**: Varsayılan seçenek, gerektiğinde verileri sıralar
- **Don't sort previously sorted**: Önceden sıralanmış veriler yeniden sıralanmaz
- **Don't sort previously grouped**: Önceden gruplanmış veriler yeniden sıralanmaz

## Duplicates (Çoğaltmalar) İle Çalışma

Çoğaltmaları belirleme ve işleme için iki temel yöntem vardır:

### 1. Sort Stage ile Çoğaltmaları Belirleme:

- **Key Change Column** kullanarak benzersiz (1) ve çoğaltma (0) kayıtları işaretleme
- **Filter Stage** kullanarak bu işaretli kayıtları farklı çıkışlara ayırma

### 2. Remove Duplicates Stage:

- Belirli sütunlara göre çoğaltmaları kaldırır
- **Duplicates Routine (Çoğaltma Rutini)** seçenekleri:
    - **First**: İlk karşılaşılan kaydı korur (yüksek performanslı)
    - **Last**: Son karşılaşılan kaydı korur

### Örnek Senaryo: En Yüksek Zaman Damgalı Kayıtları Alma

1. **Sort Stage** kullanarak önce şehre göre, sonra zaman damgasına göre (azalan) sıralama
2. **Remove Duplicates Stage** ile şehre göre çoğaltmaları kaldırma ve "First" rutinini seçme

## Önemli İpuçları

1. **Filter Stage İçin**:
    - Koşullarınız örtüşüyorsa "Output Row Only Once" seçeneğine dikkat edin
    - Reject link ile filtrelenmeyen kayıtları da yakalayın
1. **Funnel Stage İçin**:
    - Tüm giriş kaynakları aynı sütun adlarına ve veri tiplerine sahip olmalıdır
    - Performans kritikse Continuous Funnel kullanın
    - Sıralamanın korunması gerekiyorsa Sort Funnel kullanın
1. **Sort Stage İçin**:
    - Sıralama işlemleri kaynak yoğun olabilir, yalnızca gerektiğinde kullanın
    - Link Sort seçeneğiyle herhangi bir aşamada sıralama yapabilirsiniz
    - Çoğaltmaları belirlemek için Key Change Column kullanışlıdır

Bu aşamalar, DataStage'de veri hazırlama, doğrulama ve dönüştürme işlemlerinin temel yapı taşlarıdır. Doğru kombinasyonda kullanıldığında, karmaşık veri işleme gereksinimlerini verimli bir şekilde karşılayabilirler.

# DataStage Modify Stage

Modify Stage (Değiştirme Aşaması), DataStage'de sütunları değiştirmek, yeniden adlandırmak veya kaldırmak için kullanılan bir işleme aşamasıdır. Bu aşama, Transformer kadar kapsamlı değildir, ancak basit sütun değişiklikleri için daha hafif ve performanslı bir alternatiftir.

## Modify Stage'in Temel Özellikleri

### 1. Temel Fonksiyonlar

- **Sütun Kaldırma**: İstenmeyen sütunları çıkış akışından çıkarır
- **Sütun Tutma**: Yalnızca belirtilen sütunları korur, diğerlerini çıkarır
- **Sütun Yeniden Adlandırma**: Mevcut sütunların adlarını değiştirir

### 2. Orchestrate Derleyicide Çalışma

- Transformer'dan farklı olarak, Modify Stage Orchestrate derleyicide çalışır
- Bu sayede daha hızlı derlenir ve çalıştırılır
- Transformer'ın C++ derleyicisinden daha az kaynak kullanır

## Modify Stage vs. Transformer vs. Copy Stage

### Modify Stage

- Hafif ve performanslı
- Basit sütun işlemleri için ideal
- Karmaşık mantık gerektirmeyen değişiklikler için

### Transformer

- Kapsamlı veri dönüşümleri için güçlü
- İfadeler, fonksiyonlar ve karmaşık mantık için
- Daha yavaş derlenir ve daha fazla kaynak kullanır

### Copy Stage (Modern DataStage)

- Modern DataStage sürümlerinde Modify Stage'in işlevlerini içerir
- Sütun kaldırma, yeniden adlandırma ve basit değişiklikler için kullanılabilir
- Modify Stage'e göre daha kullanıcı dostu arayüze sahip

## Modify Stage Kullanımı

### Sütun Kaldırma Örneği

1. **Specification Bölümünde**: Kaldırmak istediğiniz sütunları belirtin (`drop review`)
2. **Output Bölümünde**: Çıkış eşlemesini kontrol edin
3. **Sonuç**: Belirtilen sütunlar çıkış akışından çıkarılır

### Diğer Kullanım Şekilleri

- **Sütun Tutma**: Yalnızca belirtilen sütunları korumak için `keep` komutunu kullanabilirsiniz
- **Sütun Yeniden Adlandırma**: Sütunları yeniden adlandırmak için belirli sözdizimi kullanabilirsiniz

## Uygulama Alanları

### Ne Zaman Modify Stage Kullanılmalı

- Basit sütun değişiklikleri gerektiğinde
- Yüksek performans gerektiren işlerde
- Transformer'a göre daha hızlı derleme ve çalıştırma gerektiğinde

### Ne Zaman Transformer Kullanılmalı

- Karmaşık veri dönüşümleri gerektiğinde
- Koşullu mantık ve işlevler gerektiğinde
- Veri tiplerinin karmaşık dönüşümleri gerektiğinde

## Modern DataStage'de Durum

- Modern DataStage sürümlerinde, Modify Stage işlevleri Copy Stage içine entegre edilmiştir
- Yeni projelerde genellikle Modify Stage yerine Copy Stage tercih edilir
- Eski projelerde hala Modify Stage bulunabilir, bu nedenle nasıl çalıştığını anlamak önemlidir

Modify Stage, basit sütun değişiklikleri için hızlı ve verimli bir çözüm sunar. Ancak modern DataStage sürümlerinde, aynı işlevler daha kullanıcı dostu bir arayüze sahip olan Copy Stage ile de gerçekleştirilebilir. Modify Stage'in temel işlevlerini anlamak, özellikle eski DataStage işlerini analiz ederken faydalıdır.

# DataStage'de Surrogate Key Generator Stage Kullanımı

Surrogate Key (Yapay Anahtar), veri ambarı projelerinde sıkça kullanılan, genellikle işle doğrudan ilgisi olmayan benzersiz tanımlayıcı sayısal değerlerdir.

## Surrogate Key Nedir?

Surrogate Key, bir veri satırını benzersiz şekilde tanımlayan, otomatik olarak oluşturulan sayısal değerlerdir. Bu anahtarlar:

- Genellikle numerik (integer, bigint) veri tipindedir
- İş anlamı taşımayabilir veya iş kullanıcılarına görünmeyebilir
- Veri ilişkilerini etkin bir şekilde yönetmek için kullanılır
- Örnek olarak transaction ID, invoice ID vb. sayılabilir

## Surrogate Key Oluşturma Yöntemleri

İki temel yöntem vardır:

1. **Veritabanı Sequence Kullanarak**: Veritabanında tanımlanan sequence nesneleri kullanılır
2. **ETL Aracında Oluşturarak**: DataStage gibi ETL araçlarında oluşturulan değerler kullanılır

## DataStage'de Surrogate Key Oluşturma

DataStage'de surrogate key oluşturma iki adımlı bir süreçtir:

### İlk Adım: Başlangıç Değerini Belirleme

1. İlk iş, mevcut en yüksek surrogate key değerini belirlemek için bir düz dosya (flat file) oluşturmaktır
2. Bu dosya, sonraki değerlerin hangi sayıdan başlayacağını belirler
3. Yeni bir tablo için bu değer genellikle 0'dır

### İkinci Adım: Surrogate Key Değerlerini Oluşturma ve Yükleme

1. Oluşturulan düz dosyayı okuyarak yeni surrogate key değerleri üretilir
2. Bu değerler, yeni kayıtlarla birlikte hedef tabloya yüklenir

## Surrogate Key Generator İş Akışı Detayları

### İlk İş (Düz Dosya Oluşturma)

1. **Oracle Connector Stage** ile mevcut tablodan maksimum surrogate key değerini okuma:
    - `SELECT CASE WHEN MAX(SK_column) IS NULL THEN 0 ELSE MAX(SK_column) END AS SK_column FROM training.page_view_SK`
    - Null durumunda 0 değerini kullanma
1. **Surrogate Key Generator Stage** yapılandırması:
    - Key Column: `SK_column`
    - Source Name: Tabloyu ve sütunu belirten bir isim (örn: `page_view_SK_SK_column_first`)
    - Key Source: `Flat File`
    - Update Action: İlk çalıştırma için `Create and Update`

Bu iş, maksimum surrogate key değerini içeren bir düz dosya oluşturur.

### İkinci İş (Surrogate Key Üretme ve Yükleme)

1. **Change Capture Stage** ile yalnızca yeni kayıtları seçme:
    - Insert tipindeki değişiklikleri filtreleme
1. **Surrogate Key Generator Stage** yapılandırması:
    - Generate Output Column: `SK_column`
    - Source Name: İlk işte oluşturulan dosyanın adı
    - Generate Key From Last Highest Value: `Yes` (son en yüksek değerden başlama)
    - Sequential Mode: Sıralı sayılar için etkinleştirme (Advanced sekmesinde)
1. Üretilen surrogate key değerleriyle birlikte verileri hedef tabloya yükleme

## Önemli Yapılandırma Seçenekleri

### Update Action Seçenekleri

- **Create and Update**: İlk çalıştırma için (düz dosya yoksa oluşturur)
- **Update**: Sonraki çalıştırmalar için (mevcut düz dosyayı günceller)

### Sequential Mode vs. Paralel Mode

- **Sequential Mode**: Sıralı surrogate key değerleri oluşturur (1, 2, 3, ...)
- **Paralel Mode**: Farklı partisyonlarda boşluklu surrogate key değerleri oluşturur (örn. Partition 1: 1, 3, 5... Partition 2: 2, 4, 6...)

### Generate Key From Last Highest Value

- **Yes**: Son en yüksek değerden devam eder
- **No**: Çalıştırma bazında rastgele bir başlangıç değeri seçer

## Pratik Kullanım İpuçları

1. Surrogate key'leri ilk kez oluştururken, başlangıç değeri için doğru bir düz dosya oluşturduğunuzdan emin olun
2. Değerlerin sıralı olması önemliyse "Sequential Mode" ayarını etkinleştirin
3. İkinci ve sonraki çalıştırmalarda, Update Action'ı "Update" olarak ayarlamayı unutmayın
4. Change Capture veya Lookup kullanarak sadece yeni kayıtların surrogate key almasını sağlayın

Surrogate Key Generator Stage, veri ambarı projelerinde tablolar arasındaki ilişkileri yönetmek için benzersiz tanımlayıcılar oluşturmanın etkili bir yoludur. Doğru yapılandırıldığında, hem yeni hem de artışlı yüklemelerde tutarlı surrogate key değerleri üretilmesini sağlar.

# DataStage Transformer Aşaması Kullanımı

Transformer aşaması, DataStage'in en güçlü ve esnek bileşenlerinden biridir. Daha önce gördüğümüz Filter, Copy, Modify ve Surrogate Key Generator gibi birçok aşamanın işlevlerini tek başına gerçekleştirebilir.

## Transformer'ın Temel Özellikleri

### C++ Derleyici Temelli İşleme

- Diğer DataStage aşamalarından farklı olarak C++ derleyicisinde çalışır
- Bu sayede C++ kütüphanelerinin fonksiyonlarına erişim sağlar
- Daha kapsamlı dönüşümler yapabilir ancak daha fazla kaynak kullanır

### Stage Variables (Aşama Değişkenleri)

- Transformer içinde geçici depolama alanları oluşturmanızı sağlar
- İşlem sırasında değerleri saklamak için kullanılır
- Önceki kayıtların değerlerine erişebilmenizi sağlar

## Birikimli Hesaplama Örneği

### Senaryo

Örnek veritabanımızda şu alanlar var:

- **Name**: Kişi adı
- **Account_Number**: İsim ve hesap numarasının birleşimi
- **Amount**: İşlem tutarı
- **Date**: İşlem tarihi

### İstenen Çıktı

- İsim alanı aynen korunacak
- Account_Number alanından yalnızca hesap numarası kısmı çıkarılacak
- Amount alanı aynen korunacak
- Yeni bir "Cumulative_Amount" alanı eklenecek (kişi bazında birikimli toplam)
- Date alanı sistem formatına dönüştürülecek

## Transformer'da Stage Variables Kullanımı

### Stage Variables Oluşturma

1. Transformer içinde sağ tık yapıp "Insert New Stage Variable" seçeneğini seçin
2. Properties sekmesi > Stage Variables bölümünden değişkenleri yönetin
3. Bu örnekte şu değişkenleri oluşturduk:
    - **SV_previous_name**: Önceki kaydın isim değeri
    - **SV_current_name**: Mevcut kaydın isim değeri
    - **SV_current_amount**: Birikimli toplam tutar

### Birikimli Hesaplama Mantığı

1. Her kayıt için:
    - **SV_previous_name = SV_current_name** (Önceki isim, mevcut isim olur)
    - **SV_current_name = DSLink.name** (Mevcut isim, gelen kayıttaki isim olur)
1. Birikimli tutar hesaplama:
    - Eğer **SV_previous_name ≠ SV_current_name** ise (yeni bir kişi geldi):
        - **SV_current_amount = DSLink.amount** (Birikimli tutar sıfırlanır, yeni değer atanır)
    - Eğer **SV_previous_name = SV_current_name** ise (aynı kişinin farklı kaydı):
        - **SV_current_amount = SV_current_amount + DSLink.amount** (Birikimli tutar güncellenir)

### İlk Kayıt Durumu

- İlk kayıt için **SV_previous_name** null olacağından özel bir durum oluşur
- Bu durumda **SV_current_amount = DSLink.amount** olmalıdır

## Transformer İle Diğer Dönüşümler

Birikim hesaplaması dışında, Transformer ile aşağıdaki işlemleri de gerçekleştirebiliriz:

### String İşlemleri

- Account_Number alanından hesap numarasını çıkarmak için substring fonksiyonları
- String birleştirme, bölme, değiştirme işlemleri

### Tarih Dönüşümleri

- Formatlı tarih metnini (DD-MM-YYYY) sistem formatına dönüştürme
- Tarih aritmetiği (gün ekleme, çıkarma, tarih farkı hesaplama)

### Koşullu İşlemler

- if-then-else yapıları ile koşullu dönüşümler
- case-when-then yapıları ile karmaşık koşul işlemleri

## Transformer Kullanmanın Avantajları

1. **Esneklik**: Tek bir aşamada birden fazla dönüşüm gerçekleştirebilirsiniz
2. **Güç**: C++ fonksiyonlarına erişim sağlar
3. **Kapsamlı Dönüşümler**: Karmaşık iş mantığını uygulayabilirsiniz
4. **Geçmiş Kayıt Erişimi**: Stage variables ile önceki kayıtların değerlerine erişebilirsiniz

## Dezavantajlar

1. **Performans**: C++ derleyici kullanımı nedeniyle daha yavaş derlenir
2. **Kaynak Kullanımı**: Diğer aşamalara göre daha fazla sistem kaynağı kullanır

Transformer, DataStage'in en güçlü ve çok yönlü aşamalarından biridir. Basit işlemler için Copy veya Modify aşamaları gibi daha hafif alternatifleri tercih etmek performans açısından daha iyi olabilir, ancak karmaşık dönüşümler ve hesaplamalar için Transformer idealdir.

# DataStage Transformer Aşaması ve Döngüler

DataStage Transformer, karmaşık veri dönüşümleri için kullanılan güçlü bir aşamadır. Önceki oturumlarda temel işlevleri inceledik. Bu oturumda, özellikle döngüler ve tekrarlı işlemler üzerinde duracağız.

## Transformer'da Döngü Kullanımı

Döngüler, tek bir giriş kaydını birden fazla çıkış kaydına dönüştürmek istediğimizde kullanılır. Örneğin, virgülle ayrılmış bir değer listesini ayrı satırlara bölmek için ideal bir yöntemdir.

### Döngü Değişkenleri (Loop Variables)

1. **Değişken Oluşturma**:
    - Properties sekmesi > Loop Variables bölümünden döngü değişkenleri tanımlanır
    - Örnek: `LV_rating` gibi anlamlı bir isim verilir
1. **Döngü Koşulları**:
    - `@iteration <= SV_count + 1` gibi koşullar tanımlanır
    - `@iteration`: Mevcut döngü sayısını temsil eden sistem değişkeni

### Örnek Senaryo: Virgülle Ayrılmış Değerleri Ayrı Satırlara Bölme

İşlenecek veri:

- İsim: "Anshul", Değerlendirme: "A,B,C,D"
- İsim: "Training", Değerlendirme: "X,Y,Z,P"

İstenen çıktı:

- Anshul - A
- Anshul - B
- Anshul - C
- Anshul - D
- Training - X
- Training - Y
- Training - Z
- Training - P

### Uygulama Adımları:

1. **Virgül Sayısını Hesaplama**:
Copy

`SV_count = count(rating, ",")`

1. **Döngü Değişkeni Tanımlama**:
    - `LV_rating` isminde döngü değişkeni oluştur
1. **Döngü Koşulu Belirleme**:
Copy

`@iteration <= SV_count + 1`

(Virgül sayısı+1 kadar döngü yapılacak)

1. **Her Döngüde Değer Çıkarma**:
Copy

`LV_rating = field(rating, ",", @iteration)`

Bu fonksiyon, belirtilen sıradaki alanı döndürür

1. **Çıkış Sütunlarını Eşleme**:
    - İsim değerini doğrudan aktarma
    - Değerlendirme için döngü değişkenini kullanma

## Diğer Kullanışlı Fonksiyonlar

### String İşlemleri

- **count()**: Belirli bir karakterin bir metindeki tekrar sayısını sayar
- **field()**: Metni belirtilen ayırıcıya göre böler ve belirli bir alanı döndürür

### Benzersiz Satır Numarası Oluşturma

Partisyonlar arasında benzersiz satır numaraları oluşturmak için:

Copy

`(@inrownum - 1) * number_of_partitions + partition_number`

veya çıkış satırlarına göre:

Copy

`(@outrownum - 1) * number_of_partitions + partition_number`

- Döngü kullanıyorsanız `@outrownum` tercih edilir
- Kısıtlamalar (constraints) kullanıyorsanız `@outrownum` tercih edilir

## Pratik İpuçları

1. **Stage Variables vs. Loop Variables**:
    - Stage Variables: Kayıtlar arasında değer saklamak için
    - Loop Variables: Bir kayıt içinde tekrarlı işlem yapmak için
1. **Veri Tipi Dönüşümleri**:
    - String to Timestamp dönüşümlerinde format belirtmek önemli
    - Örnek: `string_to_date(date_column, "DD/MM/YY")`
1. **Formül Test Etme**:
    - Karmaşık formülleri çıkış sütunlarına da eşleyerek test edin
    - Beklenmedik sonuçlar alırsanız kuru çalıştırma (dry run) yapın
1. **Sistem Değişkenleri**:
    - `@inrownum`: Giriş satır numarası (partition bazlı)
    - `@outrownum`: Çıkış satır numarası (partition bazlı)
    - `number_of_partitions`: İş akışındaki partition sayısı
    - `partition_number`: Mevcut partition numarası

DataStage Transformer, karmaşık veri dönüşümleri için esnek ve güçlü bir araçtır. Döngüler ve diğer gelişmiş özellikleri kullanarak, tek kaynaktan birden fazla hedef kayıt oluşturabilir, karmaşık string manipülasyonları yapabilir ve veri yapısını radikal şekilde dönüştürebilirsiniz.

# DataStage Change Capture Stage

Change Capture Stage (Değişiklik Yakalama Aşaması), DataStage'in veri ambarı projelerinde önemli bir rol oynayan, özellikle Slowly Changing Dimension Type 2 (SCD Tip 2) uygulamalarında kullanılan güçlü bir bileşenidir. Bu aşama, iki veri kaynağı arasındaki farklılıkları tespit etmek ve değişiklikleri kategorize etmek için kullanılır.

## Change Capture Stage'in Temel İşlevi

Change Capture Stage, iki veri kaynağını karşılaştırarak kayıtları şu kategorilere ayırır:

1. **Copy (Kopyalama) - Kod 0**: Her iki kaynakta da bulunan ve hiçbir değişiklik olmayan kayıtlar
2. **Insert (Ekleme) - Kod 1**: Kaynak veride bulunan ancak hedef veride bulunmayan kayıtlar
3. **Delete (Silme) - Kod 2**: Hedef veride bulunan ancak kaynak veride bulunmayan kayıtlar
4. **Update (Güncelleme) - Kod 3**: Her iki kaynakta da bulunan ancak değerleri farklı olan kayıtlar

## Mod Seçenekleri

Change Capture Stage'de üç çalışma modu vardır:

1. **Explicit Keys, All Values (Açık Anahtarlar, Tüm Değerler)**: Anahtar sütunları belirtirsiniz, diğer tüm sütunlar değer sütunu olarak kabul edilir
2. **All Keys, Explicit Values (Tüm Anahtarlar, Açık Değerler)**: Değer sütunlarını belirtirsiniz, diğer tüm sütunlar anahtar sütunu olarak kabul edilir
3. **Explicit Keys and Values (Açık Anahtarlar ve Değerler)**: Hem anahtar hem de değer sütunlarını açıkça belirtirsiniz, diğer sütunlar karşılaştırmaya dahil edilmez

En yaygın kullanılan mod, **Explicit Keys, All Values**'dir çünkü genellikle anahtar sütunları değer sütunlarından daha azdır.

## Önemli Yapılandırma Adımları

### 1. Link Ordering (Bağlantı Sıralaması)

Bağlantı sıralaması son derece kritiktir:

- **Before Link (Önceki Bağlantı)**: Önceki (mevcut) veri kümesini temsil eder (genellikle veritabanı)
- **After Link (Sonraki Bağlantı)**: Yeni veri kümesini temsil eder (genellikle dosya)

Yanlış bağlantı sıralaması, insert/delete kayıtlarının yer değiştirmesine ve hatalı güncelleme değerlerine yol açar.

### 2. Drop Output Ayarları

Her kayıt türü için (Copy, Insert, Delete, Update) çıktının oluşturulup oluşturulmayacağını belirler:

- **true**: Bu tür kayıtlar için çıktı üretme
- **false**: Bu tür kayıtlar için çıktı üret

Örneğin, sadece Insert ve Update kayıtlarıyla ilgileniyorsanız:

- Drop Output for Copy: **true**
- Drop Output for Delete: **true**
- Drop Output for Insert: **false**
- Drop Output for Update: **false**

## İlk Yükleme ve Artışlı Yükleme Stratejisi

DataStage'de tipik bir CDC iş akışı şöyledir:

### İlk Yükleme Senaryosu

İlk çalıştırmada hedef tabloda veri olmadığından:

- Change Capture tüm kayıtları "Insert" (1) olarak işaretler
- Tüm kayıtlar hedef tabloya yüklenir

### Artışlı Yükleme Senaryosu

Sonraki çalıştırmalarda:

- Change Capture kayıtları kategorize eder (0, 1, 2, 3)
- Filter Stage veya Drop Output ayarları ile sadece Insert (1) ve Update (3) kayıtları seçilir
- Seçilen kayıtlar "Insert then Update" modunda hedef tabloya yüklenir

## Pratik Uygulama Stratejileri

### Yöntem 1: Filter Stage Kullanımı

Copy

`Change Capture Stage --> Filter Stage (change_code=1 OR change_code=3) --> Database Stage`

### Yöntem 2: Drop Output Ayarları

Copy

`Change Capture Stage (Drop Output for Copy=true, Drop Output for Delete=true) --> Database Stage`

İkinci yöntem daha basit ve verimlidir.

## Dikkat Edilmesi Gereken Noktalar

1. **Link Ordering Doğruluğu**: Before/After bağlantılarının doğru şekilde atanması kritiktir
2. **Anahtar Sütun Seçimi**: Benzersizliği garanti eden sütunlar seçilmelidir
3. **Change Code Sütunu**: İş akışında kullanışlıdır ancak genellikle hedef tabloya yüklenmez
4. **Test ve Doğrulama**: Üretim öncesi test verilerinin doğru işlendiğinden emin olun

Change Capture Stage, veri ambarı projelerinde özellikle SCD Tip 2 uygulamaları için ideal bir çözümdür. Hem ilk yükleme hem de artışlı yükleme senaryolarını tek bir iş akışıyla yönetebilirsiniz, bu da bakım kolaylığı sağlar.



# DataStage Container İşlemleri

DataStage Container (Konteyner), karmaşık iş akışlarını yönetmek ve kod yeniden kullanımını sağlamak için tasarlanmış önemli bir özelliktir. Bu kapsayıcılar, büyük DataStage işlerinin parçalarını modüler hale getirerek görselleştirmeyi basitleştirir ve bakımı kolaylaştırır.

## Container (Konteyner) Nedir?

Container, bir DataStage işinin yeniden kullanılabilir bir bileşenidir. Karmaşık bir iş akışında, belirli bir işlevselliği gerçekleştiren bir grup aşama, bir Container içinde paketlenebilir. Bu, iş akışının görsel karmaşıklığını azaltır ve bakım kolaylığı sağlar.

## Container Türleri

DataStage'de iki tür Container vardır:

### 1. Local Container (Yerel Konteyner)

- Yalnızca oluşturulduğu iş içinde kullanılır
- Aynı iş içinde belirli bir işlevi birden fazla kez kullanmanız gerektiğinde faydalıdır
- İş akışını sadeleştirmeye yardımcı olur

### 2. Shared Container (Paylaşılan Konteyner)

- Birden fazla işte kullanılabilir
- Merkezi bir yerde saklanır ve birden fazla iş tarafından referans alınabilir
- Kod bakımını kolaylaştırır: bir değişiklik yapıldığında, bu kodu kullanan tüm işler güncellenir

## Local Container Oluşturma

1. General sekmesinden Container ekleyin
2. Gruplamak istediğiniz aşamaları seçin (Ctrl+C)
3. Container içine yapıştırın (Ctrl+V)
4. Input ve output bağlantılarını konteyner içinde ve dışında bağlayın
5. Artık kullanılmayan aşamaları silin

## Local Container'ı Shared Container'a Dönüştürme

1. Local Container'a sağ tıklayın
2. "Convert to Shared" seçeneğini seçin
3. Shared Container'a bir isim verin ve kaydedin
4. Container artık paylaşılabilir ve diğer işlerde kullanılabilir

## Bağlantı Eşleme (Link Mapping)

Container'ları kullanırken dikkat edilmesi gereken en önemli nokta, giriş ve çıkış bağlantılarının doğru eşlenmesidir:

1. Container'ın Properties sekmesine gidin
2. Input bölümünde, ana işteki giriş bağlantılarını konteyner içindeki bağlantılarla eşleyin
3. Output bölümünde, konteyner içindeki çıkış bağlantılarını ana işteki bağlantılarla eşleyin
4. "Validate" düğmesine tıklayarak eşleşmeyi doğrulayın
    - Bu, bağlantılardaki sütunların doğru bir şekilde eşleştiğini kontrol eder

## Container Kullanmanın Avantajları

1. **Görsel Sadelik**: Karmaşık işler daha yönetilebilir hale gelir
2. **Kod Yeniden Kullanımı**: Ortak işlevleri birden fazla yerde kullanabilirsiniz
3. **Bakım Kolaylığı**: Değişiklikleri bir yerde yapabilir ve tüm kullanımlara uygulayabilirsiniz
4. **Modülerlik**: İş akışınızı mantıksal bileşenlere ayırabilirsiniz
5. **Ekip Çalışması**: Farklı ekip üyeleri farklı kapsayıcılarda çalışabilir

Container'lar, özellikle büyük ve karmaşık DataStage projelerinde çalışırken, iş akışı tasarımını daha modüler ve yönetilebilir hale getirmek için güçlü bir araçtır. Hem görsel karmaşıklığı azaltır hem de kod yeniden kullanımını teşvik eder.

# DataStage'de Annotation (Açıklama) İşlemleri

DataStage'de projelerin anlaşılabilirliğini artırmak için iki farklı açıklama mekanizması bulunur: Annotation (Açıklama) ve Description Annotation (Açıklama Tanımı). Bu araçlar, karmaşık iş akışlarını belgelemek ve bakım yapan geliştiricilere yardımcı olmak için kullanılır.

## 1. Annotation (Açıklama)

Annotation, iş akışının belirli bölümlerini açıklamak için kullanılan etiketlerdir.

### Özellikleri:

- Bir iş içinde birden fazla annotation kullanılabilir
- İş akışının belirli bölümlerini veya aşamalarını açıklamak için idealdir
- Genellikle belirli bir stage grubu veya mantıksal bölüm için kullanılır
- Kod içindeki yorumlara benzer işlev görür

### Kullanım Alanları:

- Karmaşık iş mantığını açıklamak
- Belirli aşamaların veya konteynerların amacını belirtmek
- İş akışındaki önemli dönüşüm noktalarını vurgulamak

### Örnek:

"Implementing filter and funnel" - Bir filter ve funnel stage grubunu açıklamak için

## 2. Description Annotation (Açıklama Tanımı)

Description Annotation, tüm işi açıklamak için kullanılan kapsamlı bir belgeleme aracıdır.

### Özellikleri:

- Her iş için yalnızca bir description annotation kullanılabilir
- İşin genel amacını ve temel özelliklerini belgelemek için kullanılır
- Hem kısa (Short Description) hem de uzun (Full Description) açıklama içerebilir
- İşin üst kısmında bulunması önerilir

### Kullanım Alanları:

- İşin genel amacını belirtmek
- Oluşturma tarihi, değişiklik tarihi gibi meta verileri içermek
- İşle ilgili teknik veya iş gereksinimlerini belgelemek
- İletişim bilgileri veya sorumlu geliştirici bilgilerini eklemek

### Description Annotation'a Erişim:

1. Doğrudan canvas üzerinden ekleme
2. Job Properties menüsünden:
    - Short Job Description
    - Long Job Description

### Biçimlendirme Seçenekleri:

- Arka plan rengi değiştirilebilir
- Yatay hizalama (sol, orta, sağ)
- Dikey hizalama (üst, orta, alt)

## İyi Uygulama Önerileri

1. **Her iş için bir Description Annotation kullanın**: İşin genel amacını, oluşturma tarihini ve sorumlu geliştiriciyi belirtin
2. **Karmaşık bölümleri Annotation ile açıklayın**: Özellikle karmaşık dönüşümlerin, aggregation işlemlerinin veya business logic'in uygulandığı bölümleri
3. **Standart bir format belirleyin**: Tüm projelerde tutarlı bir annotation formatı kullanın
4. **Güncel tutun**: İşte değişiklik yapıldığında annotation'ları da güncelleyin
5. **İşin üst kısmına Description Annotation yerleştirin**: İşi ilk açan kişi hemen genel bilgiyi görebilsin

Annotation ve Description Annotation, DataStage işlerinin bakımını yapacak geliştiricilere önemli bağlam bilgisi sağlar ve büyük projelerin yönetimini önemli ölçüde kolaylaştırır. Kod içindeki yorumlar gibi, düzenli ve anlamlı açıklamalar eklemek, tüm veri entegrasyonu projelerinde iyi bir uygulama olarak kabul edilir.

# DataStage'de XML Dosyalarını Okuma

XML dosyaları, veri entegrasyonu projelerinde sıkça kullanılan yapılandırılmış veri biçimleridir. DataStage'de XML dosyalarını okumak için XML Stage bileşeni kullanılır. Bu eğitimde, XML dosyalarını DataStage'de nasıl işleyeceğimizi adım adım öğreneceğiz.

## XML ve XSD Dosyaları

XML (eXtensible Markup Language) dosyaları veriyi yapılandırılmış bir şekilde saklar, ancak bunları doğru şekilde işleyebilmek için veri şeması bilgilerine ihtiyaç duyarız. Bu şema bilgileri XSD (XML Schema Definition) dosyalarında bulunur.

### XSD Dosyaları (XML Şema Tanımı)

- XML dosyasının yapısını tanımlar
- Veri tiplerini belirtir
- XML dosyasının doğrulanmasını sağlar
- DataStage'in XML verisini nasıl yorumlayacağını belirler

Eğer proje kapsamında XSD dosyası verilmezse, XML dosyasından XSD dosyası oluşturmak için çevrimiçi araçlar kullanılabilir:

- Freeformatter.com gibi web siteleri
- Genellikle "Russian Doll" tasarım şablonu tercih edilir

## XML Stage Kullanımı

DataStage'de XML dosyaları okumak için şu adımlar izlenir:

### 1. Yeni Bir İş Oluşturma

- Parallel Job oluşturun
- Gerekli parametreleri ekleyin (örn: dosya yolu parametresi)

### 2. XML Stage'i Ekleme

- Real-Time bölümünden XML Stage'i seçin
- Hierarchical Data Stage olarak da bilinir (DataStage 11.3 ve sonrası)
- Çıkış için bir Data Set veya File Set bileşeni ekleyin

### 3. Assembly Editor'ü Açma

- XML Stage üzerinde "Edit Assembly" düğmesine tıklayın
- Burası XML işlemleri için özel bir düzenleyicidir

### 4. XSD Dosyasını İçe Aktarma

- Library sekmesine gidin
- Yeni bir kütüphane oluşturun (örn: "test")
- "Import New Source" ile XSD dosyasını içe aktarın
- Hata durumunda, ekranın üstünde bir uyarı görüntülenecektir

### 5. XML Parser Ekleme

- Palette sekmesinden XML Parser bileşenini seçin
- XML dosyasının yolunu belirtin:

```other
[Parameter: dosya_yolu]/book.xml
```

- Document Root olarak ana XML etiketini seçin (örn: "books")

### 6. Çıkış Sütunlarını Tanımlama

- Output sekmesine gidin
- XML'den çıkarmak istediğiniz sütunları ekleyin
- Sütun veri tiplerini belirleyin (genellikle VARCHAR önerilir)
- Nullable ayarlarını yapın

### 7. Eşleştirme (Mapping) Ayarları

- Mapping sekmesine gidin
- XML elemanlarını çıkış sütunlarıyla eşleştirin
- Aynı isimlere sahip alanlar otomatik olarak önerilir

### 8. İşi Çalıştırma ve Sonuçları Kontrol Etme

- Tüm yapılandırmaları tamamlayıp işi çalıştırın
- Data Set'te sonuçları kontrol edin

## Performans İpuçları

XML dosyalarını işlemek genellikle CSV gibi düz dosyalara göre daha yavaştır. Performansı artırmak için:

1. **Minimum Dönüşüm Stratejisi**: XML Stage'de sadece veriyi okuyun ve dönüşüm yapmadan Data Set'e aktarın
2. **İki Aşamalı İşleme**: XML'den veriyi bir Data Set'e çıkarın, sonra başka bir işte dönüşümleri yapın
3. **Uygun Veri Tipi Seçimi**: İlk aşamada tüm alanları VARCHAR olarak alın, daha sonra dönüşüm yapın
4. **Runtime Column Propagation Kullanmama**: RCP yerine manuel olarak sütunları tanımlayın

Bu yaklaşım, özellikle büyük XML dosyaları ile çalışırken performans avantajı sağlar ve XML aşamasında gerçekleşebilecek olası hataları azaltır.

XML Stage, karmaşık XML dosyalarını işlemek için güçlü bir araçtır, ancak doğru şekilde yapılandırılması gerekir. XSD şeması ve doğru eşleştirme ile, DataStage XML verilerini kolayca işleyebilir ve diğer formatlara dönüştürebilir.

# DataStage'de İş Arama ve Bağımlılıkları Bulma

DataStage, oluşturulmuş işleri bulmak ve bağımlılıkları tespit etmek için güçlü arama seçenekleri sunar. Bu özellikler, özellikle büyük projelerde çalışırken işleri yönetmek ve bakım yapmak için çok değerlidir.

## Quick Find ile İş Arama

DataStage'in üst kısmında bulunan "Open Quick Find" seçeneği, çeşitli kriterlere göre işleri aramanıza olanak tanır:

1. **İsim ile Arama**:
    - Joker karakterler kullanılabilir (örn: `J_*` tüm "J_" ile başlayan işleri bulur)
    - Arama sonuçları, toplam eşleşme sayısıyla birlikte görüntülenir
1. **Tarih Filtresi**:
    - "Last modification on or after" (Son değişiklik tarihi)
    - "Created on or after" (Oluşturulma tarihi)
    - Belirli bir tarih aralığı seçilebilir
    - Önceden tanımlanmış seçenekler: "Today", "Up to week ago" vb.
1. **Arama Sonuçları**:
    - İş adı
    - Oluşturan kullanıcı
    - Oluşturulma tarihi
    - Son değişiklik tarihi
    - İş türü
    - Klasör yolu

## Bağımlılıkları Bulma

DataStage'de bir bileşenin (tablo tanımı, aşama, vb.) nerede kullanıldığını bulmak için "Where Used" ve "Dependencies" seçenekleri kullanılabilir:

1. **Where Used (Nerede Kullanıldı)**:
    - Bir tablo tanımı, aşama veya diğer bileşenlerin hangi işlerde kullanıldığını gösterir
    - Örnek: Bir tablo tanımının hangi işlerde kullanıldığını bulmak için
    - Bakım ve etki analizi için çok değerlidir
1. **Dependencies (Bağımlılıklar)**:
    - Bir bileşenin bağlı olduğu diğer bileşenleri gösterir
    - "Show Dependencies Deep" seçeneği, tüm bağımlılık zincirini gösterir

## Pratik Kullanım Senaryosu

Veritabanı şeması değişikliği senaryosu:

1. Veritabanında tablo değişikliği yapılması gerekiyor
2. Bu değişiklikten etkilenecek tüm işleri bulmanız gerekiyor
3. Table Definitions > Database > [Veritabanı] > [Tablo] yolunu izleyin
4. "Where Used" seçeneğini kullanarak, bu tablo tanımını kullanan tüm işleri bulun
5. İlgili tüm işleri güncelleyin

Bu yaklaşım, standart uygulama olarak DataStage işlerinde merkezi tablo tanımları kullanıldığında en iyi şekilde çalışır. Her iş için ayrı tablo tanımları oluşturmak yerine, paylaşılan tanımlar kullanmak, bakım işlerini büyük ölçüde kolaylaştırır.

## Diğer Arama Seçenekleri

- **Folder to Search (Aranacak Klasör)**: Aramayı belirli klasörlerle sınırlandırabilirsiniz
- **Type (Tür)**: Belirli türdeki bileşenleri (iş, aşama, tablo tanımı vb.) arayabilirsiniz
- **By User (Kullanıcıya Göre)**: Belirli bir kullanıcının oluşturduğu veya değiştirdiği bileşenleri bulabilirsiniz

DataStage arama özellikleri, özellikle karmaşık projelerde kritik öneme sahiptir. İşleri ve bağımlılıkları hızlı bir şekilde bulabilmek, iş akışlarını yönetmeyi ve bakım yapmayı çok daha verimli hale getirir.

# DataStage Director Kullanımı

DataStage Director, işlerin durumunu izlemek, logları incelemek ve işleri yönetmek için kullanılan önemli bir araçtır. DataStage Designer'da iş tasarımı yaptıktan sonra, Director ile bu işlerin yürütülmesini ve durumunu izleyebilirsiniz.

## Director'a Giriş

Director'a erişmek için:

- Director Client'ı açın
- Hostname, username, password ve proje bilgilerini girin
- Bağlantıyı tamamlayın

## İş Durumunu İzleme

Director, çalıştırılan tüm işlerin durumunu gösterir:

### İş Durumları:

- **Finished (Tamamlandı - Kod 1)**: İş uyarı olmadan tamamlandı
- **Finished with Warning (Uyarı ile Tamamlandı - Kod 2)**: İş uyarılarla tamamlandı
- **Aborted (İptal Edildi - Kod 3)**: İş tamamlanamadı, hata ile sonlandı
- **Compiled (Derlendi)**: İş derlendi ama henüz çalıştırılmadı
- **Running (Çalışıyor - Kod 0)**: İş şu anda çalışıyor

Bu durum kodları, özellikle Sequencer işleri tasarlarken tetikleme koşulları için önemlidir.

## Logları İnceleme

İş loglarını görüntülemek için:

1. İşi seçin
2. Notepad simgesine tıklayın (log görüntüleme)

Loglar şunları içerir:

- İşin başlangıç ve bitiş zamanı
- Kontrol bilgileri
- Yapılandırma dosyası detayları
- Çevre değişkenleri
- Uyarılar ve hatalar
- Her aşamanın işlediği kayıt sayısı

## Monitor Özelliği

İşlerin performansını incelemek için Monitor özelliği kullanılır:

1. İşi seçin
2. "Monitor" seçeneğine tıklayın

Monitor ekranında:

- Her aşamanın işlediği kayıt sayısı görüntülenir
- Giriş ve çıkış bağlantılarındaki veri akışı izlenir
- Performans darboğazları tespit edilebilir

Bu özellik özellikle üretim ortamında kullanışlıdır, çünkü Designer'a erişim olmasa bile iş akışını izlemenize olanak tanır.

## İş Zamanlama

Director'dan işleri zamanlamak için:

1. İşi seçin
2. "Add to Schedule" seçeneğine tıklayın
3. Zamanlama seçeneklerini belirleyin:
    - Günlük, haftalık veya aylık çalıştırma
    - Belirli günler (örn: Pazartesiden Cumaya)
    - Çalıştırma saati

DataStage sadece DataStage işlerini zamanlamaya izin verdiği için, karmaşık zamanlama ihtiyaçları için genellikle dış zamanlayıcılar tercih edilir.

## Parametre Değerlerini İnceleme

İşlerin hangi parametrelerle çalıştırıldığını görmek için:

1. İşi çift tıklayın veya "Detail" seçeneğini kullanın
2. Parametre listesi ve değerleri görüntülenecektir

Bu bilgiler, özellikle bir işi farklı ortamlarda (test, üretim vb.) çalıştırırken faydalıdır.

## Message Handler Kullanımı

Belirli uyarıları yönetmek için Message Handler kullanılabilir:

1. Uyarıyı sağ tıklayın
2. "Add rule to Message Handler" seçeneğini seçin
3. Eylem seçin:
    - **Suppress from log**: Uyarıyı tamamen gizle
    - **Demote to informational**: Uyarıyı bilgi mesajına dönüştür

Bu özellik, önemsiz veya kaçınılmaz uyarıların log dosyalarını doldurmasını önlemek için kullanılabilir, ancak gerçek sorunları gizleme riski taşıdığından dikkatli kullanılmalıdır.

## Log Temizleme

Eski logları temizlemek için:

1. İşi seçin
2. "Clear Logs" seçeneğini kullanın
3. Otomatik temizleme ayarlarını yapılandırın (örn: 1 günden eski logları temizle)

Bu ayar sadece seçilen iş için geçerlidir. Tüm proje için log temizleme ayarları Administrator aracında yapılır.

DataStage Director, özellikle üretim ortamlarında işlerin durumunu izlemek ve yönetmek için kritik bir araçtır. Designer'da oluşturulan işlerin performansını ve durumunu izlemek, sorunları tanımlamak ve gerektiğinde müdahale etmek için kullanılır.

# DataStage Sequencer İşleri ve Bağımlılık Yönetimi

Sequencer işleri, DataStage'de işler arasında bağımlılıkları yönetmek ve karmaşık iş akışları oluşturmak için kullanılan güçlü bir araçtır. Bu eğitimde, Sequencer kullanarak işler arasında bağımlılıklar oluşturmayı ve koşullu dallanmaları yönetmeyi öğreneceğiz.

## Sequencer İşlerinin Amacı

Sequencer işleri şu amaçlar için kullanılır:

- İşler arasında bağımlılıklar oluşturmak
- Koşullu iş akışları tasarlamak
- İş durumlarına göre farklı eylemler gerçekleştirmek
- E-posta bildirimleri göndermek
- Dosya bekleme ve komut çalıştırma işlemlerini yönetmek

## Sequencer İşi Bileşenleri

### 1. Job Activity (İş Aktivitesi)

- Bir DataStage işini çalıştırmak için kullanılır
- Parametreleri iletebilir
- Reset ve çalıştırma seçenekleri belirlenebilir
- Tetikleme koşulları ayarlanabilir

### 2. Tetikleme Koşulları

İş tamamlandıktan sonra uygulanan koşullar:

- **OK (Kod 1)**: İş uyarı olmadan tamamlandığında
- **Warning (Kod 2)**: İş uyarılarla tamamlandığında
- **Failed (Kod 3)**: İş başarısız olduğunda/iptal edildiğinde
- **Unconditional**: Durumdan bağımsız her zaman çalışır
- **Custom**: Özel koşullar tanımlanabilir (örn: `job.status=1 OR job.status=2`)
- **Otherwise**: Diğer tüm koşullar karşılanmadığında

### 3. Sequencer Activity (Sıralayıcı Aktivitesi)

- Farklı dallardan gelen akışları birleştirmek için kullanılır
- İki modu vardır:
    - **Any**: Herhangi bir daldan aktivasyon geldiğinde tetiklenir
    - **All**: Tüm dallardan aktivasyon geldiğinde tetiklenir

### 4. Terminator Activity (Sonlandırıcı Aktivitesi)

- Sequencer işini sonlandırmak için kullanılır
- Hata durumlarında veya süreç tamamlandığında kullanılabilir

### 5. Exception Handler (İstisna İşleyici)

- Sequencer işi sırasında oluşabilecek istisnaları yakalar
- Her sequencer işinde yalnızca bir istisna işleyici olabilir

### 6. Diğer Aktiviteler

- **Notification Activity**: E-posta bildirimleri gönderir
- **Execute Command**: Shell komutları çalıştırır
- **Wait for File**: Bir dosyanın oluşmasını veya silinmesini bekler
- **Routine Activity**: DataStage rutinlerini çalıştırır

## Önemli Sequencer Ayarları

1. **Add checkpoint to sequencer**:
    - İş başarısız olduğunda yeniden başlatma noktası sağlar
    - Sadece başarısız olan işten devam etmenizi sağlar, tüm işleri yeniden çalıştırmaz
1. **Log warnings after activities that finish with status other than OK**:
    - Uyarı veya hatayla biten işler için log oluşturur
    - Sequencer'ın durumunun, içindeki işlerin durumunu doğru yansıtmasını sağlar
1. **Automatically handle activities that fail**:
    - Başarısız olan işleri otomatik olarak yöneterek işlemi sürdürür
1. **Log report message after each job run**:
    - Her iş çalıştırmasından sonra bir rapor oluşturur

## Sequencer İşi Oluşturma Örnekleri

### Örnek 1: Temel Bağımlılık

```other
İş 1 (Surrogate Key Flat File Oluşturma) --> İş 2 (Surrogate Key Generation)
```

- İş 2, İş 1 başarıyla tamamlandığında çalışır
- OK tetikleyici koşulunu kullanır

### Örnek 2: Koşullu Dallanma

```other
İş 1 (Sorgu)
  |--> [OK/Warning] --> İş 2 (Başarılı işlem)
  |--> [Failed] --> İş 3 (Hata işleme)
```

- İş 1 başarılı veya uyarıyla biterse İş 2 çalışır
- İş 1 başarısız olursa İş 3 çalışır
- Custom tetikleyici ve Otherwise kullanılır

### Örnek 3: Sequencer Activity ile Birleştirme

```other
İş 1 --> \
           |--> [Any] --> İş 3 (İşlemi tamamla)
İş 2 --> /
```

- İş 1 veya İş 2'den herhangi biri tamamlandığında İş 3 çalışır
- Sequencer Activity mode: Any

## Pratik İpuçları

1. **İş Ekleme**: Designer'dan işleri sürükleyip bırakmak, iş adını ve parametreleri otomatik olarak doldurur
2. **Otherwise Kullanımı**: Otherwise daima son tetikleyici koşul olmalıdır
3. **Checkpoint Kullanımı**: İş akışı yeniden başlatıldığında zaman kazandırır ve yinelenen çalıştırmaları önler
4. **Sequencer Durumu**: Sequencer işinin durum kodunun içindeki işlerden etkilenmesi için log warnings ayarını etkinleştirin
5. **Director'dan İzleme**: Sequencer işlerini ve içindeki işleri DataStage Director'dan izleyebilirsiniz

Sequencer işleri, karmaşık iş akışlarını tasarlamak ve yönetmek için DataStage'in sunduğu en güçlü özelliklerden biridir. Doğru tetikleyici koşulları ve yapılandırmayla, ETL süreçlerinizi tam olarak kontrol edebilir ve hataları etkili bir şekilde yönetebilirsiniz.

# DataStage İş İhracat ve İthalat İşlemleri

DataStage işlerini farklı ortamlar arasında taşımak için ihracat (export) ve ithalat (import) özellikleri kullanılır. Bu işlemler, geliştirme ortamından test veya üretim ortamlarına kod geçişlerini yönetmek için kritik öneme sahiptir.

## İş İhracat (Export) İşlemi

İşleri dışa aktarmak için iki yöntem vardır:

1. İşi seçip "Export" düğmesine tıklayarak
2. Menüden "Import/Export" seçeneğini kullanarak

### İhracat Seçenekleri:

#### 1. Export Format

- **DSX**: DataStage'in standart ihracat formatı

#### 2. İçerik Türü Seçenekleri

- **Export job with executables (Yürütülebilirlerle birlikte ihraç et)**:
    - İş tasarımı ve derlenmiş Orchestrate betikleri dahil edilir
    - Hedef ortamda yeniden derleme gerektirmez (sistem yapılandırmaları aynıysa)
    - En yaygın kullanılan seçenektir
- **Export job design without executables (Yürütülebilirler olmadan ihraç et)**:
    - Sadece iş tasarımı ihraç edilir
    - Hedef ortamda kullanmadan önce derleme gerektirir
    - Farklı sistem yapılandırmaları arasında taşıma için daha güvenlidir
- **Executables only without design (Sadece yürütülebilirler, tasarım olmadan)**:
    - Sadece derlenmiş Orchestrate betikleri ihraç edilir, tasarım görünümü olmaz
    - Üretim ortamlarında kod değişikliklerini engellemek için kullanılır
    - Hata ayıklama daha zordur ancak güvenlik açısından daha sıkıdır

#### 3. Diğer Seçenekler

- **Include dependent items (Bağımlı öğeleri dahil et)**:
    - Parametre setleri, tablo tanımları gibi bağımlılıkları dahil eder
    - Genellikle işaretli tutulmalıdır
- **Append to existing file (Var olan dosyaya ekle)**:
    - Mevcut bir DSX dosyasına yeni içerik ekler
    - Takım çalışmasında farklı kişilerin işlerini tek bir paket halinde toplamak için kullanışlıdır

## İş İthalat (Import) İşlemi

İşleri içe aktarmak için:

1. "Import" seçeneğine tıklayın
2. "DataStage Components" seçin
3. DSX dosyasını belirtin

### İthalat Seçenekleri:

#### 1. İthalat Modu

- **Import All (Tümünü İçe Aktar)**:
    - DSX dosyasındaki tüm bileşenleri içe aktarır
    - Seçim yapmadan hızlıca içe aktarmak için kullanılır
- **Import Selected (Seçilenleri İçe Aktar)**:
    - DSX dosyasındaki bileşenlerden hangilerinin içe aktarılacağını seçmenize olanak tanır
    - Sadece belirli işleri veya bileşenleri içe aktarmak istediğinizde kullanışlıdır

#### 2. Conflict Resolution (Çakışma Çözümleme)

İthalat sırasında, bileşenlerin durumu gösterilir:

- **Does not exist (Mevcut değil)**: Hedef ortamda bu bileşen yok
- **Exists (Mevcut)**: Hedef ortamda aynı isimde bir bileşen var
    - Bu durumda, mevcut bileşenin üzerine yazılır

## İyi Uygulama Önerileri

1. **Bağımlılıkları dahil edin**: Her zaman "Include dependent items" seçeneğini işaretleyin
2. **Ortam Geçişlerinde Executable Durumu**:
    - Benzer ortamlar arasında: "With executables" kullanabilirsiniz
    - Farklı yapılandırmalar arasında: "Without executables" daha güvenlidir
1. **Versiyonlama**: DSX dosyalarını versiyon kontrolü ile yönetin
2. **Güvenlik Gereksinimleri**: Üretim ortamında tasarım görünümünü kısıtlamak gerekiyorsa "Executables only" seçeneğini kullanın
3. **Toplu Taşıma**: Birden çok iş birlikte taşınıyorsa, "Append to existing file" ile tek bir paket oluşturun

DataStage ihracat ve ithalat özellikleri, farklı ortamlar arasında kod taşıma ve değişiklik yönetimini kolaylaştırır. Doğru seçeneklerle kullanıldığında, geliştirme-test-üretim döngüsünü sorunsuz bir şekilde yönetmenize yardımcı olur.

# DataStage Server ve Parallel Rutinler

DataStage'de rutinler, standart bileşenlerle gerçekleştirilemeyen özel işlevler oluşturmak için kullanılır. Bu eğitimde, Server Rutinleri ve Parallel Rutinleri inceleyeceğiz.

## Server Rutinler

Server rutinler, C++ dilinde yazılan ve Server Jobs içinde kullanılan özel kod parçalarıdır.

### Özellikleri:

- C++ dilinde kodlanır
- Sarı renkle gösterilir (Server bileşenleri gibi)
- Özel dönüşümler ve işlevler için kullanılır
- DataStage'de olmayan özellikleri uygulamak için idealdir

### Server Rutin Oluşturma:

1. **File > New > Routine > Server Routine**
2. **Rutin Bilgileri**:
    - Rutin adı
    - Dönüşüm tipi (transformation function, before/after subroutine, custom function)
    - Açıklama
1. **Argümanlar**:
    - Rutin tarafından kullanılacak argümanları tanımlama
1. **Kod**:
    - C++ kodu yazma alanı
    - Önceden tanımlı argümanlar fonksiyona otomatik eklenir
    - "ans" değişkeni dönüş değeri olarak kullanılır
1. **Derleme ve Test**:
    - Rutini kaydetme
    - Derleme ve test etme seçenekleri

### Dağıtım:

- Derlenmiş kodun DataStage bin klasörüne yerleştirilmesi gerekir
- Bu genellikle yönetici tarafından gerçekleştirilir

## Parallel Rutinler

Parallel rutinler, Server rutinlerin parallel ortamda çalıştırılmasını sağlayan arayüzlerdir.

### Özellikleri:

- Kırmızı renkle gösterilir (Parallel bileşenleri gibi)
- Server rutinlere göre daha sınırlı seçeneklere sahiptir
- Kod yazma alanı yoktur
- Mevcut Server rutinleri çağırmak için kullanılır

### Parallel Rutin Oluşturma:

1. **File > New > Routine > Parallel**
2. **Rutin Bilgileri**:
    - Rutin adı
    - External subroutine/function tipi
    - Çağrılacak Server rutinin adı
    - Gerekli kütüphane yolları
    - Açıklama ve dönüş tipi
1. **Argümanlar**:
    - Server rutine iletilecek argümanları tanımlama

## Rutinleri Kullanma

Rutinler, DataStage işlerinde çeşitli şekillerde kullanılabilir:

### İş Parametreleri Aracılığıyla:

- **Before Job Subroutine**: İş başlamadan önce çalışır
- **After Job Subroutine**: İş tamamlandıktan sonra çalışır

### IBM Tarafından Sağlanan Hazır Rutinler:

- Send mail
- Wait for file
- Execute shell commands
- Execute DOS/Windows commands

## Kullanım Alanları

Rutinler aşağıdaki durumlarda faydalıdır:

- DataStage'in standart bileşenleriyle yapılamayan işlemler
- Karmaşık dönüşümler ve hesaplamalar
- Sistem düzeyinde işlemler
- Özel dosya işlemleri

## Önemli Notlar

1. Server rutinler C++ bilgisi gerektirir
2. Parallel rutinlerde kod yazılamaz, sadece Server rutinleri çağrılır
3. Modern DataStage sürümlerinde, çoğu ihtiyaç için Sequencer gibi bileşenler kullanılabilir
4. Rutinlerin doğru derlenmesi ve dağıtılması için genellikle yönetici desteği gerekir

Server ve Parallel rutinler, günümüzde eskisi kadar yaygın kullanılmasa da, özel gereksinimler için hala değerli araçlardır. Ancak çoğu durumda, daha kolay kullanılabilen ve bakımı daha basit olan standart DataStage bileşenlerini tercih etmek daha iyi bir yaklaşımdır.

# DataStage'in Nadir Kullanılan Aşamaları

Bu eğitimde, DataStage'de bulunan ancak proje ortamlarında nadiren kullanılan aşamaları inceleyeceğiz. Bu aşamalar, standart DataStage uygulamalarında çok sık karşılaşılmasa da, özel senaryolarda kullanışlı olabilirler.

## Geliştirme ve Hata Ayıklama Aşamaları

### Column Generator Stage (Sütun Üretici Aşaması)

- **Amaç**: Mevcut veri akışına ek sütunlar eklemek için kullanılır
- **Kullanım Senaryosu**: Hassas verilerin bulunmadığı test ortamlarında, eksik sütunlar için test verileri oluşturmak
- **İşleyiş**: Giriş akışındaki her kayıt için belirtilen yeni sütunlara veri üretir

### Head Stage (Baş Aşaması)

- **Amaç**: Veri akışının başından belirli sayıda kayıt almak
- **Peak Stage ile Farkı**: Head Stage kaydı veri akışına geçirirken, Peak Stage sadece gösterir
- **Önemli Ayarlar**:
    - "Number of Rows per Partition": Partition başına alınacak kayıt sayısı
    - "All Partitions": Tüm partitionlardan kayıt alınıp alınmayacağı

### Tail Stage (Kuyruk Aşaması)

- **Amaç**: Veri akışının sonundan belirli sayıda kayıt almak
- **İşleyiş**: Head Stage ile aynı ancak son kayıtları alır
- **Dikkat**: Partition sayısına göre toplam kayıt sayısı değişebilir

## İşleme Aşamaları

### Compare, Difference, Change Apply

- **Eski SCD Uygulama Aşamaları**: Slowly Changing Dimension (SCD) uygulamaları için tasarlanmış
- **Modern Alternatif**: Change Capture Stage bu işlevlerin çoğunu birleştirir
- **Kullanılmama Nedeni**: "Kara kutu" gibi çalışırlar, süreç kontrolü ve şeffaflık eksiktir

### Checksum Stage

- **Amaç**: Kayıtlar için özet değeri (checksum) oluşturur
- **Kullanım Senaryosu**: Veri bütünlüğünü doğrulamak, aktarım sırasında değişiklikleri tespit etmek
- **Hesaplama Modu**: "All Columns" - tüm sütunlara dayalı checksum oluşturur

### Compress ve Expand Stages

- **Amaç**: Veriyi sıkıştırmak ve açmak
- **İşleyiş**: Arka planda Unix `tar`, `gunzip` gibi komutları kullanır
- **Kısıtlama**: Yalnızca Unix/Linux platformlarında çalışır

### Encode ve Decode Stages

- **Amaç**: Veriyi şifrelemek ve çözmek
- **Güvenlik Notu**: Sadece DataStage bu şifrelemeyi açabilir
- **Kullanım**: Hassas verilerin dosya tabanlı saklanması gerektiğinde

### Switch Stage

- **Amaç**: Case-when-then mantığı uygulayarak kayıtları farklı çıkışlara yönlendirmek
- **İşleyiş**: Koşula bağlı olarak kayıtlar yalnızca bir çıkışa yönlendirilir

## Önemli Notlar

1. Bu aşamaların proje ortamlarında kullanılma olasılığı oldukça düşüktür (%1 civarında)
2. Modern DataStage uygulamalarında, SCD implementasyonları genellikle Change Capture Stage ile yapılır
3. Bazı aşamalar (Compress, Expand) platform bağımlıdır ve yalnızca Unix/Linux'ta çalışır
4. Head ve Tail aşamalarını kullanırken partition sayısına dikkat edilmelidir, aksi halde beklenenden farklı sayıda kayıt üretilebilir

Bu aşamalar, standart ETL işlemlerinin dışına çıkan özel ihtiyaçlar için DataStage'de bulunmaktadır. Temel işlevlerini anlamak, özel senaryolarda kullanılabileceklerini bilmek faydalı olacaktır.

# DataStage Gelişmiş Özellikler

DataStage'de bulunan ve daha nadir kullanılan, ancak belirli durumlarda çok yararlı olabilen bazı gelişmiş özellikleri inceleyeceğiz. Bu özellikler, özellikle karmaşık projelerde ve farklı ortamlar arasında geçişlerde değerli olabilir.

## Çoklu İş Derleme (Multiple Job Compile)

DataStage'de birden fazla işi aynı anda derlemek için kullanılan bir özelliktir.

### Kullanım Senaryoları:

- **Sürüm Yükseltme**: 9.1'den 11.3'e veya 8.5'ten 8.7'ye gibi üst sürümlere geçişlerde
- **Toplu İş Güncellemesi**: Çok sayıda işin aynı anda derlenmesi gerektiğinde

### Nasıl Kullanılır:

1. Herhangi bir işe sağ tıklayın
2. "Multiple Job Compile" seçeneğini seçin
3. Derlemek istediğiniz işleri ekleyin
    - "Add" butonu ile işleri ekleyebilirsiniz
    - "Quick Find" ile isim desenine göre iş arayabilirsiniz
1. "Force Compile" seçeneğini işaretleyin (iş başka yerde açık olsa bile derlemeye çalışır)
2. "Start Compile" ile derleme işlemini başlatın
3. İşlem sonucunda rapor gösterilir

## İş Karşılaştırma (Job Comparison)

İki işin yapısal benzerliklerini ve farklılıklarını görmek için kullanılan özellik.

### İki Karşılaştırma Türü:

1. **Aynı Proje Karşılaştırması**: Aynı proje içindeki işleri karşılaştırma
2. **Çapraz Proje Karşılaştırması**: Farklı projelerdeki işleri karşılaştırma

### Aynı Proje Karşılaştırması Nasıl Yapılır:

1. Bir işe sağ tıklayın
2. "Compare Against" seçeneğini seçin
3. Karşılaştırmak istediğiniz ikinci işi seçin
4. Karşılaştırma sonuçları gösterilir
    - İki iş arasındaki farklılıklar gösterilir
    - Değiştirilmiş aşama ve özellikler listelenir

### Çapraz Proje Karşılaştırması Nasıl Yapılır:

1. Bir işe sağ tıklayın
2. "Compare Against > Job in Another Project" seçeneğini seçin
3. Diğer projeye bağlanın (giriş yapmanız gerekebilir)
4. Karşılaştırmak istediğiniz işi seçin

**Not**: Çapraz proje karşılaştırması için, diğer projeye erişim izninizin olması gerekir.

## Özel Apt Yapılandırma Dosyası Oluşturma

DataStage'in çalışma konfigürasyonunu değiştirmek için özel Apt yapılandırma dosyaları oluşturulabilir.

### Apt Yapılandırma Dosyası Nedir:

- Node sayısı, konumları, sunucu bilgileri, veri seti yolları gibi bilgileri içerir
- İşlerin nasıl bölümleneceğini ve çalışacağını belirler

### Özel Apt Yapılandırma Dosyası Oluşturma:

1. Tools > Configurations menüsüne gidin
2. "New Configuration" seçeneğini seçin
3. Node sayısı ve diğer parametreleri ayarlayın
4. "Save Configuration As" ile yeni yapılandırmayı kaydedin
5. "Check" ile yapılandırmanın doğruluğunu kontrol edin

### Kullanım:

- Çalışma zamanında Apt yapılandırma dosya değerini değiştirerek, işlerin farklı bölümleme konfigürasyonlarıyla çalışmasını sağlayabilirsiniz
- Örnek: 2 node yerine 3 node yapılandırmasıyla çalıştırma

## Diğer Gelişmiş Özellikler

### Message Handler (Mesaj İşleyici)

- Belirli uyarı ve hata mesajlarını yönetmek için kullanılır
- DataStage Director'da olduğu gibi, Designer'dan da erişilebilir
- Belirli mesajları gizlemek veya seviyesini değiştirmek için kullanılabilir

### Custom Stages (Özel Aşamalar)

- DataStage'de özel aşamalar oluşturulabilir (yaygın kullanılmaz)
- Özel gereksinimlere göre tasarlanabilir

### Tools Menüsü Ek Seçenekleri

- Run Director: Designer'dan Director'ı açar
- Multiple Job Compile: Birden fazla işi derler
- Configurations: Apt yapılandırma dosyalarını yönetir

Bu gelişmiş özellikler, özellikle büyük DataStage projelerinde, sürüm yükseltmelerinde ve karmaşık ortamlarda çalışırken oldukça faydalı olabilir. Normal günlük işlerde çok sık kullanılmasalar da, belirli durumlarda problem çözme ve verimlilik sağlama açısından değerlidirler.



