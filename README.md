# Estacion-de-monitoreo-
Repositorio del proyecto de industria 4.0, estacion de monitoreo de contaminacion atmosferica
#include <MQUnifiedsensor.h>

//Definitions
#define placa "ESP-32"
#define Voltage_Resolution 3.3
#define pin 13 // Entrada analoga para el sensor
#define type "MQ-135" //MQ135
#define ADC_Bit_Resolution 12 // Resolución ADC
#define RatioMQ135CleanAir 3.6//RS / R0 


// Declaración del sensor
MQUnifiedsensor MQ135(placa, Voltage_Resolution, ADC_Bit_Resolution, pin, type);


String medidas = "";


void setup() {

  Serial.begin(115200);

  //------------------------------------------------------------
  //------------------ Inicio Bluetooth ------------------------




  //Modelo matematico para calcular la concentración de PPM  
  MQ135.setRegressionMethod(1); //_PPM =  a*ratio^b
  //Activación del sensor
  MQ135.init(); 

  //------------ Proceso de calibración ----------------------------
  //Serial.print("Calibrating please wait.");
  float calcR0 = 0;
  for(int i = 1; i<=10; i ++)
  {
    MQ135.update(); // Update data, the arduino will read the voltage from the analog pin
    calcR0 += MQ135.calibrate(RatioMQ135CleanAir);
    Serial.print(".");
  }
  MQ135.setR0(calcR0/10);
  
  
  /**********  Calibración ***************/ 
  //Serial.println("* Values from MQ-135 ***");
  //Serial.println("|    CO   |  Alcohol |   CO2  |  Toluen  |  NH4  |  Aceton  |");  
}

void loop() {
  MQ135.update(); 
  MQ135.setA(605.18); MQ135.setB(-3.937); // Concentración de CO
  float CO = MQ135.readSensor(); 
  Serial.print("@");
  medidas += String(CO, 2)+"#"; 
  MQ135.setA(77.255); MQ135.setB(-3.18); // Concentración de Alcohol
  float Alcohol = MQ135.readSensor(); 
  medidas += String(Alcohol, 2)+"#"; 
  MQ135.setA(110.47); MQ135.setB(-2.862); // Concentración de CO2
  float CO2 = MQ135.readSensor(); 
  medidas += String(CO2+400, 2)+"#"; 
  MQ135.setA(44.947); MQ135.setB(-3.445); // Concentración de Toluen
  float Toluen = MQ135.readSensor(); 
  medidas += String(Toluen, 2)+"#"; 
  MQ135.setA(102.2 ); MQ135.setB(-2.473); // Concentración de NH4
  float NH4 = MQ135.readSensor(); 
  medidas += String(NH4, 2)+"#";
  MQ135.setA(34.668); MQ135.setB(-3.369); // Concentración de Aceton
  float Aceton = MQ135.readSensor(); 
  medidas += String(Aceton, 2)+"#";
  Serial.println(medidas);
  

  //Serial.print("|   "); Serial.print(CO); 
  //Serial.print("   |   "); Serial.print(Alcohol);
  // Note: 400 Offset for CO2 source: https://github.com/miguel5612/MQSensorsLib/issues/29
  /*
  Motivation:
  We have added 400 PPM because when the library is calibrated it assumes the current state of the
  air as 0 PPM, and it is considered today that the CO2 present in the atmosphere is around 400 PPM.
  https://www.lavanguardia.com/natural/20190514/462242832581/concentracion-dioxido-cabono-co2-atmosfera-bate-record-historia-humanidad.html
  */
  /*
  Serial.print("   |   "); Serial.print(CO2 + 400); 
  Serial.print("   |   "); Serial.print(Toluen); 
  Serial.print("   |   "); Serial.print(NH4); 
  Serial.print("   |   "); Serial.print(Aceton);
  Serial.println("   |"); 
  
  /*
    Exponential regression:
  GAS      | a      | b
  CO       | 605.18 | -3.937  
  Alcohol  | 77.255 | -3.18 
  CO2      | 110.47 | -2.862
  Toluen  | 44.947 | -3.445
  NH4      | 102.2  | -2.473
  Aceton  | 34.668 | -3.369
  */
  delay(1000); //Sampling frequency
  medidas = "";
}
