# DataStage Intro



# IBM DataStage: Topolojiler, Mimari ve Performans Stratejileri

---

## Giriş: IBM DataStage Nedir?

IBM DataStage, kurumsal veri entegrasyonu (ETL - Extract, Transform, Load) platformudur. IBM Information Server ailesinin bir parçası olarak büyük veri hacimlerini işlemek, dönüştürmek ve farklı sistemler arasında taşımak için kullanılır. DataStage, çeşitli kaynaklardan veri çekme, bu verileri iş ihtiyaçlarına göre dönüştürme ve hedef sistemlere yükleme süreçlerini otomatikleştirir.

### IBM InfoSphere Bileşenleri

InfoSphere, veri yönetimi, entegrasyonu ve analizi için geliştirilen IBM çözümler ailesidir. Temel bileşenleri:

- InfoSphere Blueprint Director
- Business Glossary
- Business Glossary Anywhere
- Data Architect
- DataStage
- Discovery
- FastTrack
- Information Analyzer
- Information Service Director
- Metadata Workbench
- QualityStage

---

## DataStage Mimarisi ve Temel Bileşenler

DataStage mimarisi, veri entegrasyonu süreçlerini verimli bir şekilde yönetmek üzere tasarlanmış çeşitli katmanlardan oluşur:

1. **Client Tier (İstemci Katmanı)**: Kullanıcı arayüzleri ve tasarım araçlarını içerir
    - Designer Client (Tasarımcı İstemcisi)
    - Administrator Client (Yönetici İstemcisi)
    - Director Client (Direktör İstemcisi)
1. **Services Tier (Hizmetler Katmanı)**: Ortak ve ürüne özel hizmetleri barındırır
    - Common Services (Ortak Hizmetler)
    - Product-specific Services (Ürüne Özel Hizmetler)
    - Application Server (Uygulama Sunucusu)
1. **Engine Tier (Motor Katmanı)**: Veri işleme motorlarını içerir
    - IBM Information Server Engine
    - Connectors (Bağlayıcılar)
    - InfoSphere QualityStage modules (InfoSphere QualityStage modülleri)
    - Service Agents (Hizmet Ajanları)
1. **Metadata Repository Tier (Metadata Deposu Katmanı)**: Tüm metadata bilgilerinin saklandığı veritabanını içerir

### DataStage Motorları

DataStage mimarisinde iki temel motor bulunur:

1. **Parallel Engine (Paralel Motor)**:
    - Partitioning ve pipelining gibi özellikler sayesinde performansı yüksektir
    - Orchestrate(OSH) Compiler'ı kullanır
    - Dönüşümler için C++ entegrasyonu mümkündür (Transformer aşaması)
    - 30-40'tan fazla stage mevcuttur, karmaşık gereksinimleri kolayca karşılar
    - En yaygın kullanılan iş türüdür
1. **Server Engine (Sunucu Motoru)**:
    - Sıralı (Sequential) çalışır, performans nispeten yavaştır
    - Basic Compiler adı verilen derleyici kullanır
    - Sınırlı sayıda stage içerir (8-10 kadar)
    - Geniş ölçekli verilerde veya karmaşık dönüşümlerde tercih edilmez

---

## DataStage'deki İş Tipleri

DataStage'de üç temel iş tipi bulunur:

1. **Server Job (Sunucu İşi)**:
    - Server Engine kullanır
    - Sıralı (Sequential) çalışır, performans nispeten yavaştır
    - Basic Compiler adı verilen derleyici kullanır
    - Sınırlı sayıda stage içerir (8-10 kadar)
    - Geniş ölçekli verilerde veya karmaşık dönüşümlerde tercih edilmez
1. **Parallel Job (Paralel İş)**:
    - Parallel Engine kullanılır
    - Partitioning ve pipelining gibi özellikler sayesinde performansı yüksektir
    - Orchestrate(OSH) Compiler'ı kullanır
    - Dönüşümler için C++ entegrasyonu mümkündür (Transformer aşaması)
    - 30-40'tan fazla stage mevcuttur, karmaşık gereksinimleri kolayca karşılar
    - En yaygın kullanılan iş türüdür
