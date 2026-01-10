# Deney Sonu Teslimatı

Sistem Programlama ve Veri Yapıları bakış açısıyla veri tabanlarındaki performansı öne çıkaran hususlar nelerdir?
  
---

# 1. Sistem Perspektifi (Operating System, Disk, Input/Output)

### Disk Erişimi

- [X]  **Blok bazlı disk erişimi** → block_id + offset
- [ ]  Rastgele erişim

### VT için Page (Sayfa) Anlamı

- [X]  VT hangisini kullanır? **Satır/ Sayfa** okuması

---

### Buffer Pool

- [X]  Veritabanları, Sık kullanılan sayfaları bellekte (RAM) kopyalar mı (caching) ?

- [X]  LRU / CLOCK gibi algoritmaları
- [X]  Diske yapılan I/O nasıl minimize ederler?

# 2. Veri Yapıları Perspektifi

- [ ]  B+ Tree Veri Yapıları VT' lerde nasıl kullanılır?
- [ ]  VT' lerde hangi veri yapıları hangi amaçlarla kullanılır?
- [ ]  Clustered vs Non-Clustered Index Kavramı
- [ ]  InnoDB satırı diskte nasıl durur?
- [ ]  LSM-tree (LevelDB, RocksDB) farkı
- [ ]  PostgreSQL heap + index ayrımı

DB diske yazarken:

- [ ]  WAL (Write Ahead Log) İlkesi
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

# Video [Linki](https://www.youtube.com/watch?v=Nw1OvCtKPII&t=2635s](https://youtu.be/PxBGjQ1xIF8) 

---

# Açıklama

Veritabanı yönetim sistemlerinde (VTYS) performansı belirleyen temel unsurların başında disk erişimi gelmektedir. 
Sistem programlama perspektifinden bakıldığında, disk erişimi bellek erişimine kıyasla oldukça maliyetlidir. 
RAM üzerinden yapılan erişimler pointer seviyesinde O(1) karmaşıklığa sahipken, disk erişimleri blok ve sayfa 
bazlı gerçekleşir ve milisaniyeler mertebesinde gecikmelere neden olur. Bu nedenle modern veritabanı sistemleri, 
disk I/O işlemlerini minimize edecek şekilde tasarlanmıştır.

İşletim sistemleri diskleri blok bazlı aygıtlar olarak ele alır. Bir veri okunmak istendiğinde, doğrudan tek bir 
bayt veya satır değil, belirli büyüklükteki disk blokları okunur. Veritabanları bu blokları kendi mantıksal 
yapılarına uyarlayarak “page (sayfa)” kavramını kullanır. PostgreSQL gibi ilişkisel veritabanlarında bir sayfa 
genellikle 8KB boyutundadır ve tablolar ile indeksler bu sayfalardan oluşur. Bu durum, veritabanlarının satır 
bazlı değil, sayfa bazlı okuma ve yazma yaptığını göstermektedir.

Disk erişim maliyetini azaltmak amacıyla veritabanları Buffer Pool (buffer cache) adı verilen bir bellek alanı 
kullanır. Buffer Pool, sık kullanılan veri ve indeks sayfalarının RAM üzerinde tutulduğu bir önbellek yapısıdır. 
Bir sorgu çalıştırıldığında, ilgili veri sayfasının öncelikle Buffer Pool içerisinde bulunup bulunmadığı kontrol 
edilir. Eğer sayfa bellekte mevcutsa, disk erişimi yapılmadan doğrudan RAM üzerinden işlem gerçekleştirilir. 
Bu duruma cache hit denir. Sayfa bellekte yoksa diskten okunur ve Buffer Pool’a eklenir; bu da cache miss olarak 
adlandırılır.

Ancak Buffer Pool boyutu sınırlı olduğundan, bellek dolduğunda hangi sayfanın çıkarılacağına karar verilmesi gerekir. 
Bu noktada veri yapıları ve algoritmalar devreye girer. PostgreSQL gibi sistemler LRU (Least Recently Used) 
yaklaşımının daha verimli bir türevi olan CLOCK algoritmasını kullanır. CLOCK algoritmasında her sayfa için bir 
kullanım biti tutulur ve uzun süredir erişilmeyen sayfalar bellekten çıkarılarak yerlerine yeni sayfalar yüklenir. 
Bu yaklaşım, sık kullanılan sayfaların bellekte kalmasını sağlayarak disk I/O miktarını ciddi ölçüde azaltır.

Sistem programlama açısından Buffer Pool, işletim sisteminin sayfa önbelleğine benzer şekilde çalışır; ancak 
veritabanına özgü erişim örüntülerini dikkate alarak optimize edilmiştir. Örneğin indeks sayfalarının bellekte 
tutulması, SELECT ve JOIN gibi sorguların performansını doğrudan artırır. Çünkü B+ Tree tabanlı indekslerde kök ve 
üst seviye düğümlere sık erişilir ve bu düğümlerin RAM’de bulunması sorgu maliyetini düşürür.

Açık kaynak PostgreSQL veritabanı incelendiğinde, buffer yönetiminin src/backend/storage/buffer dizini altında 
gerçekleştirildiği görülmektedir. ReadBuffer_common fonksiyonu ile bir sayfanın buffer pool’da olup olmadığı 
kontrol edilir. Buffer Pool doluysa, StrategyGetBuffer fonksiyonu CLOCK algoritmasını kullanarak uygun buffer’ı 
seçer. Bu tasarım sayesinde PostgreSQL, disk erişimlerini minimumda tutarak yüksek performans ve ölçeklenebilirlik 
sağlar.

Sonuç olarak, veritabanı performansını artıran en önemli hususlar; disk yerine belleğin etkin kullanımı, sayfa 
bazlı erişim modeli, Buffer Pool mekanizması ve LRU/CLOCK gibi verimli sayfa değiştirme algoritmalarıdır. Bu 
yaklaşımlar, sistem programlama ve veri yapıları prensiplerinin veritabanı tasarımına başarılı bir şekilde 
uygulanmasının somut örnekleridir.

## VT Üzerinde Gösterilen Kaynak Kodları

PostgreSQL Buffer Manager  
https://github.com/postgres/postgres/tree/master/src/backend/storage/buffer

bufmgr.c – Buffer Pool kontrolü  
https://github.com/postgres/postgres/blob/master/src/backend/storage/buffer/bufmgr.c

freelist.c – CLOCK algoritması  
https://github.com/postgres/postgres/blob/master/src/backend/storage/buffer/freelist.c
