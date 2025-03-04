# Business Flows

## Timer Mekanizması

- Timer cron ile çalışıp belirli bir zaman aralığında tetiklenir.
- Eğer cron 10 saate ayarlanmış ise, auction stateleri şekilde devam eder:
  `0-10:` upload
  `10-20:` vote
  `20-30:` auction
  `30-40:` finish
- Bu şekilde döngü tekrar eder.

<hr/>

## Fotoğraf Yüklendikten Sonra

- Yüklenen fotoğraf 'wait' satatusunde kayıt edilir.
- Yüklenen fotoğraf 'is_auctionable' false olarak kayıt edilir.
- Validator fotoğrafı 'approve' statuse çektikten sonra, kullanıcı artık fotoğrafı 'is_auctinable' true yapabilir.
- 'is_auctionable' true olan bir fotoğraf bir sonraki auction'a otomatik olarak kayıt edilir.

## Auction Oluşturulması

- Cron tetiklenir.
- Category'ler listelenir.
- Eğer bir "approve" status'üne sahip bir category varsa, ve bu category için bir auction yoksa, veya auctionlar var ise ve hepsi "finish" status'üne sahip ise, sonraki basamak ile devam et.
- Photo'lar listelenir.
- Eğer bir category içerisinde en az 1 tane status'ü "approve" olan bir photo var ise, sonraki basamak ile devam et.
- Category için 1 tane auction eklenir. category_id setlenir. start_time ve finish_time cronun zaman aralığına göre setlenir. status "vote" olarak setlenir.
- cron tablosundaki last_triggered_time setlenir.

- category'nin içerisindeki 'approved' status olan photo 'vote' statuse geçer.
- photo auction id'si setlenir.

## Auction'ın status'ü 'vote' iken 'auction' kısmına geçmesi

- Cron tetiklenir
- Eğer bir category'e ait auction 'vote' statusünde ise, sonraki basamak ile devam et.
- Her auction'ı gez ve 'vote' statusune sahip olan auction statusu 'auction' yap.

- Bu auction'a ait olan photolar eğer ilk %10 luk dilimde değil ise status'ü 'purchasable' olarak değişir.
- Bu auction'a ait olan photolar eğer ilk %10 luk dilimde ise status'ü 'auction' olarak değişir.
- Status'u auction photolar için photo_auction table'ına yeni bir row eklenir. bu tabloda auction id ve photo id tutulur.
