# Weezy-Sings-Your-Heart-Song
#include <SPI.h>

//Add the SdFat Libraries
#include <SdFat.h>
#include <SdFatUtil.h>

//and the MP3 Shield Library
#include <SFEMP3Shield.h>

// Below is not needed if interrupt driven. Safe to remove if not using.
#if defined(USE_MP3_REFILL_MEANS) && USE_MP3_REFILL_MEANS == USE_MP3_Timer1
  #include <TimerOne.h>
#elif defined(USE_MP3_REFILL_MEANS) && USE_MP3_REFILL_MEANS == USE_MP3_SimpleTimer
  #include <SimpleTimer.h>
#endif

SdFat sd;

/**
 * \brief Object instancing the SFEMP3Shield library.
 *
 * principal object for handling all the attributes, members and functions for the library.
 */
SFEMP3Shield MP3player;
int16_t last_ms_char; // milliseconds of last recieved character from Serial port.
int8_t buffer_pos; // next position to recieve character from Serial port.

  char buffer[6]; // 0-35K+null
int light_current; 
int angle;
int sensorPinPhoto = A0;
int sensorPinPulse = A1;
int pulse;
int command;
int t;

#define trigPin 13
#define echoPin 12
#define led 11
#define ledPin 8


boolean isPlaying = false;

void setup() {
  
  Serial.begin(115200);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(led, OUTPUT);
   // set the digital pin as output:
  pinMode(ledPin, OUTPUT);
 MP3player.setVolume(40, 40);

  uint8_t result; //result code from some function as to be tested at later time.


  
   if(!sd.begin(SD_SEL, SPI_FULL_SPEED)) sd.initErrorHalt();
  // depending upon your SdCard environment, SPI_HAVE_SPEED may work better.
  if(!sd.chdir("/")) sd.errorHalt("sd.chdir");

  //Initialize the MP3 Player Shield
  result = MP3player.begin();
  //check result, see readme for error codes.
  if(result != 0) {
    //Serial.print(F("Error code: "));
   // Serial.print(result);
   // Serial.println(F(" when trying to start MP3 player"));
    if( result == 6 ) {
     // Serial.println(F("Warning: patch file not found, skipping.")); // can be removed for space, if needed.
     // Serial.println(F("Use the \"d\" command to verify SdCard can be read")); // can be removed for space, if needed.
    }
  }

#if (0)
  // Typically not used by most shields, hence commented out.
 // Serial.println(F("Applying ADMixer patch."));
  if(MP3player.ADMixerLoad("admxster.053") == 0) {
   // Serial.println(F("Setting ADMixer Volume."));
    MP3player.ADMixerVol(-3);
  }
#endif

 // help();
  last_ms_char = millis(); // stroke the inter character timeout.
  buffer_pos = 0; // start the command string at zero length.
//  parse_menu('l'); // display the list of files to play


} 

void loop(){
  
  long duration, distance;
  digitalWrite(trigPin, LOW);  // Added this line
  delayMicroseconds(2); // Added this line
  digitalWrite(trigPin, HIGH);
//  delayMicroseconds(1000); - Removed this line
  delayMicroseconds(10); // Added this line
  digitalWrite(trigPin, LOW);
  duration = pulseIn(echoPin, HIGH);
  distance = (duration/2) / 450;
  if (distance < 4) {  // This is where the LED On/Off happens
    digitalWrite(led,HIGH); // When the Red condition is met, the Green LED should turn off

}
  else {
    digitalWrite(led,LOW);
  //  digitalWrite(led2,HIGH);
  }
  if (distance >= 200 || distance <= 0){
    Serial.println("Out of range");
  }
  else {
    Serial.print(distance);
    Serial.println(" cm");
  }
//  delay(500);

  t=random(4);
  light_current = analogRead(sensorPinPhoto);                                                // get current light level
  pulse = analogRead(sensorPinPulse);
  
  if (light_current < 150)      // || (pulse < 'add value here') 
{
  digitalWrite(13, HIGH);
  
   if(!isPlaying) {  //only want to trigger the music playing one time
     playMusic();
   } 
}
  else
  {
    digitalWrite(13, LOW);
  }  
  
    if (!isPlaying) 
    {
        MP3player.stopTrack();
        Serial.println(" Current Light Level: " + String(light_current) + " Pulse value: " + String(pulse));
    }
  }
 //end loop

void playMusic()
{
   Serial.println("PLAY MUSIC");
  
     int command = t; //playing track "1"
     MP3player.playTrack(command);
    
    isPlaying = true;
    
}