1. **Sequencer Job (Sıralayıcı İş)**:
    - Bir nevi kontrol yapısı (control framework) gibidir
    - Hem Server hem Parallel işleri sırasıyla tetikleyebilir veya aralarında bağımlılıklar (dependency) yaratabilir
    - Sıralı çalışır ve Server Engine kullanır
    - Stage'leri "Job Activity" veya "Routine Activity" gibi, yönetim/koordinasyon amaçlıdır. Veri dönüştürme işlemleri barındırmaz
    - Örneğin "Once şu job tamamlansın, sonra bu job başlasın" gibi iş akışlarını tanımlamakta kullanılır

### DataStage'deki Önemli Notlar

- DataStage, Transformer aşamalarının arka planında C++ kodu çalıştırır. Filtreler, tip dönüşümleri karmaşık hesaplamalar bu şekilde yapılabilir.
- Özel fonksiyonlar yazmak için "Server Routines" ve "Parallel Routines" kullanılır. Ancak Parallel Routines genelde yaygın değildir. Çünkü hemen her şey zaten Parallel Job aşamalarıyla çözülebilir.

---

## Dağıtım Topolojileri

DataStage dört farklı topolojik yapıda dağıtılabilir. Her topoloji, farklı kullanım senaryolarına ve ölçekleme ihtiyaçlarına uygun olarak tasarlanmıştır.

### Two Tier Topoloji

Two Tier (İki Katmanlı) Topoloji, Client Tier ile Engine service ve metadata repository katmanlarının iki farklı yerde kurulduğu yapıdır.

**Özellikler**:

- Client katmanı tek tarafta kurulurken, diğer üç yapı (metadata repository, engine tier ve service tier) başka bir sunucuda bulunur.
- Engine tier, metadata repository ve service tier birleştirilir ve tek bir sunucuda kurulur.
- Veri entegrasyonunun ilk zamanlarında kullanılan bir yöntemdir.
- Az kullanıcı ve veri olduğu dönemler için uygundur.
- Tek sunucu tüm işlemlere cevap verir, ancak veri ve kullanıcı sayısı arttıkça performans düşmeye başlar.

### Three Tier Topoloji

Performans sorunu yaşanmaya başlandığında geliştirilmiş bir yaklaşımdır. Engine tier, service tier ve metadata repository'den ayrılmıştır.

**Özellikler**:

- Client Tier bir veya birden fazla sunucuda bulunur
- Engine tier tek bir sunucuda konumlandırılır
- Service tier ve metadata repository ayrı bir sunucuda bulunur
- Bu iki sunucu birbirlerinin IP adreslerini bilir ve sürekli iletişim halindedir
- Kullanıcı sayısı ve veri hacmi arttıkça bu topoloji de performans sorunlarına yol açabilir

### Cluster Topoloji

Three Tier topolojiye çok benzer bir yapıdadır. Temel fark, birden fazla engine tier kullanılmasıdır.

**Özellikler**:

- Birden fazla engine sunucusu kullanılır, böylece daha fazla işlem gücü sağlanır
- En büyük sorun, belirli bir iş veya projenin hangi engine tier tarafından yönetileceğidir
- Bu sorunu çözmek için "apt configuration file" adı verilen bir yapılandırma dosyası kullanılır
- Dört veya daha fazla engine tier içeren bir yapıda bir engine devre dışı kaldığında manuel müdahale gerekir
- Bu manuel müdahale gereksinimi, Grid topolojisinin geliştirilmesine yol açmıştır

### Grid Topoloji

Cluster topolojisinin manuel müdahale gerektiren kısıtlamaları nedeniyle geliştirilmiş en gelişmiş topolojidir.

**Özellikler**:

