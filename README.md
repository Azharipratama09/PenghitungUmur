# PenghitungUmur
 Latihan 2 -Muhammad Azhari Nur Pratama-2210010326

## Deskripsi
Aplikasi ini dibuat menggunakan Java class dan jFrame From sebagai tampilan antarmuka (GUI) untuk menghitung hari ulang tahun disatu tahun kedepan dan mencari peristiwa penting pada tanggal ulang tahun. Proyek ini dibuat sebagai bagian dari tugas pada mata kuliah Pemrograman Berorientasi Objek 2.

## 1. Deskripsi Program:
   
   • Pengguna memilih tanggal lahir dari JDateChooser

   • Setelah menekan tombol Hitung,kemudian akan umur dalam tahun, bulan, dan hari 

## 2. Komponen GUI: JFrame, JPanel, JLabel, JDateChooser, JButton,JTextField

## 3. Logika Program: Manipulasi tanggal menggunakan LocalDate,
Perhitungan selisih waktu

## 4. Events:

   • ActionListener untuk tombol Hitung
~~~
 private void jButton1ActionPerformed(java.awt.event.ActionEvent evt) {                                         
      Date tanggalLahir = jDateChooser1.getDate();
    if (tanggalLahir != null) {
    // Menghitung umur dan hari ulang tahun berikutnya
    LocalDate lahir = tanggalLahir.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();
    LocalDate sekarang = LocalDate.now();
    String umur = helper.hitungUmurDetail(lahir, sekarang);
    jTextField1.setText(umur);
    // Menghitung tanggal ulang tahun berikutnya
    LocalDate ulangTahunBerikutnya = helper.hariUlangTahunBerikutnya(lahir, sekarang);
    String hariUlangTahunBerikutnya = helper.getDayOfWeekInIndonesian(ulangTahunBerikutnya);
    
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy");
String tanggalUlangTahunBerikutnya = ulangTahunBerikutnya.format(formatter);
TESS.setText(hariUlangTahunBerikutnya + " (" + tanggalUlangTahunBerikutnya + ")");

// Set stop flag untuk thread sebelumnya
stopFetching = true;
if (peristiwaThread != null && peristiwaThread.isAlive()) {
    peristiwaThread.interrupt(); // Beri sinyal ke thread untuk berhenti
}

// Reset flag untuk thread baru
stopFetching = false;

// Mendapatkan peristiwa penting secara asinkron
peristiwaThread = new Thread(() -> {
    try {
        txtAreaPeristiwa.setText("Tunggu, sedang mengambil data...\n");
        helper.getPeristiwaBarisPerBaris(ulangTahunBerikutnya,
txtAreaPeristiwa, () -> stopFetching);
        if (!stopFetching) {
            javax.swing.SwingUtilities.invokeLater(() ->
txtAreaPeristiwa.append("Selesai mengambil data peristiwa"));
        }
    } catch (Exception e) {
        if (Thread.currentThread().isInterrupted()) {
            javax.swing.SwingUtilities.invokeLater(() ->
txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
        }

    }
});
peristiwaThread.start();

        }
    }      
~~~
   • PropertyChangeListener pada JDateChooser untuk deteksi perubahan tanggal
~~~
 private void jDateChooser1PropertyChange(java.beans.PropertyChangeEvent evt) {                                             
    jTextField1.setText("");
    TESS.setText("");
    // Hentikan thread yang sedang berjalan saat tanggal lahir berubah
stopFetching = true;
if (peristiwaThread != null && peristiwaThread.isAlive()) {
    peristiwaThread.interrupt();
}
txtAreaPeristiwa.setText("");
    }   
~~~
5. Variasi:

   • Sediakan informasi tambahan seperti hari ulang tahun berikutnya
~~~
public String hitungUmurDetail(LocalDate lahir, LocalDate sekarang) {
Period period = Period.between(lahir, sekarang);
return period.getYears() + " tahun, " + period.getMonths() + "bulan, " + period.getDays() + " hari";
}
// Menghitung hari ulang tahun berikutnya
public LocalDate hariUlangTahunBerikutnya(LocalDate lahir, LocalDate
sekarang) {
LocalDate ulangTahunBerikutnya =
lahir.withYear(sekarang.getYear());
if (!ulangTahunBerikutnya.isAfter(sekarang)) {
ulangTahunBerikutnya = ulangTahunBerikutnya.plusYears(1);
}
return ulangTahunBerikutnya;
}
// Menerjemahkan teks hari ke bahasa Indonesia
public String getDayOfWeekInIndonesian(LocalDate date) {
switch (date.getDayOfWeek()) {
case MONDAY:
return "Senin";
case TUESDAY:
return "Selasa";
case WEDNESDAY:
return "Rabu";
case THURSDAY:
return "Kamis";
case FRIDAY:
return "Jumat";
case SATURDAY:
return "Sabtu";
case SUNDAY:
return "Minggu";
default:
return "";


}
}
~~~
   • Integrasikan dengan API eksternal untuk menampilkan peristiwa penting pada tanggal lahir
~~~
public void getPeristiwaBarisPerBaris(LocalDate tanggal, JTextArea txtAreaPeristiwa, Supplier<Boolean> shouldStop) {
     try {
// Periksa jika thread seharusnya dihentikan sebelum dimulai
 if (shouldStop.get()) {
        return;
}
    String urlString = "https://byabbe.se/on-this-day/" +
tanggal.getMonthValue() + "/" + tanggal.getDayOfMonth() + "/events.json";
    URL url = new URL(urlString);
    HttpURLConnection conn = (HttpURLConnection)
url.openConnection();
    conn.setRequestMethod("GET");
    conn.setRequestProperty("User-Agent", "Mozilla/5.0");
    
    int responseCode = conn.getResponseCode();
    if (responseCode != 200) {
        throw new Exception("HTTP response code: " + responseCode + ". Silakan coba lagi nanti atau cek koneksi internet.");
}
    BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
    String inputLine;
    StringBuilder content = new StringBuilder();
    while ((inputLine = in.readLine()) != null) {
    // Periksa jika thread seharusnya dihentikan saat membaca data

    if (shouldStop.get()) {
        in.close();
        conn.disconnect();
        javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
return;
}
            content.append(inputLine);
}
            in.close();
            conn.disconnect();
            JSONObject json = new JSONObject(content.toString());
            JSONArray events = json.getJSONArray("events");
            for (int i = 0; i < events.length(); i++) {
// Periksa jika thread seharusnya dihentikan sebelum memproses data
    if (shouldStop.get()) {
        javax.swing.SwingUtilities.invokeLater(() ->
        txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
return;
}
    JSONObject event = events.getJSONObject(i);
    String year = event.getString("year");       
    String description = event.getString("description");
        String translateddescription = translateToIndonesian(description);        
        String peristiwa = year + ": " + translateddescription;
    javax.swing.SwingUtilities.invokeLater(() ->
    txtAreaPeristiwa.append(peristiwa + "\n"));
}
    if (events.length() == 0) {
    javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Tidak ada peristiwa penting yang ditemukan pada tanggal ini."));
}
    } catch (Exception e) {
    javax.swing.SwingUtilities.invokeLater(() ->
    txtAreaPeristiwa.setText("Gagal mendapatkan data peristiwa: " +
    e.getMessage()));
}
   }public void getPeristiwaBarisPerBaris(LocalDate tanggal, JTextArea txtAreaPeristiwa, Supplier<Boolean> shouldStop) {
     try {
// Periksa jika thread seharusnya dihentikan sebelum dimulai
 if (shouldStop.get()) {
        return;
}
    String urlString = "https://byabbe.se/on-this-day/" +
tanggal.getMonthValue() + "/" + tanggal.getDayOfMonth() + "/events.json";
    URL url = new URL(urlString);
    HttpURLConnection conn = (HttpURLConnection)
url.openConnection();
    conn.setRequestMethod("GET");
    conn.setRequestProperty("User-Agent", "Mozilla/5.0");
    
    int responseCode = conn.getResponseCode();
    if (responseCode != 200) {
        throw new Exception("HTTP response code: " + responseCode + ". Silakan coba lagi nanti atau cek koneksi internet.");
}
    BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
    String inputLine;
    StringBuilder content = new StringBuilder();
    while ((inputLine = in.readLine()) != null) {
    // Periksa jika thread seharusnya dihentikan saat membaca data

    if (shouldStop.get()) {
        in.close();
        conn.disconnect();
        javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
return;
}
            content.append(inputLine);
}
            in.close();
            conn.disconnect();
            JSONObject json = new JSONObject(content.toString());
            JSONArray events = json.getJSONArray("events");
            for (int i = 0; i < events.length(); i++) {
// Periksa jika thread seharusnya dihentikan sebelum memproses data
    if (shouldStop.get()) {
        javax.swing.SwingUtilities.invokeLater(() ->
        txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
return;
}
    JSONObject event = events.getJSONObject(i);
    String year = event.getString("year");       
    String description = event.getString("description");
        String translateddescription = translateToIndonesian(description);        
        String peristiwa = year + ": " + translateddescription;
    javax.swing.SwingUtilities.invokeLater(() ->
    txtAreaPeristiwa.append(peristiwa + "\n"));
}
    if (events.length() == 0) {
    javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Tidak ada peristiwa penting yang ditemukan pada tanggal ini."));
}
    } catch (Exception e) {
    javax.swing.SwingUtilities.invokeLater(() ->
    txtAreaPeristiwa.setText("Gagal mendapatkan data peristiwa: " +
    e.getMessage()));
}
   }
~~~

## Contoh Gambar Project Setelah di Run
![](https://github.com/Azharipratama09/PenghitungUmur/blob/main/Cuplikan%20layar%202024-10-31%20143205.png)
 
## Indikator Penilaian:

| No  | Komponen         |  Persentase  |
| :-: | --------------   |   :-----:    |
|  1  | Komponen GUI     |    10    |
|  2  | Logika Program   |    10    |
|  3  | Event    |    20    |
|  4  |Kesesuaian UI      |    10    |
|  5  | Memenuhi Variasi |    50    |
|     | *TOTAL*        | *100* |

## Pembuat
Nama  : Muhammad Azhari Nur Pratama
NPM   : 2210010326
