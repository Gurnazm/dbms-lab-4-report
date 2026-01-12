# Deney Sonu Teslimatı

Sistem Programlama ve Veri Yapıları bakış açısıyla veri tabanlarındaki performansı öne çıkaran hususlar nelerdir?

Aşağıda kutucuk (checkbox) ile gösterilen maddelerden en az birini seçtiğiniz açık kaynak kodlu bir VT kaynak kodları üzerinde göstererek açıklayınız. Açıklama bölümüne kısaca metninizi yazıp, kod üzerinde gösterim videonuzun linkini en altta belirtilen kutucuğa yerleştiriniz.

- [X]  Seçtiğiniz konu/konuları bu şekilde işaretleyiniz. **!**
    
---

# 1. Sistem Perspektifi (Operating System, Disk, Input/Output)

### Disk Erişimi

- [X]  **Blok bazlı disk erişimi** → block_id + offset
- [x]  Rastgele erişim

### VT için Page (Sayfa) Anlamı

- [X]  VT hangisini kullanır? **Satır/ Sayfa** okuması

---

### Buffer Pool

- [X]  Veritabanları, Sık kullanılan sayfaları bellekte (RAM) kopyalar mı (caching) ?

- [X]  LRU / CLOCK gibi algoritmaları
- [ ]  Diske yapılan I/O nasıl minimize ederler?

# 2. Veri Yapıları Perspektifi

- [X]  B+ Tree Veri Yapıları VT' lerde nasıl kullanılır?
- [ ]  VT' lerde hangi veri yapıları hangi amaçlarla kullanılır?
- [ ]  Clustered vs Non-Clustered Index Kavramı
- [ ]  InnoDB satırı diskte nasıl durur?
- [ ]  LSM-tree (LevelDB, RocksDB) farkı
- [ ]  PostgreSQL heap + index ayrımı

DB diske yazarken:

- [X]  WAL (Write Ahead Log) İlkesi
- [ ]  Log disk (fsync vs write) sistem çağrıları farkı

---

# Özet Tablo

| Kavram      | Bellek          | Disk / DB      |
| ----------- | --------------- | -------------- |
| Adresleme   | Pointer         | Page + Offset  |
| Hız         | O(1)            | Page IO        |
| PK          | Yok             | Index anahtarı |
| Veri yapısı | Array / Pointer | B+Tree         |
| Cache       | CPU cache       | Buffer Pool    |

---

# Video [Linki](https://youtu.be/_ki6HNslgdk) 
Ekran kaydı. 2-3 dk. açık kaynak V.T. kodu üzerinde konunun gösterimi. Video kendini tanıtma ile başlamalıdır (Numara, İsim, Soyisim, Teknik İlgi Alanları). 

---

# Açıklama (Ort. 600 kelime)
Veritabanı sistemlerinde performansın temel belirleyicisi, disk G/Ç (I/O) operasyonlarının nasıl yönetildiğidir. Sistem programlama perspektifinden bakıldığında, veritabanları veriyi bayt seviyesinde değil, blok bazlı disk erişimi prensibiyle yönetir. Bu mimaride veri, disk üzerinde Page (Sayfa) adı verilen sabit boyutlu (genellikle 8KB veya 16KB) bloklara bölünmüştür. Veritabanı motoru, işletim sistemi seviyesindeki düşük seviyeli çağrıları kullanarak her sayfaya bir Page ID atar. Bu ID üzerinden yapılan adresleme, diskin fiziksel sektörlerine karşılık gelen block_id + offset hesaplamasıyla gerçekleştirilir.

Kullanıcı belirli bir kaydı sorguladığında, veritabanı indeksi kullanarak ilgili verinin hangi blokta olduğunu tespit eder ve o noktaya rastgele erişim (random access) yapar. Rastgele erişim, geleneksel okuma kafalı disklerde (HDD) "seek time" nedeniyle maliyetli olsa da, blok bazlı yapı sayesinde bu maliyet optimize edilir; çünkü diskten tek bir karakter okumak yerine o karakteri içeren tüm blok tek seferde RAM'e (Buffer Pool) çekilir. Bu yöntem, "spatial locality" (mekansal yerellik) ilkesinden faydalanarak bir sonraki erişim ihtimali yüksek olan verileri de önbelleğe almış olur. Modern NVMe disklerde rastgele erişim hızı çok yüksek olsa da, blok bazlı yönetim hala verinin işletim sistemi önbelleğiyle hizalanması ve I/O operasyon sayısının minimize edilmesi için kritik önem taşır. Özetle; blok bazlı erişim, rastgele erişimin doğurduğu fiziksel gecikmeleri disipline eden ve veritabanının diski devasa, yapılandırılmış bir bellek alanı gibi kullanmasını sağlayan en temel sistem stratejisidir.