- "Resource Manager" ve "Workload Manager" bileşenlerini içerir
- Resource Manager, manuel olarak oluşturulan apt config dosyasını dinamik olarak oluşturur
- Birden fazla engine tier vardır, ancak gelen tüm işleri yürütmeden sorumlu, Resource Manager'ın bulunduğu tek bir sunucu bulunur
- Resource Manager şu görevleri yerine getirir:
    - Diğer sunucularla sürekli iletişim halindedir
    - Mevcut sunucuların görevlerini, işlerini ve kaynaklarını takip eder
    - Her erişilebilir sunucu hakkında bilgi içeren bir veritabanı tutar
    - İşlerin yürütülmesi için gereken sunucuları belirler
    - Kaynak atamasını ve iş dağılımını dinamik olarak yönetir
    - Slave düğümlerle düzenli "heartbeat" mesajları ile iletişimde kalır
- Grid topolojisinin en büyük avantajı, yüksek performanslı birkaç sunucu ve çok sayıda standart bilgisayar (commodity hardware) kullanılabilmesidir
- Bu yaklaşım, maliyeti düşürürken ölçeklenebilirliği artırır
- DataStage, Quality Stage ve Information Analyzer gibi büyük veri kullanan bileşenler için idealdir

#### Çevre Değişkenleri (Environment Variables)

Grid topolojisinin çalışması için kullanılan çevre değişkenleri (versiyon 9.1 ve üstünde):

- **Apt Grid Enabled**: İşlerin Grid topolojide mi yoksa normal Cluster topolojide mi yürütüleceğini belirtir
- **Grid Queue**: İşlerin sıraya alındığı yerdir. Tüm kaynaklar veya sunucular kullanılıyorsa, sonraki işler bu sırada bekler
- **Apt Grid Partitions**: Her hesaplama düğümünde kaç bölüm olacağını belirtir (bir sunucuda bu işin kaç örneğinin yürütülebileceği)
- **Apt Grid Compute Nodes**: İşin kaç farklı sunucuda yürütülmesi gerektiğini belirtir

Bu değerler yönetici tarafından belirlenir ve yürütme sırasında değiştirilmemesi önerilir.

---

## Bölümleme (Partitioning) ve Paralellik (Parallelism)

### Veri Bölümleme Stratejisi

DataStage'de bölümleme, kodun değil verilerin bölünmesi anlamına gelir, bu çok önemli bir farktır:

- Hadoop'un MapReduce yaklaşımında olduğu gibi kodun bölünmesi söz konusu değildir
- DataStage'de komut dosyası (script) bütün olarak kalır
- Bölünen şey, üzerinde çalışılan veridir

**Örnek**: 1 milyon kayıt işlenecekse, bölümleme yaklaşımında bu kayıtlar tek bir sunucuya çekilip işlenmez. Bunun yerine, veriler apt yapılandırma dosyası tarafından belirlenen sayıda bölüme ayrılır ve paralel işlenir.

**Günlük Hayattan Benzetme**:

- Evinizde yemek pişirme, temizlik, bulaşık ve çamaşır yıkama gibi dört farklı ev işini tek başınıza yapmak için 4 saat harcadığınızı düşünün.
- Size yardım edecek bir kişi daha eklenirse ve iş bölümü yaparsanız, aynı işleri 2 saatte bitirebilirsiniz.
- Bu, görevlerin bölünmesi (partitioning) ve aynı anda yürütülmesi (parallelism) anlamına gelir.

**DataStage'de Terminoloji**: DataStage'de bölümleme (partitioning), düğüm (node) ve sunucu (server) terimleri genellikle aynı anlamda kullanılır:

- "İş dört bölümde yürütülüyor"
- "İş dört düğümde yürütülüyor"
- "İş dört sunucuda yürütülüyor"

Hepsi aynı anlama gelir. Veri bölünerek farklı sunucularda aynı kodla işlenir.

### APT (Advanced Parallel Technology)

APT, DataStage'in paralel işleme altyapısını ifade eder:

**APT Yapılandırma Dosyası (Apt Configuration File)**:

- Hangi sunucuların (nodes) kullanılacağını belirtir
- Her sunucunun IP adresi veya adını içerir
- Özel işlemlerin hangi sunucularda gerçekleştirileceğini tanımlayabilir
- Veri kümeleri ve geçici dosyaların nerede saklanacağını belirtir

**APT Yapılandırma Dosyası Örnek İçeriği**:

