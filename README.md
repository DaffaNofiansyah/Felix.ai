# Deskripsi

Felix.ai merupakan asisten rumah tangga berbasis pengenalan suara. Proyek ini menggunakan *Arduino Microcontroller* dengan *Voice Recognition Module V3*.


# Cara Penggunaan

1. ### Melakukan instalasi terhadap arduino IDE dan `  menginstall [Voice Recognition V3](https://www.elechouse.com/product/speak-recognition-voice-recognition-module-v3/)
    

	Melakukan instalasi aplikasi arduino IDE pada [link ini](https://www.arduino.cc/en/software). Kemudian mendownload file zip voice recognition pada [link ini](https://www.elechouse.com/product/speak-recognition-voice-recognition-module-v3/). setelah itu exact file zip tersebut ke folder libraries di bagian document. Contoh pathnya adalah kurang lebih sebagai berikut : C:\Users\ASUS\Documents\Arduino\libraries.

2.  ### Melakukan konfigurasi terhadap arduino
	Sambungkan voice recognition module dengan Arduino, sebagai berikut:

	1.  Hubungkan pin VCC Voice Recognition Module V3 dengan pin VCC 5V Arduino.
	2.  Hubungkan pin GND Voice Recognition Module V3 dengan pin GND Arduino.
	3.  Hubungkan pin RXD Voice Recognition Module V3 dengan pin digital 2 Arduino.
	4.  Hubungkan pin TXD Voice Recognition Module V3 dengan pin digital 3 Arduino.

	Konfigurasi train pada voice recognition module, buka file vr_sample_train.ino pada “File →Examples→VoiceRecognitionV3→vr_sample_train”, lalu pilih board Arduino Uno pada “Tool→Board”, dan pilih port serial. Klik tombol Upload dan tunggu proses selesai. Buka Serial Monitor, atur baud rate pada 115200, dan send dengan Newline atau Both NL & CR.
	
	Selanjutnya melakukan train pada voice recognition module. Ketik train 0  sebagai command untuk melakukan train pada record 0. Ketika Serial Monitor mencetak Speak now, ucapkan kalimat yang sesuai dengan fungsi record tersebut, dan ketika Serial Monitor mencetak Speak again, ulangi kalimat tersebut. Jika kedua suara yang diterima dianggap cocok, Serial monitor akan mencetak Success, dan record 0 telah di-train. Jika suara yang diterima dianggap tidak cocok, ulangi kalimat hingga berhasil.

3.  ### Implementasi kode

	Pertama dilakukan inisiasi variabel konstan dan Voice Recognition itu sendiri. Di buat array record yang digunakan untuk menyimpan suaranya dan array buffer.

	1.  fungsi void setup() adalah fungsi awal untuk melakukan inisiasi. dilakukan penyetelan record[i] untuk masing-masing command (Felix, commandMusic, commandLight, commandSelf, commandClock, commandDate, commandBack).
	    
	2.  fungsi void loop() adalah fungsi yang terus menerus dijalankan. fungsi tersebut mulanya akan mendengarkan suara dari pengguna. Dari suara tersebut akan ditentukan termasuk suara yang mana. Terdapat beberapa kasus, yaitu untuk kasus Felix, commandMusic, commandLight, commandSelf, commandClock, commandDate, commandBack. Untuk masing-masing kasus akan dijalankan fungsi yang bersangkutan (misalnya untuk commandMusic, akan dijalankan fungsi felixMusic() yang dimana akan menanyakan musik apa yang ingin dijalankan) setelah itu dijalankan fungsi defaultSettings() yang akan mengambilkan fungsi ke setelan semula.
	    
	3.  fungsi void felixMusic() adalah fungsi untuk menjalankan lagu. pertama, myVR akan diclear dan array recordnya diubah berdasarkan musik yang diinginkan. terdapat 7 jenis command, yaitu Felix, musicLofi, musicRock, musicHiphop, musicStop, musicPlay, dan commandBack. Kemudian akan meminta pengguna untuk berbicara lagi untuk menentukan jenis musiknya. Sesuai dengan perintahnya, musicLofi akan menjalankan file lofi.wav, musicRock akan menjalankan file rock.wav, musicHiphop akan menjalankan file hiphop.wav, musicStop akan menghentikan playback, dan commandBack untuk kembeli ke fungsi utama, yakni fungsi loop().
	    
	4.  fungsi void felixLight() adalah fungsi untuk mengatur lampu. pertama, myVR akan diclear dan array recordnya diubah berdasarkan musik yang diinginkan. record akan diisi dengan lightson, dan lightsoff, dan commandBack. Kemudian akan meminta pengguna berbicara lagi untuk menyalakan lampu, mematikan lampu, atau kembali.
	    
	5.  fungsi void felixSelf() adalah fungsi untuk menjelaskan identitas felix yang merupakan voice recognition itu sendiri. pertama, myVR akan diclear dan array recordnya diubah berdasarkan musik yang diinginkan. record akan diisi dengan Felix, selfName, selfRelation, selfUser, selfThank, dan commandBack. Kemudian akan meminta pengguna berbicara lagi untuk bertanya tentang nama felix, hubungan felix, pengguna felix, terima kasih felix, dan kembali. untuk masing-masing kasus akan menjalankan file yang berbeda-beda.
	    
	6.  fungsi void defaultSettings() berfungsi untuk mengubah setelah ke semula. variable record tadi diubah-ubah pada fungsi felixMusic(), felixLight(), dan felixSelf(). agar record tetap sama ke setingan awal yang ada pada fungsi setup(), maka record[i] akan diubah ke setingan awal (Felix, commandMusic, commandLight, commandSelf, commandClock, commandDate, commandBack).


# Anggota Kelompok
|Nama| NIM|
|--|--|
|Daffa Nofiansyah|G6401211098|
|Steven Hesang|G6401211098|
|Dharma Pratama|G6401211098|
|Ronald Abner|G6401211098|
