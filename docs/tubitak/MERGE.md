> PROJE ANA DALI: `Yazılım`
> 
> PROJE TEMATIK ALANI: `Dijital Dönüşüm`
> 
> Proje ADI: Merge Paket Yöneticisi Emülatörü

# ÖZET

Son yıllarda dijital dünyada elde edilen yeni teknolojiler sayesinde program
geliştirmek için başvurulan yöntemlerin verimliliği artmıştır. Örneğin 70'li yılların yazılımcıları programlarını delikli kartlar üzerine
yazarken bu işlem daha sonra assmebly gibi düşük seviye makine dillerine,
ardından C gibi düşük seviye programlama dillerine, günümüzde de LLVM gibi
modern altyapılar kullanan Zig ve Rust benzeri programlama dillerine dönüşmüştür.

Yaşanan değişikliğin büyük bir kısmı da standardizasyon alanında gerçekleşmiştir. Mesela C++'ta standart bir paket yöneticisi bulunmaması
nedeniyle bağımlılık yönetimi aşamsında belirsizler yaşanırken Rust bunu standardize etmiştir.

Benzer sıkıntıları işletim sistemlerinin paket yöneticileri için de söyleyebiliriz.
Örnek olarak Arch Linux `pacman -S <paket>` komutu, Scoop paket yöneticisinde
`scoop install <paket>` şeklindedir. Sanal makine gibi çok sayıda işletim sistemi kullanımı gerektiren ortamlarda yazılımcılar yüzlerce komutla çalışmaktadır
Bu durum yaşanan kafa karışıklığının boyutunu ortaya koymaktadır.

Benzeri uyumsuzluklar, açık kaynaklı programları kurarken,
sanal makineler arası geçiş yaparken veya işletim sistemi değiştirirken
sıklıkla yaşanmaktadır.

Bu probleme çözüm olarak geliştirdiğimiz `merge` emülatörü,
herhangi bir sisteme bağlı kalmadan tanımlı bütün komutların kullanılabilmesine olanak sağlar.
Az önce verilen örnekten yola çıkacak olursak
Arch linux işletim sisteminde `pacman -S <paket>` komutunun yanında `scoop install <paket>` komutunu da kullanabilirsiniz.

Bu yöntem sayesinde az önce belirtilen sorunların tamamı çözülebilmektedir.

> ANAHTAR KELIMELER: paket yöneticisi, işletim sistemi, standartlaştırma, çapraz
> platform, soyutlama

## Kaynaklar

- Punch Cards: <https://en.wikipedia.org/wiki/Punched_card_input/output>

# AMAÇ

Günümüzde yazılım geliştiricileri ana makineleriyle aynı işletim sistemine sahip olmayan sunucu ve sanal makine gibi
araçlar ile çalışmaktadırlar.
Bu da birbirinden farklılık arz eden ve karmaşıklığa sebep olan çok sayıda komut kümesi kullanmalarını gerektirmektedir.
Komutların tamamını hatırlamakta zorluk çeken yazılımcıların sık sık referanslara göz atma ihtiyacı ile birlikte dikkatleri
dağılmakta ve iş akışı olumsuz etkilenmektedir.

Figür1'de en bilinen paket yöneticileri için çok kullanılan komutlar listelenmiştir.
Açıkça görüldüğü üzere komutlar arasında standart bir bağ bulunmamaktadır.

Bu sistemleri aynı anda kullanmak durumunda kalan bir yazılımcı, her bir sistem için geçerli komutu birbirine karıştırmadan
yazmak zorundadır. Sonuç olarak da kod yazarken sürekli `man` sayfalarına ve `--help` argümanlarına başvurmaktadır.

> Figür I