- **Node**: Bölüm/düğüm numarası (node one, node two, vs.)
- **Fastname**: Sunucunun adı veya IP adresi
- **Pools**: Bu sunucuda hangi işlemlerin yürütüleceği
- **ResourceDisk**: Veri kümelerinin oluşturulacağı konum
- **ScratchDisk**: Geçici dosyaların saklanacağı konum

Grid topolojisinde bu dosya Resource Manager tarafından otomatik oluşturulurken, Cluster topolojisinde manuel olarak yapılandırılır.

---

## Yürütme Stratejileri

DataStage, veri işleme performansını optimize etmek için çeşitli yürütme stratejileri kullanır.

### Sıralı (Sequential) Yürütme

Sıralı yürütmede, veriler bir aşamadan diğerine adım adım geçer:

1. İlk aşama tüm veriyi işler
2. Bu işlem tamamlandığında, çıktı bir sonraki aşamaya gönderilir
3. İkinci aşama tüm veriyi işler
4. Bu işlem tamamlandığında, çıktı bir sonraki aşamaya gönderilir
5. Bu süreç, tüm aşamalar tamamlanana kadar devam eder

**Dezavantajı**: Herhangi bir anda sadece bir aşama aktif olarak çalışır. Diğer aşamalar ya veri bekliyor ya da işlerini tamamlamış durumdadır.

### Boru Hattı (Pipelining) Yürütme

Boru hattı yaklaşımında, veriler daha küçük parçalara bölünür ve aşamalar eşzamanlı olarak çalışır:

1. İlk aşama, veri kümesinin ilk parçasını işler
2. Bu parçanın işlenmesi tamamlandığında, ilk parça ikinci aşamaya gönderilir
3. Aynı anda, ilk aşama veri kümesinin ikinci parçasını işlemeye başlar
4. Bu süreç devam ederek bir "veri akışı" oluşturur

**Günlük Hayattan Örnek**: Video izlerken gerçekleşen tamponlama (buffering) işlemi. Video akışı küçük parçalara bölünür, bir parça oynatılırken diğer parçalar arka planda indirilir.

**Avantajı**: Tüm aşamalar eşzamanlı olarak çalışır, böylece sistem kaynakları çok daha verimli kullanılır.

### Karma Stratejiler

Bölümleme ve boru hattı stratejileri birlikte kullanılabilir. Aşağıda bu stratejilerin performans karşılaştırması yer almaktadır:

Varsayalım ki:

- 4 eşit kapasiteli sunucu var
- Toplam 20.000 kayıt işlenecek
- Her aşama 5.000 kaydı 1 dakikada işleyebiliyor
- İş 5 farklı aşamadan oluşuyor: Database stage, Funnel, Filter, Sort, Target

**1. Sıralı Yürütme (Sequential Execution)**:

- Tüm 20.000 kayıt sırasıyla her aşamadan geçer
- Her aşama 5 dakika sürer
- Toplam süre: 25 dakika

**2. Boru Hattı (Pipelining) ile Yürütme**:

- 20.000 kayıt 5.000'lik 4 parçaya bölünür
- İlk 5.000 kayıt işlenirken diğer parçalar sırayla işlenir
- Toplam süre: 8 dakika

**3. Sadece Bölümleme (Partitioning) ile Yürütme**:

- 20.000 kayıt 4 sunucuya 5.000'er olarak dağıtılır
- Her sunucu kendi verisi üzerinde sıralı işlem yapar
- Toplam süre: 5 dakika

**4. Bölümleme ve Boru Hattı Birlikte**:

- 20.000 kayıt 4 sunucuya dağıtılır, ayrıca boru hattı yöntemi uygulanır
- Her sunucudaki 5.000 kayıt, daha küçük parçalara bölünür
- Toplam süre: 2 dakika

Görüldüğü gibi, sıralı yürütmeden (25 dakika) bölümleme ve boru hattının birlikte kullanıldığı modele (2 dakika) geçtiğimizde, aynı işin yürütme süresi 12.5 kat azalabilir.

---

## Çevre Değişkenleri ve Konfigürasyon

