# DataStage Yönetici İstemcisi (Administrator Client) ve Proje Yönetimi

# DataStage Administrator Client

DataStage kurulduktan ve Web Console bileşeni tamanlandıktan sonra, Administrator Client'a geçeriz. Burada farklı seçeneklerin ayarlanması, projelerin oluşturulması ve yönetilmesi gibi işlemler gerçekleştirilir.

Administrator Client'a giriş yaparken:

- Host name alanına service tiear bilgilerini gireriz.
- username ve password ile giriş yaparız
- Inactivicity timeout genellikle 24 saat olarak ayarlanmıştır. Yöneticiler için
- Geliştiriciler için Designer veya Director'da bu süre genellikle 30 dakikadır.

## Proje Yönetimi

Administrator Client'da yapabileceklerimiz.

- Mevuct projeleri görüntüleme
- Yeni Projeler oluşturma
- Proje silme

Yeni proje oluşturmak için;

1. Project sekmesinde "New Project" 
2. Proje adı ve detayları girilir.
3. Mevcut bir projeden rolleri kopyalama seçeneği sunulur
4. Proje oluşturma işlemi birkaç dakika sürebilir.

## Proje Özellikleri Yapılandırması

Proje oluşturduktan sonra "Properties" sekmesinden yaparız.

1. **Enable Job Administrator in Director**
Bu seçenek bazı yönetim aktivileterinin Director üzeirnden yapılabilmesini sağlar. Yöneticilerin her işlem için Administrator'a giriş yapmasına gerek kalmaz. Ancak geliştiriciler yanlışlıkla projeye zarar verebilecek değişiklikler yapabilirler
Genellikle bu seçenek işaretlenmez
2. **Enable Runtime Column Propagation for Parallel Job**
Runtime Column Propagation (RCP) DataStage'de önemli bir özelliklitir.
Benzer yapıdaki işler için tek bir job tasarlamamıza olanak tanır. örnek: birden fazla tablodan veri çekip farklı hedeflere yükleme, iki farklı tablodaki verileri karşılaştırma, eski betiklerin(script) DataStage'e taşınması ve test edilmesi
Genellikle "generic job" oluşturmak için kullanılır.
Administarator'da etkinleştirilmezse, Designer'da kullanılamaz
Administratorde etkinleştirilirse, Developer isterse job seviyesinde devre dışı bırakabilir.
3. **Enable Editin of Internal References in Job**
Bu seçenek bir DataStage job'undan başka bir job'u çağrıma veya shared container kullanımı gibi durumlarda referansların düzenlenebilmesini sağlar.
Genellikle karışıklara neden olabileceği için sçeilmez.
Shared Container gibi generic senaryolarda yanlışlıkla değişiklik yapılmasını önlemek için kapatılır.

## Internal References - İç Referanslar

Bir işin başka bir işi çağırabilme özelliğidir. Bu özellik genellikle devre dışı bırakılır çünkü:
* Bir iş içinden başka bir işi açıp değiştirme imkanı verir
* Generic jobs veya shared containers yanlışlıkla değişitirebilir.

## Metedata Importu ve Paylaşımı

"Metedata when importing connectors" seçeneği, connector stageler arasında metadata paylaşımını sağlar. İlk stage'de tanımlanan sütun bilgileri, sonraki stage'lere otomatik olarak aktarılır. Böylece her stage de tekrar metadata tanımlaması yapmaya gerek kalmaz. Bu seçenek genellikle aktif bırakılır

## Operational Metadata Oluşturma

"Generate operational metadata" seçeneği, iş çalışırken her aşamadaki sütun bilgilerini kaydeder.Genellikle işlerin genel performasını etkilediği için devre dışı bırakılır. Ancak karmaşık veri eşleştirmelerinde hata ayıklama için çok faydalıdır. Developerlar bunu ortam env. variables ile geçici olarak etkinleştirebilir. Sorun çözüldükten sonra devre dışı brakılabilir.

## Permissions

Projeye kullanıcı ve group erişimi sağlar. Bir projeyi oluştururken mevcut bir projeden roller kopyalanabilir. En iyi uygulama, Web Console'dan gruplar oluşturup bu gruplara roller atamaktır.

## Tracing

