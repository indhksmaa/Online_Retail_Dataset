# Analisis Performa Penjualan Online Retail

Analisi ini bertujuan untuk menganalisis dan mengukur performa penjualan dalam bisnis ritel online. Data yang digunakan mencakup semua transaksi yang terjadi pada perusahaan ritel online selama periode tahun 2009-2011. Data ini terdiri dari 1067371 baris dan 8 kolom dengan deskripsi berikut:

- **Invoice**: Nomor invoice 6 digit yang ditetapkan secara unik untuk setiap transaksi. Jika kode ini dimulai dengan huruf 'C', itu menunjukkan pembatalan.
- **StockCode**: Kode produk (barang) yang terdiri dari angka 5 digit. Setiap produk memiliki kode yang unik.
- **Description**: Nama produk yang terkait dengan kode produk (StockCode).
- **Quantity**: Jumlah kuantitas produk yang terjual atau tercatat dalam setiap transaksi.
- **InvoiceDate**: Tanggal dan waktu invoice. Ini mencakup hari dan jam transaksi dibuat.
- **UnitPrice**: Harga satuan atau harga produk per unit dalam mata uang sterling (£).
- **CustomerID**: Nomor identifikasi pelanggan yang terdiri dari 5 digit. Setiap pelanggan memiliki nomor identifikasi yang unik.
- **Country**: Nama negara tempat tinggal pelanggan yang melakukan transaksi.

## Langkah-Langkah Analisis Data

### 1. Load Data
Pertama, kami memuat data yang akan digunakan dalam analisis. Data ini berasal dari file `online_retail_II.csv`.

```python
import pandas as pd
df = pd.read_csv('/content/drive/MyDrive/Rakamin/online_retail_II.csv')
```

### 2. Eksplorasi Data (EDA)

Setelah load data, langkah selanjutnya adalah melakukan eksplorasi data (EDA) untuk memahami data yang kita miliki. Kami melihat informasi dasar dari dataset, seperti jumlah baris dan kolom, serta deskripsi statistik dari data.

**Langkah-langkah Eksplorasi Data:**

1. **Informasi Dataset:** Pertama, kami memeriksa informasi dasar tentang dataset, termasuk tipe data, nilai yang hilang, dan jumlah entri.

    ```python
    df.info()
    ```

2. **Statistik Deskriptif:** Selanjutnya, kami melihat deskripsi statistik dari dataset untuk mendapatkan pemahaman awal tentang distribusi data.

    ```python
    df.describe()
    ```

3. **Ukuran Data:** Kami juga mencatat ukuran dataset, yaitu jumlah baris (entri) dan kolom (fitur).

    ```python
    df.shape
    ```

Eksplorasi data ini membantu kami memahami struktur dan karakteristik data yang akan kami analisis lebih lanjut.

### 3. Rata-Rata Pendapatan per Tahun

Kami ingin mengukur rata-rata pendapatan per tahun, dan untuk itu kami melakukan langkah-langkah berikut:

1. **Mengubah Tanggal Invoice ke Tahun:** Pertama, kami mengubah tipe data kolom 'InvoiceDate' menjadi tipe data datetime untuk memungkinkan kami mengekstrak tahun dari tanggal. Kemudian, kami membuat kolom baru 'Year' yang berisi informasi tahun dari 'InvoiceDate'.

    ```python
    df['InvoiceDate'] = pd.to_datetime(df['InvoiceDate'])
    df['Year'] = df['InvoiceDate'].dt.year
    ```

2. **Filter Data:** Selanjutnya, kami melakukan filter data untuk menghilangkan transaksi dengan nilai negatif pada kolom 'Quantity' dan 'Price'. Kami juga menghapus transaksi dengan kode 'C' yang menandakan pembatalan.

    ```python
    sales = df[(df['Quantity'] > 0) & (df['Price'] > 0) & (~df['Invoice'].str.contains('C'))]
    ```

3. **Membuat Fitur Pendapatan (Revenue):** Kami membuat fitur baru dalam dataset, yaitu kolom 'Revenue', yang dihitung dengan rumus `Quantity * Price` untuk setiap transaksi yang lolos filter.

    ```python
    df['Revenue'] = sales['Quantity'] * sales['Price']
    ```

4. **Menghitung Rata-Rata Pendapatan per Tahun:** Terakhir, kami menghitung rata-rata pendapatan per tahun dengan mengelompokkan data berdasarkan tahun ('Year') dan mengambil rata-rata dari kolom 'Revenue'.

    ```python
    avg = df.groupby('Year')['Revenue'].mean().reset_index()
    ```

Dengan langkah-langkah ini, kami dapat memahami bagaimana pendapatan berubah dari tahun ke tahun dalam bisnis ritel online ini.

### 4. Visualisasi Data

Kami menggunakan Seaborn untuk membuat visualisasi rata-rata pendapatan per tahun dalam bentuk grafik batang. Berikut adalah langkah-langkah yang kami lakukan:

1. **Mengimpor Library Seaborn dan Matplotlib:** Pertama, kami mengimpor library Seaborn dan Matplotlib untuk membantu dalam pembuatan visualisasi.

    ```python
    import seaborn as sns
    import matplotlib.pyplot as plt
    ```

2. **Membuat Grafik Batang Rata-Rata Pendapatan:** Selanjutnya, kami membuat grafik batang yang menunjukkan rata-rata pendapatan per tahun. Kami menggunakan data yang telah dihitung sebelumnya ('avg').

    ```python
    plt.figure(figsize=(10,6))
    ax = sns.barplot(data=avg, x='Year', y='Revenue')
    plt.xlabel('Year')
    plt.ylabel('Average Revenue')
    plt.ylim(0, 25)
    plt.title('Average Revenue per Year')
    plt.show()
    ```