DataStage performansını ve davranışını etkileyen çeşitli çevre değişkenleri bulunur:

### Grid Topolojisi için Çevre Değişkenleri

- **APT_CONFIG**: Kullanılacak yapılandırma dosyasının yolunu belirtir
- **APT_GRID_ENABLED**: Grid topolojisinin kullanılıp kullanılmayacağını belirtir
- **APT_GRID_PARTITIONS**: Her hesaplama düğümünde kaç bölüm olacağını belirtir
- **APT_GRID_COMPUTE_NODES**: İşin kaç farklı sunucuda yürütüleceğini belirtir
- **APT_GRID_QUEUE**: İşlerin sıraya alındığı yerdir

### Bölümleme için Çevre Değişkenleri

- **APT_PARTITION_METHOD**: Bölümleme metodunu belirtir (hash, modulus, random, round-robin, same, entire vb.)
- **APT_DEFAULT_PARTITION_METHOD**: Varsayılan bölümleme metodunu belirtir
- **APT_FILE_BLOCKSIZE**: Dosya bloğu boyutunu belirtir

---

## Performans Optimizasyonu

DataStage uygulamasında performans optimizasyonu için bazı teknikler:

1. **Doğru Topoloji Seçimi**: İş yükünüze ve veri hacminize en uygun topolojiyi seçin
2. **Etkin Bölümleme**: Veri dağıtımını dengeli yapacak bölümleme yöntemlerini kullanın
3. **Boru Hattı Kullanımı**: İşlem adımları arasında veri akışını optimize edin
4. **Kaynak Yönetimi**: Resource Manager'ı etkin kullanarak kaynakları verimli dağıtın
5. **Paralel Stratejiler**: Karmaşık işlemleri paralel yürütecek stratejiler geliştirin
6. **Hardware Optimizasyonu**: Grid topolojisinde karma donanım altyapısı kullanın
7. **İş Tasarımı**: İşleri optimize edilmiş adımlarla tasarlayın
8. **Kaynak ve Hedef Sistemler**: Giriş/çıkış operasyonlarını optimize edin

---

## Özet ve En İyi Uygulamalar

### DataStage'in Temel Özellikleri

- Kod değil, veri bölümleme stratejisi
- Paralel işleme yeteneği
- Boru hattı veri akışı
- Çeşitli topoloji seçenekleri (Two Tier, Three Tier, Cluster, Grid)
- Dinamik kaynak tahsisi (Grid topolojisinde)
- Ölçeklenebilir mimari

### En İyi Uygulamalar

1. **Topoloji Seçimi**: Kurumsal ihtiyaçlara ve veri hacmine göre uygun topolojiyi seçin
2. **Bölümleme Stratejisi**: Veri karakteristiğine uygun bölümleme stratejileri kullanın
3. **Kaynak Tahsisi**: Grid topolojisinde Resource Manager'ı etkin yapılandırın
4. **İş Tasarımı**: Parallel iş tiplerini tercih edin ve karmaşık işlemleri optimize edin
5. **Rejim Yönetimi**: Büyük veri kümeleri için Grid topolojisini kullanın
6. **Zaman Kritik İşlemler**: Bölümleme ve boru hattı stratejilerini birlikte kullanın
7. **Donanım Kullanımı**: Grid topolojisinde karma donanım stratejisi uygulayın
8. **Monitoring**: Performans izleme ve iyileştirme stratejileri geliştirin

DataStage, doğru yapılandırıldığında ve uygun stratejilerle kullanıldığında, büyük veri entegrasyonu projelerinde çok yüksek performans sağlayabilen güçlü bir ETL aracıdır. Topoloji seçiminden bölümleme stratejilerine, iş tasarımından kaynak yönetimine kadar çeşitli parametreler, DataStage uygulamasının başarısını doğrudan etkiler.

![Image.png](DataStage%20Intro.assets/Image.png)





![Image.png](DataStage%20Intro.assets/Image%20(2).png)





![Image.png](DataStage%20Intro.assets/Image%20(3).png)





![Image.png](DataStage%20Intro.assets/Image%20(4).png)