Server-side tracing genellikle devre dışı bırakılır. Etkinleştirildiğinde sistem düzeyinde ayrıntılı günlükler oluşturur. Sunucuya ekstra yük getirdiği için sadece karmaşık hata durumlarında kullanılır.

Örneğin bir kullanıcının giriş yapamaması ve nedenini bulunamaması durumunda

## Schedule

Zamanlanmış işlerin çalıştırılması için kimlik bilgilerinin tanımlandığı bölümdür. Zamanlanmış işler, belirtilen kullanıcı adı ve şifre ile çalıştırılır. Bu geliştirici hesaplarından farklı olabilir. Güvenlik amacıyla geliştiricelerin engine server'da doğrudan erişimi kısıtlanabilir.

## Tunables - Ayarlanabilir Parametre

Genellikle eski Server Jobs için performans ayarları içerir. Parallel jobs için çoğunlukla gerekli değildir.
"Enable row buffer" - Server Job'larda satır tamponlamayı etkinleştirir.
"Read cache size" ve "Write cache size" - Hash File stage için önbellek boyutları(128MB varsayılan değer)

## Parallel Seçenekleri

"Enable grid support" Grid topolojisinde çalışan DataStage kurulumları içindir.

"Generate OSH visible" - OSH scriplerini görüntülemeyi sağlar. OSH, iş çalıştırıldığında arka planda oluşturulan scriptlerdir. Hata ayıklama ve performans analizi içi çok faydalıdır. Genellikle etkinleştirilir.

## Varsayılan Formatlar

Tarih, zaman, zaman damgası ve ondalık ayırıcı için varsayılan formatlar tanımlanabilir.

Varsayılan olarak tarihler "yyyy-mm-dd" formatında beklenir

"%" işareti biçimlendirme tanımlayıcı olarak kullanılır ve değiştirilmemelidir.

## Remote Seçenekleri

Eski bir özellik, günümüzde kullanılmaz

Eskiden administrator sadece sunucuda bulunur ve uzaktan yönetim gerekiyordu. Modern DataStage kurulumlarında, client kurulumlar kendi administrator bileşenlerini içerir.

## Log Seçenekleri

"Auto purge of job log" eski günlüklerin otomatik temizlenmesiini sağlar. Production ortamında çok kullanışlıdır. Belirli bir gün sayısından sonra veya belirli sayıda çalışmadan sonra günlükler silinebilir.

"Generate operational metadata repository" Max günlük mesaj sayısını belirler. varsayılan olarak 10 uyarı ve 10 bilgi mesajı kaydedilir. Bu sayı aşılırsa artık mesaj kaydedilmez.

## Sequence ( Sıralı ) Seçenekleri

Administrator'da ayarlandığında, tüm sequencer işleri için varsayılan olarak uygulanır. Developer, iş seviyesinde bu ayarları değiştirebilir.(iş seviyesindeki ayarlar önceliklidir). Sequencer konusunda detaylı anlatacğaım

## Environment Variables

DataStage'de çok sayıda ortam değişkeni bulunur. Tümünü bilmek gerekli değildir. 

Sadee en önemli ve sık kullanılanları açıklayacağım. IBM document sayfasında tüm env. variables vardır.

# DataStage'de Ortam Değişkenleri ve Parametre Setleri

## Ortam Değişkenleri - Environment Variables

Ortam değişkenleri, DataStage projesindeki tüm işlerde kullanılabilen, yönetici tarafından tanımlanan değişkenlerdir.

Genel Özellikler :

- Yönetici tarafından tanımlanır ve ayarlanır.
- Geliştiriciler yönetici arabiriminde bunları değiştiremez
- İşler içine çekilip çalışma anında (runtime) değerleri değiştirilebilir.
- Değişkenlerin isimlendirmesinde sadece alfanümerik karakterler ve alt çizgi (_) kullanılabilir.
- Proje düzeyince tanımlanır ve tüm işlerde kullanılabilir.

## Ortam Değişkenlerinin Kategorileri