Dengan visualisasi ini, kami dapat dengan jelas melihat bagaimana pendapatan rata-rata berfluktuasi dari tahun ke tahun dalam bisnis ritel online ini. Grafik ini memberikan pemahaman visual tentang tren pendapatan per tahun.

### 5. Jumlah Transaksi Selesai dan Dibatalkan

Kami menghitung jumlah transaksi yang selesai dan dibatalkan, serta membandingkannya per tahun. Berikut adalah langkah-langkah yang kami lakukan:

1. **Customer yang Menyelesaikan Transaksi (Finished Transactions):** Kami memisahkan transaksi yang selesai dari dataset dengan menghilangkan entri yang memiliki nilai null pada kolom 'Customer ID'.

    ```python
    finished = sales.dropna(subset=['Customer ID'])
    ```

2. **Customer yang Membatalkan Transaksi (Canceled Transactions):** Selanjutnya, kami mengidentifikasi transaksi yang dibatalkan dengan mencari entri yang memiliki kode 'C' dalam kolom 'Invoice'.

    ```python
    cancel = df[df['Invoice'].str.contains('C')]
    ```

3. **Menghitung Jumlah Transaksi Selesai dan Dibatalkan per Tahun:** Kami menghitung jumlah transaksi selesai dan dibatalkan per tahun dengan mengelompokkan data berdasarkan tahun ('Year') dan menghitung jumlah entri 'Invoice' untuk masing-masing kategori.

    ```python
    num_finished = finished.groupby('Year')['Invoice'].count().reset_index()
    num_canceled = cancel.groupby('Year')['Invoice'].count().reset_index()
    ```

### 6. Tingkat Pembatalan

Kami menghitung tingkat pembatalan (cancellation rate) dan membandingkannya per tahun. Langkah-langkah yang kami lakukan adalah sebagai berikut:

1. **Menghitung Jumlah Semua Transaksi per Tahun:** Pertama, kami menghitung jumlah semua transaksi per tahun dengan mengelompokkan data berdasarkan tahun ('Year').

    ```python
    all_tr = df.groupby('Year')['Invoice'].count().reset_index()
    ```

2. **Menggabungkan Data Jumlah Pembatalan dengan Data Semua Transaksi:** Kami menggabungkan data jumlah pembatalan ('num_canceled') dengan data jumlah semua transaksi ('all_tr') berdasarkan tahun ('Year').

    ```python
    cancel_rate = pd.merge(num_canceled, all_tr, on='Year', suffixes=('_num_cancel', '_all_tr'))
    ```

3. **Menghitung Tingkat Pembatalan:** Terakhir, kami menghitung tingkat pembatalan dengan membagi jumlah transaksi pembatalan oleh jumlah semua transaksi dan mengalikannya dengan 100.

    ```python
    cancel_rate['cancel_rate'] = cancel_rate['Invoice_num_cancel'] / cancel_rate['Invoice_all_tr'] * 100
    ```

### 7. Visualisasi Tingkat Pembatalan

Kami menggunakan Seaborn untuk membuat visualisasi tingkat pembatalan per tahun dalam bentuk grafik batang. Visualisasi ini memberikan pemahaman visual tentang sejauh mana pembatalan transaksi terjadi dalam bisnis ritel online ini.

```python
plt.figure(figsize=(10,5))
sns.barplot(data=cancel_rate, x='Year', y='cancel_rate')
plt.xlabel('Year')
plt.ylabel('Cancel Rate')
plt.ylim(0, 4)
plt.title('Cancellation Rate per Year')
plt.show()
```
Dengan langkah-langkah ini, kami dapat mengidentifikasi tren pembatalan transaksi dan mengukur tingkat pembatalan dalam bisnis ritel online kami.

## Hasil Analisis Data

### Rata-rata Pendapatan per tahun
![Average Revenue per Year](https://github.com/indhksmaa/Online_Retail_Dataset/blob/main/grafik_average_revenue.png)

Dilihat dari grafik diatas menunjukan bahwa Rata-rata pendapatan tahunan tidak bervariasi secara signifikan dari tahun ke tahun dengan fluktuasi yang relatif kecil, sehingga mengindikasikan bahwa online retail ini stabilitas dalam bisnis. <br>
Perusahaan perlu mempertimbangkan untuk Analisis pelanggan dengan melakukan analisis lebih lanjut mengenai profil pelanggan, preferensi dan umpan balik dapat membantu meningkatkan kepuasan pelanggan dan mengidentifikasi peluang untuk menghasilkan pendapatan tambahan

### Perbandingan Antara Transaksi yang selesai vs Transaksi yang di batalkan
![Comparison_Between_Finished_Transaction_and_Canceled_Transactions](https://github.com/indhksmaa/Online_Retail_Dataset/blob/main/grafik_Comparison%20of%20Finished%20and%20Canceled%20Transactions%20per%20Year.png)

Data menunjukkan bahwa transaksi yang selesai meningkat secara signifikan dari tahun 2009 ke tahun 2010 dari 30754 ke 403067, namun mengalami sedikit penurunan di tahun 2011 menjadi 371728, hal ini menunjukkan mengalami pertumbuhan awal yang signifikan diikuti dengan penurunan sedikit. Dan untuk transaksi yang dibatalkan juga mengalami peningkatan seiring waktu akan tetapi, tidak sebanyak transaksi yang selesai. Hal ini menunjukkan bahwa sebagian besar transaksi telah diselesaikan. tetapi ada beberapa yang dibatalkan. Perusahaan perlu melakukan analisis tren lebih lanjut untuk mengetahui apakah ada faktor musiman atau faktor eksternal tertentu yang mempengaruhi jumlah transaksi yang selesai dan dibatalkan.

### Persentase Transaksi yang dibatalkan
![Cancellation Rate]()


