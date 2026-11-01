# Cleaning Log

Ringkasan keputusan pembersihan data Airbnb NYC. Kode lengkapnya ada di [`01_airbnb_data_preparation.ipynb`](../01_airbnb_data_preparation.ipynb).

**Hasil:** 102.599 baris dan 26 kolom menjadi 102.045 baris dan 30 kolom (`airbnb_cleaned_for_dashboard.csv`).

## Langkah umum

| # | Tahap | Keputusan | Dampak |
|---|---|---|---|
| 1 | Nama kolom | Huruf kecil, spasi menjadi garis bawah (misalnya `host id` menjadi `host_id`) | 26 kolom |
| 2 | Duplikat | Baris yang sama persis di semua kolom dibuang | 541 baris (0,53%) |
| 3 | Kolom `license` | Dibuang karena kosong 100% setelah duplikat dibuang | 1 kolom |
| 4 | Salah ketik wilayah | `brookln` menjadi Brooklyn, `manhatan` menjadi Manhattan | 2 baris |
| 5 | Tipe data teks | `price` dan `service_fee`: simbol `$` dan koma dibuang, jadi desimal; `last_review`: jadi tanggal | 3 kolom |
| 6 | Nilai kosong | Lihat tabel berikut | — |
| 7 | Tipe bilangan bulat | Lima kolom desimal diubah ke `int64` setelah nilai kosongnya terisi | 5 kolom |
| 8 | Nilai tidak valid | Lihat bagian "Nilai tidak valid" | 3.185 + 13 + 13 |
| 9 | Outlier | Dilaporkan, tidak dibuang | — |
| 10 | Kolom baru | Lima kolom kategori | 5 kolom |

## Penanganan nilai kosong

| Kolom | Kosong | Penanganan | Alasan |
|---|---|---|---|
| `license` | 100% | Dibuang | Tidak informatif |
| `house_rules` | 50,80% | Diisi `No Special Rules`; sebelum itu dibuat penanda `has_house_rules` (Yes/No) | Kosong berarti tidak ada aturan khusus |
| `last_review` | 15,51% | Dibiarkan kosong | Listing yang belum punya review tidak punya tanggal review |
| `reviews_per_month` | 15,50% | Diisi 0 | Tidak ada review berarti 0 review per bulan |
| `number_of_reviews` | 0,18% | Diisi 0 | Sama seperti di atas |
| `calculated_host_listings_count` | 0,31% | Diisi 1 | Host yang terdaftar pasti punya minimal 1 listing |
| `price`, `service_fee`, `minimum_nights`, `availability_365`, `construction_year`, `review_rate_number` | 0,21% sampai 0,44% | Diisi median | Sebaran miring, median lebih tahan terhadap nilai ekstrem |
| `host_identity_verified` | 0,28% | Diisi `unconfirmed` | Mengikuti nilai bawaan data |
| `name`, `host_name` | 0,24%, 0,40% | Diisi `Anonymous` | Penanda identitas yang tidak tercantum |
| `country`, `country_code` | 0,52%, 0,13% | Diisi `United States`, `US` | Seluruh data berlokasi di New York |
| `instant_bookable`, `cancellation_policy` | 0,10%, 0,07% | Diisi `Not Specified` | Tidak ditebak |
| `neighbourhood_group`, `neighbourhood` | 0,03%, 0,02% | Diisi `Unknown` | Tidak ditebak |
| `lat`, `long` | 0,01% | Dibiarkan kosong | Menghindari titik peta palsu |

## Nilai tidak valid

| Aturan | Nilai tidak valid | Penanganan |
|---|---|---|
| `price` lebih dari 0 | 0 | — |
| `minimum_nights` minimal 1 | 13 | Dibatasi ke minimal 1 (*capping*) |
| `availability_365` antara 0 dan 365 | 3.185 (3,12%) | Dibatasi ke rentang 0-365 (*capping*) |
| `service_fee` minimal 0 dan lebih kecil dari `price` | 13 | Baris dibuang |
| `review_rate_number` antara 1 dan 5 | 0 | — |
| `construction_year` tidak di masa depan | 0 | — |
| Koordinat bukan 0,0 | 0 | — |

Tabel lengkap ada di [`validity_check.csv`](validity_check.csv).

## Outlier

Metode IQR (batas 1,5 kali jangkauan antarkuartil). Semua outlier dipertahankan karena merupakan variasi model bisnis yang valid, bukan kesalahan input.

| Kolom | Outlier | Persentase |
|---|---|---|
| `minimum_nights` | 18.236 | 17,87% |
| `calculated_host_listings_count` | 17.699 | 17,34% |
| `number_of_reviews` | 11.695 | 11,46% |
| `reviews_per_month` | 5.726 | 5,61% |
| Lima kolom numerik lain | 0 | 0% |

Tabel lengkap ada di [`outlier_check.csv`](outlier_check.csv).

## Kolom baru (5)

| Kolom | Dasar | Kategori |
|---|---|---|
| `price_category` | `price` | Budget (sampai 200), Standard (201-500), Business-Class (501-1000), High-End (di atas 1000) |
| `host_type` | `calculated_host_listings_count` | Single Host (1), Dual Host (2), Multi Host (3-10), Enterprise (lebih dari 10) |
| `stay_length_category` | `minimum_nights` | Short Stay (1-3), Weekly Stay (4-7), Monthly Stay (8-30), Yearly-Stay (31-365), Long-Term (lebih dari 365) |
| `review_activity` | `number_of_reviews` | No Review (0), Very Low (1-5), Low (6-20), Medium (21-50), High (lebih dari 50) |
| `has_house_rules` | `house_rules` sebelum diisi | Yes, No |

## Koreksi dari versi sebelumnya

`has_house_rules` pada versi awal selalu bernilai "Yes" karena dibuat setelah `house_rules` diisi teks pengganti. Sekarang dibuat sebelum pengisian, sehingga 50.209 baris "Yes" dan 51.836 baris "No". Kolom lain tidak berubah.