![Image.png](DataStage%20Intro.assets/Image%20(5).png)

---

### Terimler ve Anlamları

1. Stage (Aşama)
DataStage iş akışında kullanılan yapı taşları, yani işlem birimleridir. Her stage belirli bir işlevi yerine getirir. veri okuma, dönüştürme, yazma v.b
örnek olarak: Sequential File Stage ( dosya okuma), Transformer Stage(Veri dönüştürme), Database Stage(veritabanı işlemleri)
2. Job - İş
Birden fazla stage'in bir araya gelerek oluşturduğu veri akışı sürecine denir. ETL süreçlerini gerçekleştirmek için kullanılır. Günlük satış verilerini toplama işi, müşteri veri temizleme işi...
3. Partitioning - Bölümleme
Büyük veri kümelerini daha küçük parçalara bölerek paralel işleme yapma tekniğidir. Performansı arttırmak için veri işleme işini birden fazla işlemciye dağıtır. 1 milyon satır veriyi 4 işlemciye 250'şer bin satır olarak dağıtmak gibi
4. Pipelining - İşlem Hattı
Bir stage'in çıktısının işlemi tamamlamadan diğer stage'e girdi olarak verilmesi tekniğidir. İşlem süresini kısaltır, veri akışını hızlandırır. İlk 1000 satır okunur okunmaz dönüşüm stage'ine gönderilmesi(transformer) tüm verilerin okunmasını beklemeden gibi.
5. Key Column - Anahtar Sütun
Verileri bölümlemek veya birleştirmek için kullanılan önemli sütunlardır. Hash, Modulus gibi bölümleme yöntemlerinde hangi kaydın hangi işlemciye gideceğini belirler. Örnek müşteri numarası, hesap numarası, TC kimlik numarası
6. Record - Kayıt
Veri akışındaki her bir satırdır. birden fazla alanı içerir. İşlenecek temel veri birimidir. Örneğin bir müşteri ait tüm bilgileri içeren satır.
7. Field/Column -Alan/Sütun
Bir kaydın içindeki her bir veri parçasıdır. Belirli bir veri türünü temsil eder, isim,yaş,fiyat v.b 
8. Lookup - Arama
Bir veri kümesindeki değerleri başka bir veri kümesinde arayarak eşleştirme işlemidir. Referans verilerini ana veri kümesiyle birleştirmek için kullanılır. Örneğin müşteri kodlarına göre müşteri isimlerini getirme, ürün kodlarına göre ürün detaylarını bulma
9. Reference Link - Referans Bağlantı
Lookup stage'inde arama yapılacak ikincil veri setidir. Ana veri akışındaki değerleri aramak için kullanılan tablo veya dosyadır. Örneğin ürün kodlarının detaylı açıklamalarını içeren bir tablo
10. Value Range - Değer Aralığı
Verileri belirli değer aralıklarına göre filtreleme veya bölümleme yöntemidir. Range partitionde hangi değer aralığının hangi işlemciye gideceğini belirler. örneğin 1-100 arası değeler 1.işlemciye, 101-200 arası değeler 2. işlemciye gider.
11. Join
İki farklı veri setini ortak anahtar sütünlar üzerinden birleştirme işlemidir. İlişkili verileri bir araya getirmek için kullanılır. örneğin satış tablosu ile müşteri tablsonunu müşteri ID'sine göre birleştirme
12. Aggregation - Toplama
Verileri belirli kriterlere göre gruplayıp özet değeler elde etme işlemidir. Toplu analiz yapmak, özet raporlar oluşturmak için kullanılır. Örneğin ürünlere göre toplam satış miktarını hesaplama, aylık ortalama geliri bulma
13. Transformer - Dönüştürücü
Verileri dönüştürmek için kullanılan özel bir stage türüdür. Veri tpi döünüşümleri, hesaplamalar, koşullu işlemler yapar. mrneğin tarih formatını değiştirme, fiyat hesalamaları, metinleri birleştirme
14. Sequntial File - Sıralı Dosya
Düz metin dosyaları CSV,TXT v.b ile işlem yapmak için kullanılan stage'dir. Metin dosyalarını okumak veya yazmak iiçin kullanılır. CSV dosyasından müşteri verilerini okuma, işlem sonuçlarını TXT dosyasına yazma
15. Parallel Processing - Paralel İşleme
Birden fazla işlemi aynı anda yürütme tekniğidir. İşlem süresii kıslatmak için çoklu işlemcileri kullanır. + işlemcide aynı anda veri işleme yaparak süreyi dörtte bire indirme diyebiliriz.
16. Reusable Sequence - Yeniden Kullanılabilir Dizi
Birden fazla job'da kullanılabilen stage gruplarıdır. Kod tekrarını öner, bakımı kolaylaştırır. Örneğin sık kullanılan dönüşüm işlemlerini tek bir pakette toplama
17. Metadata - Üstveri
Veri hakkındaki veridir. veri tanımları, yapıları, ilişkileri. Verinin yapsını ve özelliklerini tanımlar. Bir sütnunun veri tpi, uzunluğu, zorunlu olup olmadığı bilgileri gibi
18. Repository
DataStage nesnelerinin job,stage,sequnce v.b saklandığı merkezi veritabanınıdır. Projeleri ve bilenlerini organize eder, yönetir. Tüm ETL joblarının, dönüşümmantıklarının saklandığı yer
19. Constraint - Kısıtlama
Verilerin belirli kurallara uymasını sağlayan koşullardır. Veri kalitesini korumak için kullanılır. Sayısal bir alanın belirli bir aralıkta olmasını zorunlu kılma
20. Runtime Column Propagation RCP
Sütun bilgilerinin çalışma zamanında iletilmesi tekniğidir. Veri yapısı önceden bilinmediğinde dinamik olarak çalışmayı sağlar. Farklı yapılardaki dosyaları tek bir job ile işleyebilme.



