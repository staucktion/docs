# Business Flows

## Timer Mekanizması

- Timer cron ile çalışıp belirli bir zaman aralığında tetiklenir.
- Eğer cron 10 saate ayarlanmış ise, auction stateleri şekilde devam eder:
  `0-10:` vote
  `10-20:` auction
  `20-30:` finish
- Bu şekilde döngü tekrar eder.

<hr/>

## Fotoğraf Yüklendikten Sonra

- Yüklenen fotoğraf 'wait' satatusunde kayıt edilir.
- Yüklenen fotoğraf 'is_auctionable' false olarak kayıt edilir.
- Validator fotoğrafı 'approve' statuse çektikten sonra, kullanıcı artık fotoğrafı 'is_auctionable' true yapabilir.
- 'is_auctionable' true olan bir fotoğraf bir sonraki auction'a otomatik olarak kayıt edilir.

<hr/>

## Auction Oluşturulması

- Cron tetiklenir.
- Category'ler listelenir.
- Eğer bir "approve" status'üne sahip bir category varsa, ve bu category için bir auction yoksa, veya auctionlar var ise ve hepsi "finish" status'üne sahip ise, sonraki basamak ile devam et.
- Photo'lar listelenir.
- Eğer bir category içerisinde en az 1 tane 'is_auctionable' true olan bir photo var ise, sonraki basamak ile devam et.
- Category için 1 tane auction eklenir. category_id setlenir. start_time ve finish_time cronun zaman aralığına göre setlenir. status "vote" olarak setlenir.
- cron tablosundaki last_triggered_time setlenir.
- photo auction id'si setlenir.
- category'nin içerisindeki photo 'approved' statusten 'vote' statuse geçer.

## Auction status'ü 'vote' iken 'auction' durumuna geçmesi

- Cron tetiklenir
- Eğer bir category'e ait auction 'vote' statusünde ise, sonraki basamak ile devam et.
- Her auction'ı gez ve 'vote' statusune sahip olan auction statusu 'auction' yap.
- Bu auction'a ait olan photolar eğer ilk %10 luk dilimde ise status'ü 'auction' olarak değişir.
- Bu auction'a ait olan photolar eğer ilk %10 luk dilimde değil ise status'ü 'purchasable' olarak değişir.
- Status'u auction photolar için photo_auction table'ına bu photo için alan eklenir. Status 'auction' olarak setlenir.

<hr/>

## Provision
- Kullanıcı banka credential'ları ile birlikte istek yapar.
- .env'de belirtilen kadar provision yapılır.
- Provision yapmak için bank-api isteği gönderilir.
- Provision bakiye yetersizliği sebebiyle yapılamazsa kullanıcının status'ü 'banned' olarak değiştirilir.
- Porvision yapılabilirse, provision kaldırılır ve kullanıcının status'ü 'active' olarak değiştirilir.

<hr/>

## Bid Cycle
- Kullanıcı auction_photo kısmındaki bir fotoğrafa bid yapmak için istek atar.
- Eğer kullanıcının status'ü 'active' değil ise reddedilir ve provision yapması için mesaj döner.
- İstek atılan photo auction_photo table'ında sorgulanır. Eğer varsa ve status 'auction' ise sonraki basamak ile devam et.
- İstek atılan photo eğerki kendi photo'su ise reddedilir.

