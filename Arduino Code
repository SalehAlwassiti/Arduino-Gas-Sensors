#include <MQ2.h>
#include "MQ131.h"
#include <DFRobot_ENS160.h>

#include <SPI.h>
#include <SD.h>


#define I2C_COMMUNICATION

#ifdef  I2C_COMMUNICATION
  DFRobot_ENS160_I2C ENS160(&Wire, /*I2CAddr*/ 0x53);
#else
  uint8_t csPin = D3;
  DFRobot_ENS160_SPI ENS160(&SPI, csPin);
#endif

int pin = A0;
float CO_Gas,O3_Gas;
int Hours;

File dataFile;

MQ2 mq2(pin);

void setup(void){
  
  Serial.begin(115200);

  //initial setup for SD Card
  if (!SD.begin(10)) {
    // Serial.println("Card failed, or not present");
    while (1);
  }

  delay(3000);

  dataFile = SD.open("datalog.txt", FILE_WRITE);

  Serial.println(dataFile);

  if (dataFile) {

    dataFile.println("O3_Gas    CO_Gas    CO2_Gas    Time    ");
  
    dataFile.close();
 
  }


  //ENS160
  while( NO_ERR != ENS160.begin() ){
    Serial.println("Communication with device failed, please check connection");
    delay(3000);
  }
  Serial.println("ENS 160 Device is OK!");

  ENS160.setPWRMode(ENS160_STANDARD_MODE);
  ENS160.setTempAndHum(/*temperature=*/25.0, /*humidity=*/50.0);


  //MQ131
  MQ131.begin(2,A1, LOW_CONCENTRATION, 1);
  Serial.println("MQ131 Calibration in progress...");
  MQ131.calibrate();

  Serial.println("Calibration done!");
  Serial.print("R0 = ");
  Serial.print(MQ131.getR0());
  Serial.println(" Ohms");
  Serial.print("Time to heat = ");
  Serial.print(MQ131.getTimeToRead());
  Serial.println(" s");

  // MQ2 Setup
  mq2.begin();
  Serial.println("MQ2 Warming up");
  delay(20000);
  Serial.println("<<<<<<Ready>>>>>>");
}

void loop(){
  
  //O3 Mq131
  Serial.println("Sampling...");
  MQ131.sample();
  O3_Gas = MQ131.getO3(PPB);
  Serial.print("Concentration O3 : ");
  Serial.print(O3_Gas);
  Serial.println(" ppb");

  //Carbon monoxide Code
  CO_Gas = mq2.readCO();
  Serial.print("CO Gas Value: ");
  Serial.print(CO_Gas);
  Serial.println(" ppm");

  //Carbon Dioxide Code
  uint8_t Status = ENS160.getENS160Status();
  Serial.print("Sensor operating status : ");
  Serial.println(Status);

  uint16_t ECO2 = ENS160.getECO2();
  Serial.print("Carbon dioxide equivalent concentration : ");
  Serial.print(ECO2);
  Serial.println(" ppm");

  //SD Card Code
  dataFile = SD.open("datalog.txt", FILE_WRITE);

  if (dataFile) {

    dataFile.print(O3_Gas);
    dataFile.print("PPB    ");

    dataFile.print(CO_Gas);
    dataFile.print("PPM    ");

    dataFile.print(ECO2);
    dataFile.print("PPM    ");

    dataFile.print("Hour_");
    dataFile.println(Hours);    

    dataFile.close();

    Serial.println("Printed");

    Hours +=1;   
  }

  // delay(1);
  delay(3600000);
}