---

# Partitioning & Pipelining Kavramları

#### Partitioning Algoritmaları (Bölümleme Algoritmaları)

DataStage'de iki ana gruba ayrılmış toplam dokuz partitioning algoritması bulunmaktadır.

## Keyed ve Keyless Partitioning Yöntemleri ( Anahtarı ve Anahtarsız Bölümleme)

- Keyed Partitioning (Anahtarlı Bölümleme): Bölümleme için bir key column (anahtar sütunu) gerektirir.
- Keyless Partitioning (Anahtarsız Bölümleme): Bölümleme için bir key column gerektirmez.

Mevcut Partitioning Algoritmaları:

1. Round Robin
2. Random
3. Range
4. Hash
5. DB2
6. Modulus
7. Same
8. Entire
9. Auto

İlk sekiz algoritma, keyed ve keyless partitioning yöntemlerine ayrılmıştır. Dokuncusu(Auto) ise hangi partitioning method'un kullanılacağını otomatik olarak belirleyen generic bir bölümleme yöntemidir.

### Keyed ve KEyless Partioning Arasındaki Farklar

1. Key Column Gereksinimi
* Keyless partitioning'de key column sağlamak gerekmez.
* Keyed partitioning'de key column zorunludur.
2. Record Dağılımı:
* Keyless partioning'de, her patition' daki veya server'daki record sayısı equal ve almost equal olur.
* Örneğin, 1000 recod ve 4 partition varas, her partition 250 record alır.
* 1001 record varsa,ekstra record dört server'dan birine gider, bu nedenle "almos equally divided" deriz.
3. Avantaj ve Dezavantajlar:
* Keyless partitioning yönteminin avantajı, her server'da process edilen record sayısının aynı olmasını garanti etmesidir.
* Dezavantaj ise, belirli bir key column'a sahip ilgili recordların aynı partition'da olacağını garanti edememesidir.

Örnek ile Açıklama:

Banking transaction örneği:

- Account number 1: $100 transaction
- Acoount number 2: $50 transaction
- Account number 1: $300 transaction
- Account number 3: $200 transaction

Keyless partitioning'de:

- Record'lar farklı server'lara dağıtılır. örneğin account number 1'in transaction'ları farklı server'lara gider.  Her account için total transaction amount'ı hesaplamak istediğimizde account number 1'in toplam işlemini($400) doğru hesaplayamayız çünkü transactionlar farklı serverlardadır.