1. General
* PATH: Shell arama yolu
* TMP_DIR : Geçici dosyaların saklanacağı dizin
* Bu dizinler hem sistem tarafından hem de geliştiriceler tarafından kullanılabilir.
* TMP_DIR, ara sonuçları ve test dosyalarını saklamak için kullanılır. 
2. User Defined (Kullanıcı Tanımlı)
* Yönetici tarafından özel olarak oluşturulan değişkenler
* Veritabanı bağlantı bilgileri , dosya yolları, kullanıcı adları vb. için kullanılır.
* Örnek : SOURCE_FILE_PATH, DB2_SERVER, ORACLE_USER
* Developerların her işte tekar tekrar aynı bilgileri girmesini önler.
* Güvenlik sağlar çünkü bu bilgiler tek bir kaynakta kontrol edilir.
3. Parallel Değişkenler
* Paralel işlerin çalışma davranışını kontrol eden değişkenler
* APT_BUFFERING_POLICY: Önbelleğe alma davranışını kontrol eder.
        * Automatic: Varsayılan, DataStage uygun gördüğü yerde önbelleğe alır .
        * Force: Tüm stager'lerde önbelle alma zorlanır. Join gibi bazı stagelerde çalışmaz
        * None: Önbelleğe alma devre dışı bırakılır. Parallel job'u Sequential job gibi davranır.
* APT_DISABLED_COMBINATION: Stage birleştirme özelliğini kontrol eder.
     *False (varsayılan): Stage birleştirme etkin, DataStage benzer stageleri otomatik birleştirir.
    * True: Stage birleştirmeyi devre dışı bırakır, hata ayıklama için kullanılır.
    * Otomatik birleştirme, kötü tasarlanmış işlerin performansını artırır.
    * Birkeltirme sırasında stage isimleri "APT_transform_combined" gibi değişir
* APT_CONFIG_FILE: İşin çalışacağı node yapılandırmasını belirler.
    * Hangi node'ların kullanılacağını ve özelliklerini tanımlar
    * Node sayısı, server ismi, resource disk, scratch disk bilgilerini içerir.
* APT_NO_PART_INSERTION: Otomatik bölümleme stage eklemeyi kontrol eder.
    * False(Varsayılan): DataStage, gerekli yerlere otomatik bölümleme stageleri ekle
    * True: Otomatik bölümleme devre dışı bırakılır, geliştirici kendi bölümlemeyi sağlar
* APT_NO_SORT_INSERTION: Otomatik sıralama stage eklemeyi kontrol eder.
    * False(Varsayılan): DataStage, gerekli yerlere otomatik sıralama stageleri ekler.
    * True: Otomatik sıralama devre dışı bırakılır.
    * Join veya Remove Duplicates gibi stageler için sıralama ön koşuldur.
* APT_NO_PART_SORT_OPTIMIZATION: Bölümleme ve sıralama optimizasyonunu kontrol eder.
    * False(Varsayılan): DataStage, bölümleme ve sıralama stratejelerini optimize eder.
   * True: Optimizasyon devre dışı bırakılır, geliştirici kendi stratejisini kullanır.
4. Reporting Değişkenleri
* OSH_PRINT_SCHEME: Metadata yapısını Director loglarında gösterir.
     * False(Varsayılan): Metadata bilgisi loglanmaz
     * True: Her stage'in giriş ve çıkış metadatası loglanır.
* APT_DUMP_SCORE: İş performans istatistiklerini gösterir.
    * False(Varsayılan): Performans istatistikleri loglanmaz
    * True: Her stage'in hangi partitioning'de çalıştığı ve her partitioning deki kayıt sayısı loglanır.
   * Perdormans analizi ve optimizasyon için çok değerlidir.
APT_RECORD_COUNT: Her stage'deki kayıt sayısını loglar.
    * False(Varsayılan): Record sayısı loglanmaz
    * True: Her stage'deki toplam kayıt sayısı loglanır. (parti bazında değil)
5. Operator Specific Değişkenler
* Belirli stagere özgü değişkenler(DB2,Oracle,SAS,ODBC vb.)
* Bu değişkenler genellikle hazır olarak optimize edilmiştir.
* Değiştirilmeleri önerilmez, ancak gerekirse trial-and-error yöntemiyle değiştirilebilir.
6. Compiler Değişkenler ( Derleyici )
* Kesinlikle değiştirilmemesi gereken değişkenler
* Değişiklik yapılması, işlerin derlenmesini ve çalışmasını engelleyebilir.

## Ortam Değişkenlerini Dışa/içe Aktarma (Export - Import)