| Paket Yöneticisi | İndirme Komutu           | Güncelleme Komutu          | Sorgulama Komutu          | Silme Komutu               |
| :--------------- | :----------------------- | :------------------------- | :------------------------ | :------------------------- |
| `APT`            | `apt install <paket>`    | `apt upgrade <paket>`      | `apt search <paket>`      | `apt remove <paket>`       |
| `Pacman`         | `pacman -S <paket>`      | `pacman -S <paket>`        | `pacman -Ss <paket>`      | `pacman -Rsc <paket>`      |
| `Nix`            | `nix-env -i <paket>`     | `nix search <paket>`       | `nix-env -u <paket>`      | `nix -e <paket>`           |
| `Homebrew`       | `brew install <paket>`   | `brew upgrade <paket>`     | `brew serach <paket>`     | `brew uninstall <paket>`   |
| `Chocolatey`     | `choco install <paket>`  | `choco upgrade <paket>`    | `choco search <paket>`    | `choco uninstall <paket>`  |
| `Scoop`          | `scoop install <paket>`  | `scoop update <paket>`     | `scoop search <paket>`    | `scoop uninstall <paket>`  |
| `Yum`            | `yum install <paket>`    | `yum update <paket>`       | `yum search <paket>`      | `yum remove <paket>`       |
| `Dnf`            | `dnf install <paket>`    | `dnf update <paket>`       | `dnf search <paket>`      | `dnf remove <paket>`       |
| `Zypper`         | `zypper install <paket>` | `zypper update <paket>`    | `zypper search <paket>`   | `zypper remove <paket>`    |
| `APK`            | `apk add <paket>`        | `apk upgrade <paket>`      | `apk search <paket>`      | `apk del <paket>`          |
| `Xbps`           | `xbps-install <paket>`   | `xbps-install -Su <paket>` | `xbps-query -Rs <paket>`  | `xbps-remove <paket>`      |
| `RPM`            | `rpm -i <paket>`         | `rpm -U <paket>`           | `rpm -qf <paket>`         | `rpm -e <paket>`           |
| `Portage`        | `emerge <paket>`         | `emerge --update <paket>`  | `emerge --search <paket>` | `emerge --unmerge <paket>` | [//]: # (Validate) |

Karşılaşılan bu sorunu çözmek için bütün standartları anlamlandırabilecek şekilde geliştirdiğimiz
`merge` paket yöneticisi emülatörü, sisteminize bağlı kalmadan çalışarak istediğiniz komutları
başka bir sistemde kullanmanıza olanak sağlamaktadır.

## Kaynaklar

<https://pkgs.org/search/?q=curl>

<https://github.com/ScoopInstaller/Scoop/issues/897>
<https://apple.stackexchange.com/questions/56419/how-can-i-update-everything-installed-through-homebrew-after-osx-upgrade>

<https://pkgs.org/search/?q=curl>
<https://forums.fedoraforum.org/showthread.php?191104-install-libcurl>

- Nix Paket Yöneticisi
  komutları: <https://www.mankier.com/1/nix-env> <https://github.com/brainrake/nixos-tutorial/blob/master/cheatsheet.md>
- Pacman Paket Yöneticisi komutları: <https://devhints.io/pacman>
- HomeBrew Paket Yöneticisi
  komutları: <https://devhints.io/homebrew> <https://stackoverflow.com/questions/8833230/how-do-i-find-a-list-of-homebrews-installable-packages>
- Chocolatey Paket Yöneticisi
  komutları: <https://gist.github.com/yunga/> <https://docs.chocolatey.org/en-us/choco/commands/upgrade> <https://docs.chocolatey.org/en-us/choco/commands/uninstall>
- Scoop Paket Yöneticisi komutları: <https://github.com/ScoopInstaller/Scoop/wiki/Commands>
- Yum Paket Yöneticisi
  komutları: <https://access.redhat.com/sites/default/files/attachments/rh_yum_cheatsheet_1214_jcs_print-1.pdf>
- Dnf Paket Yöneticisi komutları: <https://docs.fedoraproject.org/en-US/quick-docs/dnf/>
  [//] # (- RPM Paket Yöneticisi
  komutları: <https://www.golinuxcloud.com/rpm-command-in-linux/> <https://access.redhat.com/solutions/1189>)
- Zypper Paket Yöneticisi komutları: <https://www.maketecheasier.com/cheatsheet/zypper-package-manager/>
- APK Paket Yöneticisi komutları: <https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management>
- Xbps Paket Yöneticisi komutları: <https://docs.voidlinux.org/xbps/index.html>

# GİRİŞ

Paket yöneticilerinin belirli bir standarda bağlı olmaması problemi şimdiye kadar bazı
yazılımlar tarafından giderilmeye çalışılsa da yeterli düzeyde standartlaşma elde edilememiştir.

Yapılan literatür araştırması sonucu `merge` ile benzerlik gösteren projeler aşağıda listelenmiştir.

## Pacapt

Pacapt, adından anlaşılacağı üzere bütün paket yöneticisi komutlarını `pacman` paket yöneticisine
benzetmeye çalışmaktadır.

POSIX `sh` ile yazılan Pacapt, daha önceden `pacman` kullanmamış olan yazılımcılara hitap
etmemektedir.

## Mew

Mew paket yönetici komutlarını standartlaştırma konusunda kullanıcılara yardımcı olmayı hedefleyen ufak çaplı bir
projedir. Mew, `.PO` ([GNU gettext utilities](https://www.gnu.org/software/gettext/manual/html_node/PO-Files.html))
dosyaları gibi çalışır.  Geliştiricisi, 6 yıl önce projeyi geliştirmeyi bırakmıştır.

Planlarının çoğu tamamlanmadan bırakılmış olan Mew, veri tabanında yalnızca `rpm` ve `pacman`'i
debian paket yöneticilerine çevirebilmektedir.

Merge ise sadece bir kaç paket yöneticisiyle sınırlı olmayıp sık kullanılan bütün sistemler arasında
çapraz çevirme yapabilmektedir.

## Pacman Rosetta

- Site: <https://wiki.archlinux.org/title/Pacman/Rosetta>

İnternet üzerinde, paket yönetici komutlarının eşleştirildiği bir dökümandır. Dökümanda yer alan paket yöneticisi sayıları kısıtlı olduğundan
ve yazılımcının dökümanı okuması gerektiğinden yeterli değildir.

Sonuç olarak sistem komut standartlaştırması için yeterli yazılım bulunmamaktadır.
Bu nedenle `merge` projesinin alanında etkili olması öngörülmektedir.

## Kaynaklar

- yumitude: <https://github.com/timols/yumitude> Project with the same idea but not implemented.
- MEW: <https://github.com/fossasia/mew> Project similar to pacaptr.
- whohas: <https://github.com/whohas/whohas> - A system utility to search from general package registries.
- pacaptr: <https://github.com/icy/pacapt> - pacman-like syntax for all package managers.
- bedrock linux: <https://bedrocklinux.org/0.7/pmm-beta.html> package manager manager (a.k.a pmm)

# YÖNTEM

`merge` emülatörünü etkili bir şekilde geliştirmek için kullandığımız araçlar
aşağıda listelenmiştir:

## Rust Programlama Dili

### Cargo Paket Yöneticisi

Günümüz programlama dili paket yöneticilerinin modern standartlarına uygun şekilde
yöneten `cargo`, `merge` ekosisteminin geliştirilmesinde büyük rol oynamıştır.

`cargo` sayesinde projemizi derlemek, test etmek, yayımlamak gibi eylemler
daha verimli hale getirilmiştir

### Yüksek Seviye Sözdizimi

C ve C++ gibi düşük seviye programlama dillerinin aksine Rust, yüksek seviye bir
sözdizimine sahiptir. Bu sayede programcılar, düşük seviye programlama
dillerinde karşılaştıkları okunabilirik, yeniden düzenleme (refactoring) gibi
konularda sıkıntılar yaşamazlar.

### Ödünç Alma Denetleyicisi (Borrow Checker)

Rust sahiplik (ownership) ve ödünç alma (borrowing) kavramları ile bütün
bellek yönetimini derleme zamanında yapar. Böylece
programcılar, bellek yönetimi ile uğraşmak zorunda kalmayıp
işleyiş anındaki bellek hatalarının çoğunun önüne geçmiş olur.

### Sistem Seviyesinde Performans

Rust, LLVM Derleyici altyapısını temel alan bir programlama dilidir. Bu yüzden
yüksek seviyesinde performans sağlar.

### Yeni Nesil Programlama

Rust yarı fonksiyonel bir programlama dili olduğundan dolayı fonksiyonel programlama
dillerinin sahip olduğu kısa ve okunabilir kod yapısına sahiptir.

Desteklediği güçlü makro sistemi projemizi geliştirirken kod tasarrufu yapmamızı
sağlamıştır.

Desen eşleme, trait sistemi, güçlü ve cebirsel veri tipleri, fonksiyonel programlama dillerinde
öne çıkan özellikler sayesinde temiz ve deyimsel (idiomatic) bir kod tabanı elde edilmiştir.

`rustc` ile birliklte çapraz derleyebildiğimiz `merge`,
bütün modern işletim sistemlerinde çalışabilmektedir.

### Eskiye Uyumluluk (Backward Compatiblity)

Rust eskiye uyumlu olduğu için `merge` kaynak kodunu ileri safhalardaki derleyiciler de
çalıştırabilecek ve böylece projemiz daha güvenli şekilde geliştirilebilecektir.

## Git Versiyon Kontrol Sistemi

Merge projesinin mekandan bağımsız ve eş zamanlı geliştirilebilmesi için bir
organizasyon sistemine ihtiyaç duyulmuştur. Proje geliştirilirken, sürdürülebilirlik,
geliştirme, test, dağıtım gibi pek çok aşamada kolaylık sağlanması adına
endüstriyel standartlardan birisi olan Git Versiyon Kontrol sistemini kullanıldı.

## Github

Sorun takipçisi (issue tracker), kod incelemesi (code review), özellik
talebi (feature request), wiki gibi geniş çaplı projeler için gerekli olan
yapıları oluşturacak ve Git ile entegre çalışacak bir barındırma servisi
olarak GitHub platformunu kullanılmıştır.

Bu sayede `merge`'in geliştirilmesi ve kullanıcıların karşılaştıkları problemlerin
daha hızlı çözülmesi hedeflenmiştir.

## JetBrains IDE, VSCode, Helix ve Vimacs

`merge`, IDE ve editör sektöründe profesyonel yazılımcılar tarafından önerilen JetBrains
temelli RustRover, CLion ve VSCode IDE'leri; Vimacs[1] ve Helix terminal editörleri
kullanılarak geliştirilmiştir.

Büyük bir kod tabanı ile çalışırken, kodun okunabilirliği ve yeniden
düzenlenebilirliği (refactoring) gibi konulara dikkat etilmesi gerektiğinden
kullanılan editörler, bulut kullanılarak senkorize edilmiş;
ileri düzeyde etkili araç entegrasyonları (Git, GitHub, DB, JetBrains AI, Github
Copilot vb.), güçlü grafiksel arayüz tasarımı, kod üretimi ve
düzenleme araçları, konfigüre edilip `merge` projesini geliştirmek
için kullanılımıştır.

4 farklı editörün aynı proje için kullanılmasının nedeni her bir editörün kendine
özgü güçlü yanlarının bulunmasıdır.

İşletim sistemi olarak Windows ve NixOS,
Linux sistem kabuğu olarak `nushell`, `bash` ve `fish`,
Sistem paket yöneticisi olarak `nix`,
Scriptler için `sh`,
Klavye konfigürasyon modu olarak `vim`,
Web temelli editor olarak replit ve rust-playground

kullanılmıştır.

Konfigurasyon dosyaları için bkz. Ek 2:

- NixOS işletim sistemi: <https://github.com/utfeight/dotnix>
- Vimacs:
  - Geliştirdiğim vimacs yazılımının kaynak kodu:
    <https://github.com/utfeight/vimacs>
  - Vimacs konfigurasyon dosyaları: <https://github.com/utfeight/vimax>
- JetBrains:
  - ideavimrc: <https://github.com/utfeight/dotideavimrc>
  - TODO: link dotnix ft
- VSCode:
  - Nix ile yazılmış dekleratif konfigürasyon: TODO: link dotnix ft
- bütün configurasyon dosyaları için: TODO: link em all

## Merge Algoritmaları

Merge, geliştirilmeye açık olarak tasarlanmak istenildiğinden temel programlama
prensiplerine uygun olarak temiz bir kod tabanı üzerine geliştirilmesi
planlanmıştır. Bunun için endişelerin ayrılması (Seperation of Concerns) ile doğru miktarda uyum ve
bağlantı (cohesion & coupling) gibi pek çok programlama prensibi göz önünde bulundurularak
tasarlanmıştır.

Geliştirilmesi için bir çok rust kütüphanesinden yararlanılmıştır. (bkz.
Cargo.toml's : TODO)

### MgTWIN

> Merge çift yönlü tercüme (`merge` Two-Way InterpretatioN)

Merge programının yeni bir standart oluşturmadan diğer standartları anlaması
için geliştirilmiş olan çift yönlü tercüman modülüdür. Bu modül, Paket
Yöneticilerinin eymleri için belirlediği komutları anlamlandırarak diğer paket
yöneticilerinin komutlarına çevirmek için geliştirilmiştir.

Örneğin dwm X pencere yöneticisini indirmek isteyen bir kullanıcı aşağıdaki
komutlardan herhangi birini kullanabilir.

```shell
merge apt install dwm
merge pacman -S dwm
merge emerge --install dwm
merge nix shell dwm
...
```

yukarıda verilen komutlar bütün sistemlerde çalışacaktır çünkü `merge`
bu komut sistemlerinin hepsini anlamlandırıp kullandığınız işletim sisteminin
komutlarına çevirebilmektedir.

### MgMIR

`merge`'in inovatif konfigürasyon stili için kullanılan `MgMIR`,
hacimsel olarak çok az yer kaplamasıyla kendisine benzeyen sistemlerden ayrılmaktadır. `merge`'in ileriye dönük
geliştirilmesinin kolay olması adına konfigürasyon dosyalarını minimum hacimde
tutulmaya yönelik bir ara dil üretilmiştir.

Veritabanı kullanımı açısından `merge` emülatörü, `mew` projesine benzemektedir.
`merge`, aşağıda belirtildiği üzere `mew`'in eksik yanlarından ders alıp onları
tamamlamıştır.

### Minimal konfigürasyon hacimleri

LLVM derleyici altyapı sistemi ve JVM Byte Code gibi tasarım şekillerinden
esinlenerek `merge`, konfigürasyonu minimumda tutmak adına kendi MIR'ini
geliştirdik. MgMIR adını verdiğimiz bu basit MIR, son kullanıcının `merge`'e
eklemeler yapmasını kolaylaştırmaktadır.

### Geniş Sistem Desteği

Mew yalnızca `pacman` ve `rpm` ile debian temelli paket yöneticilerine çevirim
yapabildiği için yeterli sayıda kullanıcıya hitap etmemektedir.

`merge` ise aşağıda listelenen bütün paket yöneticilerini desteklemektedir.

1. apk
2. apt
3. homebrew
4. choco
5. dnf
6. emerge
7. nix
8. pacman
9. scoop
10. yum
11. zypper

Bütün durumları aynanda anlamlandırabilen `merge` toplamda 121 farklı kombinasyon ile çalışabilir.

Yeni konfigürasyonlar eklemenin de kolay olduğu `merge`'in çalıştırabildiği komutların günden güne artması beklenmektedir.

### MgPMS (Merge Package Manager Search)

> Merge Paket arama aracı

`merge`'in yazdığınız komutları çalıştırabilmesi için sisteminiz hakkında bilgi
edinmesi gerekir. Bu verilere ulaşmak için geliştirdiğimiz `MgPMS` modülü,
sisteminizdeki paket yöneticilerini güvenli bir şekilde `merge`'e aktarır.

### MgCLI (Merge Commandline Interface)

> Merge Komut Satırı Arayüzü

Rust'ın güçlü presedürel makro sistemini kullanan `clap` kütüphanesini baz alan
`MgCLI` sayesinde `merge`'in kullanımı daha pratik hale getirilmiştir.

### MgDB (Merge Database)

> Merge Veri tabanı

`merge`, kendisine verdiğiniz komutları anlamlandırabilmek için MgDB adını
verdiğim bir hafıza birimi kullanır. Bu hafıza birimi varsayılan şekilde sistem
konfigürasyon dosyası olarak `$XDG_CONFIG_HOME` sistem değişkeninde bulunur.

Böylece `merge`'i gerçek zamanlı konfigüre edebilir ve istediğiniz mantıksal
terimleri `MgMIR` kullanarak tanımlayabilirsiniz.

## Ekler

GitHub: <https://en.wikipedia.org/wiki/GitHub> Rust as a functional lang:
<https://kerkour.com/rust-functional-programming>

# Proje İş-Zaman Çizelgesi

> Projenin iş-zaman çizelgesine sahip olması çok önemlidir. İlk haftadan itibaren araştırma projenizin süresi boyunca ulaşmanız gereken hedefleri ve zamanlarını belirten aşağıda verilene benzer bir iş-zaman çizelgenizin olması gerekir.

# Bulgular

> Çalışmada toplanan veriler ve verilere ait analiz sonuçları verilir.

Projemizin çalışır haldeki asciinema klibini izlemek için: <https://asciinema.org/a/630971>

## Büyük O notasyonu

`merge`, Performans açısından muadillerine karşı üstünlük elde etmiştir.

Öyle ki `mew` paket çevirmeni, konfigürasyon dosyaları için büyük O notasyonu uzay karmaşıklığı (Space Complexity) `O(n)` iken
spesifik olarak hacimsel optimizasyon ile `MgMIR` aynı işlemi düz uzay (optimum karmaşa) olan `O(1)`
ile gerçekleştirebilmektedir.

## Kaynak Kodu

`merge`, tamamen ücretsiz ve açık kaynak bir yazılımdır.
Kaynak kodu için [Ek2'ye](https://github.com/orhnk/merge) bakınız.

## Loglar

`merge` çalışırken oluşabilecek sıkıntılar için yapılan işlemi anlık olarak kullanıcıya bildirmektedir.

Aşağıda Rust'ın standart kütüphanesinden `dbg!()` makrosunun çıktıları verilmiştir:

![image](https://github.com/denizbaba0/merge/assets/107251435/1af6acf0-2885-4030-946b-ad1c6604b3d3)

Fotoğrafta verildiği üzere oluşturulan son komut kullanıcıya `INFERRED CMD: <komut>` şeklinde bildirilmektedir.

# Sonuç ve Tartışma

> Bu bölümde proje çalışması ile elde edilen bulgular araştırma sorusuna veya problemine uygun olarak yorumlanır.

İnternet üzerinden projemiz üzerine tartıştığımızda faydalı olduğu kanısına varılmıştır.

Özellikle linux kullanmaya yeni başlayan kişiler sıklıkla sistem değiştirmekte oldukları için
öğrendikleri komutları tekrar başka sistemler için öğrenmeleri gerekmektedir.

Bulut üzerinde veya sanal makineler ile çalışan yazılımcılar, ana makinelerinden farklı işletim sistemlerini kullanmak zorunda olduğu için
ana bilgisayarlarındaki komutları diğer sistemlerde kullanamazlar.

Bu gibi adaptasyon sorunlarını çözmekte olan `merge` hem kolay konfigüre edilebilen hem de geniş kullanıcı kitlesine hitap eden bir proje olduğundan
sektördeki yazılımcıların hoş karşılayacağı bir projedir.

# Öneriler

> Bu bölümde benzer çalışmalar yapacak olanlara yol göstermesi bakımından öneriler varsa belirtilir.

## Kullanım Kolaylığının Önemi

`merge`'i geliştirmeye başladığımız zaman kompleks ayırıcı algoritmalar ve gerçekleştirilmesi zor planlar içerisindeydik.
Daha sonraları projemizi sadeleştirmenin hem bakım faaliyetleri hem de sonuç bakımından bize daha faydalı olacağını düşündük
ve kod tabanımızı sadeleştirdik.

Böylece kullanımı ve geliştirilmesi kolay bir kod tabanı elde ettik.

## Veri Tabanı Gerekli mi?

Veri tabanı kod tabanımızı basitleştirmekte çok yardımcı olduğundan dolayı
programınızı yazarken sizlere rahatlık sağlayacaktır.

## Veri Tabanı Derlenen Koda Dahil Olmalı Mı?

`merge` veritabanını konfigüre edilebilir tutma amacı ile verilerini sistem dosyalarınızdan çekmektedir.
Eğer daha az esnek bir yapıya sahip bir proje ile yalnızca kullanımı kolaylaştırmak istiyorsanız
veritabanınızı programınıza gömmeyi tercih edebilirsiniz.

## Çözümleme (Parsing) Gerekli Mi?

`merge` `MgMIR` dışında cüzi miktarda çözümleme yapmakla beraber kompleks algoritmalar kullanmamaktadır.
Komut satırından edindiğiniz kod parçaları yapıları bakımından basit oldukları için regex gibi yazı
manipülasyon sistemleri kullanılabilir.

# Kaynaklar

> Bu bölümde, proje sürecinde yararlanılan ve proje raporu içerisinde atıf yapılan tüm kaynaklar listelenir. Kaynaklar APA yazım kuralları ve kaynak gösterme biçimine göre listelenir.

# Ekler

> Varsa konuyu dağıtacağı düşünülen veya çok uzun metinlerden oluşan, çeşitli araştırma bulgularına dayalı çok uzun tablolar, formüller, ayrıntılı deney verileri, bilgisayar programları, anketler vb. EKLER bölümünde verilebilir. Araştırmayı yapmak için alınan yasal izinler, yazışmalar, gerekirse e-posta örnekleri de burada verilmelidir. Eklerin her biri için uygun bir başlık seçilerek metin içerisinde geçiş sıralarına göre "Ek 1., Ek 2..." şeklinde, ayrı bir sayfadan başlayacak şekilde yer almalıdır.

> Eklerin proje raporunun sayfa sınırı olan 20 sayfaya sığmaması durumunda e-bideb sisteminde EK BELGELER kısmına yüklenmesi gerekmektedir. Bu durumda proje raporunda EK BELGELER kısmına dosya yüklendiği belirtilmeli ve eklenen belgeler liste halinde yazılmalıdır.