Keyed partitioning'de:

- Aynı account number'a ait tüm transaction'lar aynı server'a gider. Account number 1'in tüm transactionları aynı server'da olacağından toplam transaction amount'u doğru hesaplayabiliriz.

Partitioning Algoritmalarının Detaylı İncelenmesi

1. Roun Robin Partitioning
* Keyless bir partitioning yöntemidir.
* Recorlar'ları sırayla server'lara dağıtır.
* First record server 1'e, second record server 2'ye, third record server 3'e...
* Cyclic olarak devam eder( fourth record tekrar server 1'e)
* Key Column'u dikkate almaz, sadece record sırasına göre dağıtır.
* Her serverdaki record sayısı equal veya almost equal'dır.
2. Random Partitioning
* Keyless bir partitioning yönetimidir.
* Record'ları random bir şekilde server'lara dağıtır.
* Belirli bir cyclic pattern içermez.
* Round Robin gibi, her server'daki record sayısının approximately equal olmasını sağlamaya çalışır.
3. Entire Partitioning
* Özellikle lookup stage'lerinde kullanılır.
* Tüm record'ları her server'a kopyalar
* Hiçbir gerçek partitioning yapmaz
* Reference link'te kullanılır.
* Master data ve reference data farklı partioning methodlarıyla bölünürse, lookup operation'ın doğru sonuç vermesini sağlar.
4. Same Partitioning
* Previous stage'de kullanılan partitioning method'u kullanır.
* Data'yı repartition yapmaz.
* Özellikle key partitioning sonrasında kullanıldığın faydalıdır.
* Aynı key column'lar üzerinde işlem yapıyorsak veriyi yeniden partition'lamaya gerek kalmaz.
5. Hash Partioning
* En yayın kullanılan key partitioning method'dur.
* Key column'lar üzerinde bir hashing algortithm çalıştırır
* Aynı key value'ya sahip tüm record'ların aynı server'a girmesini garanti eder.
* Multiple key column kullanabilir. (composite keys)
* Tüm data type'ları destekler(numeric, varchar,date vb..)
* Her server'daki record sayısının equal olacağını garanti etmez.
6. Modulues Partitioning
* Key partitioning method'udur.
* Sadece numeric key column'lar için çalışır.
* Sadece single key column kullanabilir.
* Key value'nun number of partitions'a bölümünden kalan değerine (remainder) göre record'ları dağıtır.
* Aynı key value'ya sahip recordların aynı server'a gitmesini garanti eder.
7. DB2 Partitioning
* DB2 database'indeki mevcut partitioned data'yı kullanır.
* Datayı DataStage'de repartition yapmak yerine, DB2'nin partition stucture'ını kullanır.
* Rarely used'dır. Çünkü database partitions genellik DataStage partitions ile compatible olmayabilir.
8. Range Partitioning
* Belirli bir value range'indeki recordları belirli partitionlara atar
* Lookup stage'de kullanılabilir.
* DataStage'de implementation'ı databaselerde olduğunda data challenging'dir.
9. Auto Partitioning
* Kullanılan stage'e bağlı olarak best partitioning method'u seçer.
* DataStage 8.7 ve sonraki versionlarda oldukça intelligentdır.
* Stage type ve used columnlara göre karar verir
* Çoğu durumda best choice yapar.

Önemli Notlar:

- Modern DataStage versionlarında 8.7+ genllikle Auto Partitioning yeterlidir.
- Eski versionlarda partitioning methodu explicity belirlememiz gerekiyordu
- Partitioning algorithm bilgisi, torubleshooting ve performance optimizasyonu için önemlidir.
- Özellikle aggregation, join veya lookup operations yapıyorsak, doğru partitioning methodu seçmek kritik önem taşır.
- Genel olarak Toplama(Aggegation) İşlemleri için Hash veya Modulus
- Basist veri işleleme için Round Robin veya Random
- Arama(Lookup) işlemleri için Entire
- Ardışık işlemler için Same kullanılabilir.