Veritabanı sistemlerinde performans ve güvenliğin temel taşı olan WAL (Write-Ahead Logging), yavaş olan disk yazma işlemlerini hızlandırmak için "önce günlük tut, sonra asıl işi yap" prensibiyle çalışan bir mekanizmadır. Normal şartlarda, veritabanındaki bir bilgiyi değiştirmek için disk üzerindeki dağınık veri bloklarını (B+ Tree sayfalarını) bulup güncellemek, diskin farklı noktalarına sürekli zıplamayı gerektiren ve sistemi yavaşlatan bir rastgele yazma işlemidir; oysa WAL, yapılan her değişikliği küçük bir not olarak diskteki özel bir günlük dosyasına ardışık (sequential) olarak ekler. Bu yöntem, hem diskin en hızlı olduğu yazma biçimini kullanarak performansı maksimize eder hem de fsync() sistem çağrısı sayesinde verinin RAM'den uçup gitmesini engelleyerek fiziksel diske kalıcı olarak kazınmasını sağlar. Eğer bir elektrik kesintisi veya sistem çökmesi yaşanırsa, veritabanı yeniden açıldığında asıl veri dosyaları güncellenmemiş olsa bile bu WAL kayıtlarına (notlarına) bakarak eksik işlemleri tamamlar (Redo); böylece WAL olmasaydı yaşanacak olan "ya aşırı yavaşlık ya da veri kaybı" ikilemini ortadan kaldırarak veritabanının hem saniyede binlerce işlem yapabilmesini hem de hiçbir veriyi unutmamasını sağlar.

Veritabanının diskteki yavaşlığına karşı kurduğu en güçlü savunma hattı olan Buffer Pool, RAM (bellek) üzerinde ayrılmış devasa bir çalışma alanıdır ve temel görevi diskten okunan veri sayfalarını (Page) burada kopyalayarak saklamaktır. Bir sorgu geldiğinde veritabanı doğrudan diske gitmek yerine önce bu alana bakar; eğer aranan sayfa buradaysa (Cache Hit) işlem saniyeler yerine milisaniyeler içinde tamamlanır. Bellek alanı sınırlı olduğu için, yeni sayfalara yer açmak amacıyla LRU (En Son En Az Kullanılan) algoritması kullanılır; bu sayede sistem, popüler ve sık kullanılan verileri sürekli elinin altında tutarken, nadir kullanılanları bellekten atarak sınırlı RAM kapasitesini en verimli şekilde yönetir ve diskle olan trafiği minimuma indirir

Veritabanının diskteki yavaşlığına karşı kurduğu en güçlü savunma hattı olan Buffer Pool, RAM (bellek) üzerinde ayrılmış devasa bir çalışma alanıdır ve temel görevi diskten okunan veri sayfalarını (Page) burada kopyalayarak saklamaktır. Bir sorgu geldiğinde veritabanı doğrudan diske gitmek yerine önce bu alana bakar; eğer aranan sayfa buradaysa (Cache Hit) işlem saniyeler yerine milisaniyeler içinde tamamlanır. Bellek alanı sınırlı olduğu için, yeni sayfalara yer açmak amacıyla LRU (En Son En Az Kullanılan) algoritması kullanılır; bu sayede sistem, popüler ve sık kullanılan verileri sürekli elinin altında tutarken, nadir kullanılanları bellekten atarak sınırlı RAM kapasitesini en verimli şekilde yönetir ve diskle olan trafiği minimuma indirir.


## VT Üzerinde Gösterilen Kaynak Kodları

Açıklama [Linki](https://...) \
Açıklama [Linki](https://...) \
Açıklama [Linki](https://...) \
... \
...
