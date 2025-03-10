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
- Eğer bir category içerisinde en az 1 tane 'is_auctionable' true olan bir photo var ise, ve en az 1 tane status 'approve' olan bir photo var ise sonraki basamak ile devam et.
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

## Auction status'ü 'auction' iken 'finsih' veya 'wait_purchase_after_auction' durumuna geçmesi

- Cron tetiklenir
- Eğer bir category'e ait auction 'auction' statusünde ise, sonraki basamak ile devam et.
- Her auction'ı gez.
- Her auction'ın auction_photo ları gez.
- Auction_photo status 'auction' ise,
- Bidleri listele
- Eğer bid yok ise auction_photo status'ü 'finish' yap, bu auction_photo içerisindeki photo'nun status'u de 'finish' yap, auction status u de 'finish' yap.
- Eğer bid var ise auction_photo status'ü 'wait_purchase_after_auction' yap, bu auction_photo içerisindeki photo'nun status'u de 'wait_purchase_after_auction' yap, auction status u de 'wait_purchase_after_auction' yap.
- Auction_photo tablosunda 'current_winner_order' 1 olarak setle.
- Auction_photo tablosunda 'winner_user_id_1', 'winner_user_id_2' ve 'winner_user_id_3' ü eğer uygunsa setle (Sadece 1 kişi bid vermiş olabilir bu durumda sadece 1 setlenir).
- Auction içerisindeki 'purchasable' photoları filtrele status 'finish' olarak setle.

## Auction 'wait_purchase_after_auction' statusunde iken photo alınması

- Auction'ın status u 'wait_purchase_after_auction' durumuna geçtiği anda 'current_winner_id' li kullanıcı 'last_bid_amount' tutarındaki banka bilgileri ile istek yaparak bu photo'yu satın alabilir.
- Cron'un bir sonraki tetiklenişinde:
- 'winner_user_id_1' bid yapıp satın almadığı için status 'banned' setlenir.
- 'winner_user_id_2' var mı diye kontrol edilir.
- Eğer var ise 'current_winner_id' bir artar (2 oldu) ve artık 'winner_user_id_2' photo yu satın alabilir.
- Eğer yok ise auction, auction_photo photo, photo status u 'finish' durumuna geçer.
- Cron'un bir sonraki tetiklenişinde:
- 'winner_user_id_3' var mı diye kontrol edilir.
- Eğer var ise 'current_winner_id' bir artar (3 oldu) ve artık 'winner_user_id_3' photo yu satın alabilir.
- Eğer yok ise auction, auction_photo photo, photo status u 'finish' durumuna geçer.

## Photo satın alımı

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
- Kullanıcının yaptığı istekteki 'bidAmount' eğer 'auction_photo' tablosundaki 'last_bid_amount' değerine eşit veya daha düşükse, istek reddedilir. Bid miktarı yetersiz mesajı döner.
- 'auction_photo' tablosundaki 'last_bid_amount' değeri güncellenir ve 'bid' tablosuna yeni veri kaydedilir.

<hr/>

## Voting

- Kullanıcı photo id ile istek atar.
- photo kendisine ait ise istek reddedilir.
- photo'nun auction'ı getirilir.
- auction bulunamazsa, veya auction 'vote' statuste değil ise, istek reddedilir.
- Kullanıcıya ait olan vote'lar auction id filtresi ile getirilir.
- Vote sayısı 10 ise istek reddedilir.
- Kullanıcı bu photo yu daha önce bu photo için vote kullandıysa istek reddedilir.
- photo'da bulunan 'vote_count' bir artar.
- Vote tablosuna user id , photo id setlenir. 'transfer_amount' null olarak setlenir. status 'vote' olarak setlenir.