- Export:  Ortam değişkenleri bir dosyaya aktarılabilir. Tüm değişkenler veya sadece seçilen kategoriler dışa aktarılabilir. Bu dosya test ve production ortamlarına taşınabilir.
- Import: Değişkenler bir dosyadan içe aktarılabilir. Elle yeniden girmek zorunda kalmadan değişkenleryüklenebilir. İsim, tip, prompt ve değeler korunur. Sıralama değişebilir ama önemli değildir.

## Parametre Setleri (Parameter Sets)

Benzer amaçlar için kullanılan parametrelerin gruplandırılmış halidir.

Temel Özellikler: 

- Geliştiriciler tarafından oluşturulabilir. 
- İlgili parametreleri mantıksal olarak gruplar örn: DB' bağlandı parametreleri
- Bir kez oluşturulup birçok işte kullanılabilir.
- Her işte tekrar tekrar parametreleri tanımlamak gerekmez.
- Standart isimlendir sağlar, bu da otomatik çalıştırmayı kolaylaştırır.

#### Parametre Seti Örneği

```bash
DB2_CONNECTION:
  - DB2_SERVER: "server_name"
  - DB2_USER: "username"
  - DB2_PASSWORD: "encrypted_password"

```

#### Parametre Seti Oluşturma

1. File → New → Parameter Set yolunu izleyin
2. İsim verin örn: DB2_CONNECTION
3. Parametreleri tanımlayın server,user,password vb.
4. Değerleri atayın ve şifreleri "Encrypted" olarak işaretleyin
5. Bir klasöre kaydedin örn: Parameter Sets Klasörü
6. İsteğe bağlı olarak farklı ortamlar için değeler tanımlayın DEV,TEST,PROD

#### Bir  Job'ta Parametre Setlerini Kullanma

1. Job özelliklerindeki "Job Properties" Parameters sekmesine gidin
2. Üçüncü butonu (Pull Parameter Set) kullanarak seti seçin
3. Parametre seti iş parametreleri listesine eklenir
4. Parametre setinin içindeki değerler, ilgili stagelerde kullanılabilir.

#### Parametre Setlerinin Avantajları

1. Zaman Tasarrufu: Her işte aynı parametreleri tekrar tekrar girmek gerekmez
2. Tutarlılık: Tüm işlerde aynı parametre isimleri kullanılır.
3. Bakım Kolaylığı: Değişikik yapmak gerek tiğinde tek bir yerde yapılır.
4. O Çoklu Ortam Desteği: DEV;PROD, TEST gibi farklı ortamlar için değerler tanımlanabilir.

#### Önemli Notlar

- Parametre setleri silinebilir, bu nedenle dikkatli olunmalı
- Developerlar yeni parametre setleri oluşturabilir.
- İş içinde farklı değerler kullanılabilir.



## Değişken türlerini Ayırt Etme

1. İş içinde görünümleri:
- Ortam Değişkenleri: $ işareti ile başlar örn: $APT_CONFIG_FILE
- Parametre Setleri: Normal isimle görünür örn : DB2_CONNECTION
- Normal Parametreler: Normal isimle görünür. örn: filename
2. Type sütunu
- Ortam Değişkenleri : String, Integer vb.
- Parametre setleri: "Parameter Set" olarak gösterilir
- Normal Parametreler: String, Integer vb.
3. İçerik görüntüleme
- Parametre setlerinin içeriği görüntülenebilir. (ikinci buton)
- Normal parametrelerin içeriği yoktur.

# Önemli Kullanım Örnek Senaryolar

1. APT_DISABLED_COMBINATION Kullanımı:
Sorun giderme sırasında true yapılarak stagelerin kombinasyonu engellenir.
Hata hangi stage'den geliyor belirlenebilir. Sorun çözüldükten sonra tekrar false yapılarak performans artırılır.
2. APT_DUMP_SCORE Kullanımı:
Performans sorunlarını tespit etmek için kullanılır. örneğin gender örneğinde 4 node yapılandırmasında sadece 2 node kullanıyorsa bunu gösterir. Bunun sonucunda node sayısı azaltılarak performans artırılaiblir.
3. Protect Project Özelliği:
Test ve Prod ortamlarında kullanılır. Yanlışlıkla job silmeyi engeller. Geliştirme ortamında genellikle kullanılmaz.