#include <Adafruit_NeoPixel.h>
#include <SoftwareSerial.h>

#define LED_PIN 6
#define SLEEP_PIN 5
#define NUMPIXELS 16
Adafruit_NeoPixel pixels(NUMPIXELS, LED_PIN, NEO_GRB + NEO_KHZ800);

// AQI PM2.5 Levels
#define AQI_GOOD_MAX 50
#define AQI_MODERATE_MAX 100
#define AQI_UNHEALTHY_SG_MAX 150
#define AQI_UNHEALTHY_MAX 200
#define AQI_VERY_UNHEALTHY_MAX 300
// Last color is used to prevent setting the same color repeatedly
int last_color = -1; // Initialize to a value that won't match any color index

#define LENG 31 // 0x42 + 31 bytes equal to 32 bytes
unsigned char buf[LENG];

SoftwareSerial SoftSerial(10, 11); // RX, TX

void setup() {
  SoftSerial.begin(9600);
  Serial.begin(9600);
  pixels.begin();
  pinMode(SLEEP_PIN, OUTPUT);
}

void loop() {
  digitalWrite(SLEEP_PIN, HIGH);
  delay(1000L * 10L); // Wait for 1 minute
  int pm25_level = 0;
  int air_quality[3];

  CheckAirQuality(air_quality);
  pm25_level = air_quality[1]; // Only use PM2.5 reading

  // Determine the color based on PM2.5 levels
  if (pm25_level <= AQI_GOOD_MAX) {
    // Green
    SetColorIfChanged(0, 255, 0, 0);
  } else if (pm25_level <= AQI_MODERATE_MAX) {
    // Yellow
    SetColorIfChanged(255, 255, 0, 1);
  } else if (pm25_level <= AQI_UNHEALTHY_SG_MAX) {
    // Orange
    SetColorIfChanged(255, 165, 0, 2);
  } else if (pm25_level <= AQI_UNHEALTHY_MAX) {
    // Red
    SetColorIfChanged(255, 0, 0, 3);
  } else if (pm25_level <= AQI_VERY_UNHEALTHY_MAX) {
    // Purple
    SetColorIfChanged(128, 0, 128, 4);
  } else {
    // Maroon
    SetColorIfChanged(128, 0, 0, 5);
  }

  // Print PM2.5 level to Serial
  Serial.print("PM2.5: ");
  Serial.println(pm25_level);

  digitalWrite(SLEEP_PIN, LOW);
  delay(1000L * 60L * 3L); // Sleep for 3 minutes
}

void SetColorIfChanged(int r, int g, int b, int colorIndex) {
  if (last_color != colorIndex) {
    SetColor(r, g, b);
    last_color = colorIndex;
  }
}

void SetColor(int r, int g, int b) {
  pixels.clear();
  for (int i = 0; i < NUMPIXELS; i++) {
    pixels.setPixelColor(i, pixels.Color(r, g, b));
  }
  pixels.show();
}

// Existing functions CheckAirQuality, checkValue, transmitPM01, transmitPM2_5, transmitPM10 remain unchanged.
void CheckAirQuality(int* air_quality){
  if(SoftSerial.find(0x42)){    //start to read when detect 0x42
    SoftSerial.readBytes(buf,LENG);

    if(buf[0] == 0x4d){
      if(checkValue(buf,LENG)){
        air_quality[0] = transmitPM01(buf); //count PM1.0 value of the air detector module
        air_quality[1] = transmitPM2_5(buf);//count PM2.5 value of the air detector module
        air_quality[2] = transmitPM10(buf); //count PM10 value of the air detector module
      }
    }
  }
}

char checkValue(unsigned char *thebuf, char leng)
{
  char receiveflag=0;
  int receiveSum=0;

  for(int i=0; i<(leng-2); i++){
  receiveSum=receiveSum+thebuf[i];
  }
  receiveSum=receiveSum + 0x42;

  if(receiveSum == ((thebuf[leng-2]<<8)+thebuf[leng-1]))  //check the serial data
  {
    receiveSum = 0;
    receiveflag = 1;
  }
  return receiveflag;
}

int transmitPM01(unsigned char *thebuf)
{
  int PM01Val;
  PM01Val=((thebuf[3]<<8) + thebuf[4]); //count PM1.0 value of the air detector module
  return PM01Val;
}

//transmit PM Value to PC
int transmitPM2_5(unsigned char *thebuf)
{
  int PM2_5Val;
  PM2_5Val=((thebuf[5]<<8) + thebuf[6]);//count PM2.5 value of the air detector module
  return PM2_5Val;
  }

//transmit PM Value to PC
int transmitPM10(unsigned char *thebuf)
{
  int PM10Val;
  PM10Val=((thebuf[7]<<8) + thebuf[8]); //count PM10 value of the air detector module
  return PM10Val;
}
