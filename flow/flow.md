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

## 1 Auction Oluşturulması

- Cron tetiklenir.
- Category'ler listelenir.
- Eğer bir "approve" status'üne sahip bir category varsa, ve bu category için bir auction yoksa, veya auctionlar var ise ve hepsi "finish" status'üne sahip ise aşağıdaki basamaklardan devam et
- Photo'lar listelenir.
- Eğer bir category içerisinde en az 1 tane status'ü "approve" olan bir photo var ise, aşağıdaki basamak ile devam et
- Category için 1 tane auction eklenir. category_id setlenir. start_time ve finish_time cronun zaman aralığına göre setlenir. status "vote" olarak setlenir.
- cron tablosundaki last_triggered_time setlenir.