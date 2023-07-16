#include <SPI.h>
#include <MFRC522.h>
#include <SoftwareSerial.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2); // I2C address 0x27, 16 column and 2 rows

SoftwareSerial mySerial(8, 7); //SIM800L Tx & Rx is connected to Arduino #3 & #2

#define SS_PIN 10
#define RST_PIN 5
MFRC522 mfrc522(SS_PIN, RST_PIN);   // Create MFRC522 instance.

void setup()
{
  mySerial.begin(9600);
  Serial.begin(9600);   // Initiate a serial communication
  lcd.init(); // initialize the lcd
  lcd.backlight();

  SPI.begin();      // Initiate  SPI bus
  mfrc522.PCD_Init();   // Initiate MFRC522
  Serial.println("Approximate your card to the reader...");
  Serial.println();
  Serial.println("Initializing...");
  delay(1000);

  mySerial.println("AT"); //Once the handshake test is successful, it will back to OK
  updateSerial();

  lcd.setCursor(0, 0);         // move cursor to   (0, 0)
  lcd.print("   NO-PARKING   ");        // print message at (0, 0)
  lcd.setCursor(0, 1);         // move cursor to   (2, 1)
  lcd.print("      AREA      "); // print message at (2, 1)

}
void loop()
{
  lcd.setCursor(0, 0);         // move cursor to   (0, 0)
  lcd.print("   NO-PARKING   ");        // print message at (0, 0)
  lcd.setCursor(0, 1);         // move cursor to   (2, 1)
  lcd.print("      AREA      "); // print message at (2, 1)

  // Look for new cards
  if ( ! mfrc522.PICC_IsNewCardPresent())
  {
    return;
  }
  // Select one of the cards
  if ( ! mfrc522.PICC_ReadCardSerial())
  {
    return;
  }
  //Show UID on serial monitor
  Serial.print("UID tag :");
  String content = "";
  byte letter;
  for (byte i = 0; i < mfrc522.uid.size; i++)
  {
    Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
    Serial.print(mfrc522.uid.uidByte[i], HEX);
    content.concat(String(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " "));
    content.concat(String(mfrc522.uid.uidByte[i], HEX));
  }
  Serial.println();
  Serial.print("Message : ");
  content.toUpperCase();
  if (content.substring(1) == "33 62 C2 94" || content.substring(1) == "33 56 27 95") //change here the UID of the card/cards that you want to give access
  {
    lcd.setCursor(0, 0);         // move cursor to   (0, 0)
    lcd.print("   VEHICLE IN   ");        // print message at (0, 0)
    lcd.setCursor(0, 1);         // move cursor to   (2, 1)
    lcd.print("NO-PARKING ZONE "); // print message at (2, 1)


    mySerial.println("AT"); //Once the handshake test is successful, it will back to OK
    updateSerial();
    mySerial.println("AT+CMGF=1"); // Configuring TEXT mode
    updateSerial();
    mySerial.println("AT+CMGS=\"+918141195455\"");//change ZZ with country code and xxxxxxxxxxx with phone number to sms
    updateSerial();
    mySerial.print("Warning !!! you have parked your vehicle in no-parking zone, move it immediately or you will get fined"); //text content
    updateSerial();
    mySerial.write(26);
    Serial.println("Unauthorized access");
    Serial.println();
    delay(3000);
  }

  else   {
    Serial.println("Authorized denied");

    lcd.setCursor(0, 0);         // move cursor to   (0, 0)
    lcd.print("   NO-PARKING   ");        // print message at (0, 0)
    lcd.setCursor(0, 1);         // move cursor to   (2, 1)
    lcd.print("      AREA      "); // print message at (2, 1)
  }
}
void updateSerial()
{
  delay(500);
  while (Serial.available())
  {
    mySerial.write(Serial.read());//Forward what Serial received to Software Serial Port
  }
  while (mySerial.available())
  {
    Serial.write(mySerial.read());//Forward what Software Serial received to Serial Port
  }
}
