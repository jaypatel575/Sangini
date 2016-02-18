Sangini:
SMS based Instant Information System: An SMS enabled information system which gives all relevant details of a particular city including emergency numbers and safe places to stay. 

Hardware Side Server Code Of Arduino with Fona GSM Module.

How It Work??

In this system when user want to go any places(e.g.Rajkot) then he/she just send the message(Rajkot) to the server number and server 
replies him/her with all the use full information of place like Place to stay, place to visit, food stall, shopping malls etc. in the form of simple text message. 


CODE:

      /*
      THIS CODE IS MADE BY BINARYBROS!
      
      Open up the serial console on the Arduino at 115200 baud to interact with FONA.
      
      SMS based Instant Information System : An SMS enabled information system which gives all relevant details of a particular city        including emergency numbers and safe places to stay. 
      
      This code will receive an SMS, and send information which gives all relevant details of a particular city including emergency         numbers and safe places to stay.
      
      For use with FONA 800 & 808.
      */
      
      #include "Adafruit_FONA.h"
      
      #define FONA_RX 2 //connect Fona Rx pin with digital pin 2 of arduino.
      #define FONA_TX 3 //connect Fona Tx pin with digital pin 3 of arduino.
      #define FONA_RST 4 //connect Fona RST pin with digital pin 4 of arduino.
      
      // this is a large buffer for replies
      char replybuffer[255];

    // We default to using software serial. If you want to use hardware serial
    // (because softserial isnt supported) comment out the following three lines 
    // and uncomment the HardwareSerial line
    #include <SoftwareSerial.h>
    SoftwareSerial fonaSS = SoftwareSerial(FONA_TX, FONA_RX);
    SoftwareSerial *fonaSerial = &fonaSS;

    // Hardware serial is also possible!
    //  HardwareSerial *fonaSerial = &Serial1;

    Adafruit_FONA fona = Adafruit_FONA(FONA_RST);

    uint8_t readline(char *buff, uint8_t maxbuff, uint16_t timeout = 0);

    void setup() {
    while (!Serial);

    Serial.begin(115200);
    Serial.println(F("FONA SMS caller ID test"));
    Serial.println(F("Initializing....(May take 3 seconds)"));

    // make it slow so its easy to read!
    fonaSerial->begin(4800);
    if (! fona.begin(*fonaSerial)) {
    Serial.println(F("Couldn't find FONA"));
    while(1);
    }
    Serial.println(F("FONA is OK"));
    }

  
    char fonaInBuffer[64];          //for notifications from the FONA

    void loop() {
  
    char* bufPtr = fonaInBuffer;    //handy buffer pointer
  
    if (fona.available())      //Check any data available from the FONA?
    {
    int slot = 0;            //this will be the slot number of the SMS
    int charCount = 0;
    
    //Read the notification into fonaInBuffer
    do  {
      *bufPtr = fona.read();
      Serial.write(*bufPtr);
      delay(1);
    }
    while ((*bufPtr++ != '\n') && (fona.available()) && (++charCount < (sizeof(fonaInBuffer)-1)));
    
    //Add a terminal NULL to the notification string
    *bufPtr = 0;
    
    //Scan the notification string for an SMS received notification.
    //  If it's an SMS message, we'll get the slot number in 'slot'
    if (1 == sscanf(fonaInBuffer, "+CMTI: \"SM\",%d", &slot)) {
      Serial.print("slot: "); Serial.println(slot);
      
      char callerIDbuffer[32];  //we'll store the SMS sender number in here
      
      // Retrieve SMS sender address/phone number.
      if (! fona.getSMSSender(slot, callerIDbuffer, 31)) {
        Serial.println("Didn't find SMS message in slot!");
      }
      Serial.print(F("FROM: ")); Serial.println(callerIDbuffer);
      
      //Send back an automatic response
      Serial.println("Sending reponse...");
      // All Information of Places.
      if (!fona.sendSMS(callerIDbuffer, "WELCOME TO RAJKOT,GUJARAT.....                                             PLACE TO VISIT:                                  1. BIZZ THE HOTEL                      www.bizzhotel.in                     0281-2473000              PALACE TO VISIT:                          1. GONDAL FORT HOSPITALS:                         1. CIVIL HOSPITAL                           0281-2471118                             PLACE TO SHOPPING:                          1.BIG BAZAR - 150,FEET RING ROAD, RAJKOT.                     TESTY FOOD:                               1. PREMVATI - SWAMINARAYAN TEMPLE, KALAWAD ROAD,RAJKOT.                                TRAVELLING:                                        1. BUS STATION 0281-2235025                             EMERGENCY CONTACTS:                     POLICE - 100                               FIRE SERVICE - 101")) {
        Serial.println(F("Failed"));
      } else {
        Serial.println(F("Sent!"));
      }
      
      // delete the original msg after it is processed
      //   otherwise, we will fill up all the slots
      //   and then we won't be able to receive SMS anymore
      if (fona.deleteSMS(slot)) {
        Serial.println(F("OK!"));
      } else {
        Serial.println(F("Couldn't delete"));
      }
    }
  }
}
