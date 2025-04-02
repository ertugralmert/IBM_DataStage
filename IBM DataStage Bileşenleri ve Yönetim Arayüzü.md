# IBM Information Server Suite (DataStage) Bileşenleri ve Yönetim Arayüzü

# IBM Information Server Suite Bileşenleri

4 temel yazılımı inceleyelim

- Designer
- Director
- Administrator
- Web Console

Diğer bileşenler MultiClient Manager, Metadaa Manager vb. ileride bakacağım.

## Web Console Üzerinden Yönetim

DataStage kurulumu tamamlandıktan sonra bir yönetici olarak yapılması gereken ilk adımlar Web Console Üzerinden gerçekleştirilir.

- Web Console'da üç ana bölüm vardır. Home, Administrator ve Reporting

### Kullanıcı ve Grup Yönetimi

1. Kullanıcı Oluşturma:
Users bölümüne girilir NEw user seçilerek adım tamamlanır. Suit rolü otomatik olarak tanır.
2. **Grup Oluşturma:**
Groups böümünde new Group seçeneği ile ilerlenir. Gruba roller atanır örneğin Suit Administrator, Suit User, DataStage izinleri
3. **Kullanıcıları Gruplara Atama:**
Oluşturulan gruplar içinde "Browse" seçeneği ile kullanıcılar eklenir. Kullanıcılar atandıkları grupların rollerini otomatik olarak miras alır.

### Engine Credentials - Motor Kimlik Bilgileri

1. **Engine ve Server Tier Bağlantısı**
DataStage mimarisi Service Tier ve Engine Tier içerir. Kullanıcılar Web Console'da oluşturulduğunda, Service Tier'da tanımlanır ancak Engine Tier'a erişim için ek yapılandırma gerekir.
2. **Engine Credentials Ayarlama**
Domain Management bölümüne gidilir.
Engine Credentials seçeneği seçilir. Kullanıcıya Engine Tier'ın çalıştığı işletim sistemi kimlik bilgileri atanır. Bu ayar olmadan, kullanıcılar DataStage Designer veya Director'a giriş yapamaz.

### Session Management - Oturum Yönetimi

1. **Aktif Oturumları Görüntüleme**
Active Sessions bölümünde sisteme bağlı tüm kullanıcılar listelenir.
Kullanıcı adı, oturum ID'si bağlantı tipi(Web Console, Designer, Administator) görüntülenir.
2. **Oturum Sonlandırma**
Sorun yaşayan kullanıcıların oturumlarını sonlandırmak için kullanılır. Bir kullanıcı Citrix veya başka bir nedenle oturum kapanması yaşarsa yönetici burada oturumu kapatabilir. Bu işlem kullanıcının yeniden giriş yapabilmesini sağlar.

### Log Management

1. **Logging Components - Günlük Bileşenleri**
DataStage, ISF Agent, Security Service gibi farklı bileşenlerin günlüklerini yönetebilir.
Hangi olayınların kaydelieceği ve hangi detay seviyesinde kaydeliği ayarlanabilir.
2. **Log Configurations - Günlük Yapılandırmaları**
Authentication, Job Execution(İş Çalıştırma) gibi farklı işlem türleri için günlük seviyesi ayarlanabilir. SSeviyeler Information, Warning, Error, Fatal, All şeklinde.
İş çalıştırma günlükleri için genellikle "All" seviyesi seçilir.

### Schedule Monitoring

DataStage'de yerleşik zamanla özelliği şu anda tam olarak desteklenmemektedir. Gelecek sürümlerde bu özellik geliştirirebilir. Kurumsal ortamlarda genellikle Informatica gibi farklı ETL araçlarıyla entegrasyon gerekir ve bu tür karmaşık iş akışlarını düzenlemek için harici zamanlama araçları kullanılır.

### Kullanıcı Rolleri Arasındaki Farklar

- Administrator: Tüm yönetim özelliklerine erişebilir, kullanıcı ve grup oluşturabilir. Engine'ler arası kimlik bilgilerini ayarlayabilir.
- User : Sınırlı erişime sahiptir, sadece engine credentials, log, views ve schedule monitoring gibi belirli bölümleri görebilir.