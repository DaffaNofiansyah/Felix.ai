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

	Kode tanpa fitur output speaker
	```
   	#include <SoftwareSerial.h>
	#include "VoiceRecognitionV3.h"
	#include <SD.h>
	#include <TMRpcm.h>
	
	VR myVR(2, 3); // 2:RX 3:TX, you can choose your favourite pins.
	TMRpcm tmrpcm;
	
	uint8_t record[7]; // save record
	uint8_t buf[64];
	
	int led = 13;
	
	int group = 0;
	
	#define Felix (0)
	
	#define commandMusic (1)
	#define commandLight (2)
	#define commandSelf (3)
	#define commandClock (4)
	#define commandDate (5)
	#define commandBack (6)
	
	#define musicLofi (10)
	#define musicRock (11)
	#define musicHiphop (12)
	#define musicStop (13)
	#define musicPlay (14)
	
	#define lightsOn (20)
	#define lightsOff (21)
	
	#define selfName (30)
	#define selfRelation (31)
	#define selfUser (32)
	#define selfThank (33)
	
	boolean ack;
	
	void setup()
	{
	  /** initialize */
	  myVR.begin(9600);
	
	  Serial.begin(115200);
	  Serial.println("Elechouse Voice Recognition V3 Module\r\nMulti Commands sample");
	
	  pinMode(led, OUTPUT);
	
	  if (myVR.clear() == 0)
	  {
	    Serial.println("Recognizer cleared.");
	  }
	  else
	  {
	    Serial.println("Not find VoiceRecognitionModule.");
	    Serial.println("Please check connection and restart Arduino.");
	    while (1)
	      ;
	  }
	
	  record[0] = Felix;
	  record[1] = commandMusic;
	  record[2] = commandLight;
	  record[3] = commandSelf;
	  record[4] = commandClock;
	  record[5] = commandDate;
	  record[6] = commandBack;
	  group = 0;
	  if (myVR.load(record, 7) >= 0)
	  {
	    printRecord(record, 7);
	    Serial.println(F("loaded."));
	  }
	
	  // Initialize SD card
	  if (!SD.begin(4)) // Pin 4 is connected to the SD card module's CS pin
	  {
	    Serial.println("SD card initialization failed!");
	    while (1);
	  }
	
	  // Set up the tmrpcm library
	  tmrpcm.speakerPin = 9; // Pin 9 is connected to the positive terminal of the speaker
	}
	
	void loop()
	{
	  int ret;
	  ret = myVR.recognize(buf, 50);
	  if (ret > 0)
	  {
	    switch (buf[1])
	    {
	    case Felix:
	      break;
	    case commandMusic:
	      felixMusic();
	      defaultSettings();
	      break;
	    case commandLight:
	      felixLight();
	      defaultSettings();
	      break;
	    case commandSelf:
	      felixSelf();
	      defaultSettings();
	      break;
	    case commandDate:
	      break;
	    case commandClock:
	      break;
	    case commandBack:
	      break;
	    default:
	      break;
	    }
	    /** voice recognized */
	    printVR(buf);
	  }
	}
	
	void printSignature(uint8_t *buf, int len)
	{
	  int i;
	  for (i = 0; i < len; i++)
	  {
	    if (buf[i] > 0x19 && buf[i] < 0x7F)
	    {
	      Serial.write(buf[i]);
	    }
	    else
	    {
	      Serial.print("[");
	      Serial.print(buf[i], HEX);
	      Serial.print("]");
	    }
	  }
	}
	
	void printVR(uint8_t *buf)
	{
	  Serial.println("VR Index\tGroup\tRecordNum\tSignature");
	
	  Serial.print(buf[2], DEC);
	  Serial.print("\t\t");
	
	  if (buf[0] == 0xFF)
	  {
	    Serial.print("NONE");
	  }
	  else if (buf[0] & 0x80)
	  {
	    Serial.print("UG ");
	    Serial.print(buf[0] & (~0x80), DEC);
	  }
	  else
	  {
	    Serial.print("SG ");
	    Serial.print(buf[0], DEC);
	  }
	  Serial.print("\t");
	
	  Serial.print(buf[1], DEC);
	  Serial.print("\t\t");
	  if (buf[3] > 0)
	  {
	    printSignature(buf + 4, buf[3]);
	  }
	  else
	  {
	    Serial.print("NONE");
	  }
	  Serial.println();
	}
	
	void printRecord(uint8_t *buf, uint8_t len)
	{
	  Serial.print(F("Record: "));
	  for (int i = 0; i < len; i++)
	  {
	    Serial.print(buf[i], DEC);
	    Serial.print(", ");
	  }
	}
	
	void defaultSettings()
	{
	  myVR.clear();
	  record[0] = Felix;
	  record[1] = musicLofi;
	  record[2] = musicRock;
	  record[3] = musicHiphop;
	  record[4] = musicStop;
	  record[5] = musicPlay;
	  record[6] = commandBack;
	  if (myVR.load(record, 7) >= 0)
	  {
	    printRecord(record, 7);
	    Serial.println(F("loaded."));
	  }
	}
	
	void felixMusic()
	{
	  myVR.clear();
	  record[0] = Felix;
	  record[1] = musicLofi;
	  record[2] = musicRock;
	  record[3] = musicHiphop;
	  record[4] = musicStop;
	  record[5] = musicPlay;
	  record[6] = commandBack;
	  if (myVR.load(record, 7) >= 0)
	  {
	    printRecord(record, 7);
	    Serial.println(F("loaded."));
	  }
	  ack = true;
	  while (ack == true)
	  {
	    ret = myVR.recognize(buf, 50);
	    if (ret > 0)
	    {
	      switch (buf[1])
	      {
	      case musicLofi:
	        playWavFile("lofi.wav");
	        break;
	      case musicRock:
	        playWavFile("rock.wav");
	        break;
	      case musicHiphop:
	        playWavFile("hiphop.wav");
	        break;
	      case musicStop:
	        tmrpcm.stopPlayback();
	        break;
	      case commandBack:
	        ack = false;
	        break;
	      default:
	        break;
	      }
	    }
	  }
	}
	
	void felixLight()
	{
	  myVR.clear();
	  record[0] = Felix;
	  record[1] = lightsOn;
	  record[2] = lightsOff;
	  record[3] = commandSelf;
	  record[4] = commandClock;
	  record[5] = commandDate;
	  record[6] = commandBack;
	  if (myVR.load(record, 7) >= 0)
	  {
	    printRecord(record, 7);
	    Serial.println(F("loaded."));
	  }
	  ack = true;
	  while (ack == true)
	  {
	    ret = myVR.recognize(buf, 50);
	    if (ret > 0)
	    {
	      switch (buf[1])
	      {
	      case lightsOn:
	        // Code to turn on lights
	        break;
	      case lightsOff:
	        // Code to turn off lights
	        break;
	      case commandBack:
	        ack = false;
	        break;
	      default:
	        break;
	      }
	    }
	  }
	}
	
	void felixSelf()
	{
	  myVR.clear();
	  record[0] = Felix;
	  record[1] = selfName;
	  record[2] = selfRelation;
	  record[3] = selfUser;
	  record[4] = selfThank;
	  record[5] = commandDate;
	  record[6] = commandBack;
	  if (myVR.load(record, 7) >= 0)
	  {
	    printRecord(record, 7);
	    Serial.println(F("loaded."));
	  }
	  ack = true;
	  while (ack == true)
	  {
	    ret = myVR.recognize(buf, 50);
	    if (ret > 0)
	    {
	      switch (buf[1])
	      {
	      case selfName:
	        playWavFile("name.wav");
	        break;
	      case selfRelation:
	        playWavFile("relation.wav");
	        break;
	      case selfUser:
	        playWavFile("user.wav");
	        break;
	      case selfThank:
	        playWavFile("thanks.wav");
	        break;
	      case commandBack:
	        ack = false;
	        break;
	      default:
	        break;
	      }
	    }
	  }
	}
	
	void playWavFile(const char *filename)
	{
	  // Open the file
	  File myFile = SD.open(filename);
	  
	  if (myFile)
	  {
	    tmrpcm.play(myFile); // Play the file
	    myFile.close();      // Close the file
	  }
	  else
	  {
	    Serial.println("Error opening file");
	  }
	}
   	```

 	Kode dengan fitur output speaker
	```   	
	#include <SoftwareSerial.h>
	#include "VoiceRecognitionV3.h"
	
	VR myVR(8, 9);    // 8:TX 9:RX, you can choose your favorite pins.
	
	const int redPin = 3;    // Pin for the red LED
	const int greenPin = 10;  // Pin for the green LED
	const int bluePin = 13;   // Pin for the blue LED
	int8_t records[7]; // save record
	uint8_t buf[64];
	
	int led = 13;
	
	#define onRecord    (0)
	#define offRecord   (1) 
	
	/**
	  @brief   Print signature, if the character is invisible, 
	           print hexible value instead.
	  @param   buf     --> command length
	           len     --> number of parameters
	*/
	
	#define lightsOn (20)
	#define lightsOff (21)
	#define lightsRed (23)
	#define lightsGreen (24)
	#define lightsBlue (22)
	#define lightsYellow (25)
	
	void printSignature(uint8_t *buf, int len)
	{
	  int i;
	  for(i=0; i<len; i++){
	    if(buf[i]>0x19 && buf[i]<0x7F){
	      Serial.write(buf[i]);
	    }
	    else{
	      Serial.print("[");
	      Serial.print(buf[i], HEX);
	      Serial.print("]");
	    }
	  }
	}
	
	/**
	  @brief   Print signature, if the character is invisible, 
	           print hexible value instead.
	  @param   buf  -->  VR module return value when voice is recognized.
	             buf[0]  -->  Group mode(FF: None Group, 0x8n: User, 0x0n:System
	             buf[1]  -->  number of record which is recognized. 
	             buf[2]  -->  Recognizer index(position) value of the recognized record.
	             buf[3]  -->  Signature length
	             buf[4]~buf[n] --> Signature
	*/
	void printVR(uint8_t *buf)
	{
	  Serial.println("VR Index\tGroup\tRecordNum\tSignature");
	
	  Serial.print(buf[2], DEC);
	  Serial.print("\t\t");
	
	  if(buf[0] == 0xFF){
	    Serial.print("NONE");
	  }
	  else if(buf[0]&0x80){
	    Serial.print("UG ");
	    Serial.print(buf[0]&(~0x80), DEC);
	  }
	  else{
	    Serial.print("SG ");
	    Serial.print(buf[0], DEC);
	  }
	  Serial.print("\t");
	
	  Serial.print(buf[1], DEC);
	  Serial.print("\t\t");
	  if(buf[3]>0){
	    printSignature(buf+4, buf[3]);
	  }
	  else{
	    Serial.print("NONE");
	  }
	  Serial.println("\r\n");
	}
	
	void setup()
	{
	  /** initialize */
	  myVR.begin(9600);
	
	  Serial.begin(115200);
	  Serial.println("Elechouse Voice Recognition V3 Module\r\nControl LED sample");
	
	  pinMode(redPin, OUTPUT);
	  pinMode(greenPin, OUTPUT);
	  pinMode(bluePin, OUTPUT);
	
	  if (myVR.clear() == 0)
	  {
	    Serial.println("Recognizer cleared.");
	  }
	  else
	  {
	    Serial.println("Not find VoiceRecognitionModule.");
	    Serial.println("Please check connection and restart Arduino.");
	    while (1)
	      ;
	  }
	
	  if (myVR.load((uint8_t)lightsOn) >= 0)
	  {
	    Serial.println("lightsOn loaded");
	  }
	
	  if (myVR.load((uint8_t)lightsOff) >= 0)
	  {
	    Serial.println("lightsOff loaded");
	  }
	
	  if (myVR.load((uint8_t)lightsRed) >= 0)
	  {
	    Serial.println("lightsRed loaded");
	  }
	
	  if (myVR.load((uint8_t)lightsGreen) >= 0)
	  {
	    Serial.println("lightsGreen loaded");
	  }
	
	  if (myVR.load((uint8_t)lightsBlue) >= 0)
	  {
	    Serial.println("lightsBlue loaded");
	  }
	
	  if (myVR.load((uint8_t)lightsYellow) >= 0)
	  {
	    Serial.println("lightsYellow loaded");
	  }
	}
	
	void loop()
	{
	  int ret;
	  ret = myVR.recognize(buf, 1500);
	
	  if (ret > 0)
	  {
	    switch (buf[1])
	    {
	    case lightsOn:
	      setColor(255, 255, 255);
	      delay(1000); // Add a delay after turning on lights
	      break;
	    case lightsOff:
	      setColor(0, 0, 0);
	      delay(1000); // Add a delay after turning off lights
	      break;
	    case lightsRed:
	      setColor(255, 0, 0);
	      delay(1000); // Add a delay after setting lights to red
	      break;
	    case lightsGreen:
	      setColor(0, 255, 0);
	      delay(1000); // Add a delay after setting lights to green
	      break;
	    case lightsBlue:
	      setColor(0, 0, 255);
	      delay(1000); // Add a delay after setting lights to blue
	      break;
	    case lightsYellow:
	      setColor(255, 150, 200);
	      delay(1000); // Add a delay after setting lights to yellow
	      break;
	    default:
	      break;
	    }
	  }
	}
	
	void setColor(int red, int green, int blue)
	{
	  analogWrite(redPin, red);
	  analogWrite(greenPin, green);
	  analogWrite(bluePin, blue);
	}
   	```
      
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
