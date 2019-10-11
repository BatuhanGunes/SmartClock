# SmartClock
arduino project hosting modules:
- dht11 ( Humidity & Temperature Sensor)
- MQ-4 (methane & natural gas detector)
- Real Time Clock 
- Buzzer 
- 16x2 Lcd Display
  - 10k potans
- leds
- jumpers
- resistances

### Informations

Coming soon

### Required Libraries

  - DS3231.h
  - dht11.h
  - LiquidCrystal.h  (a library that exists by default)

### Pictures

Coming soon

### Code

```javascript

  #include <DS3231.h>           // Real Time Clock Library
  #include <LiquidCrystal.h>    // 16x2 LCD led screen library
  #include <dht11.h>            // temperature&humidity sensor library
  
  DS3231  rtc(SDA, SCL);                    //set Real Time Clock pins
  LiquidCrystal lcd(2, 3, 4, 5, 6, 7);      //set Lcd screen pins
  dht11 DHT11_sensor;                       // We have created a DHT11 object named DHT11_sensor.
  
  // We define our sensor pins:
  #define dht11_pin A0
  #define mq4_AnalogPin A1
  #define redLed_pin 11
  #define greenLed_pin 10
  #define yellowLed_pin 9
  #define buzzer_pin 8

  int sensorValue = 0;    // Mq4 methane gas output value
  
  // We set the preheating time required for sensor operation to 5sn
  #define preheat_time 5000
  // We set a threshold of the alarm to sound.
  #define threshold 400
  // We set a Display transition time 
  #define numberOfCycles 138
  // time taken for a cycle = 0.216 micro second
  // responseTimeOfSensors X numberOfCycles:138 = displayTransitionTime:29.9second
  

//-----------------------------------------------------------------------------------------------------------------------------
  
  void setup() { 
   //We define the buzzer module as input for the alarm
   pinMode(buzzer_pin, INPUT);
   pinMode(redLed_pin, OUTPUT);
   pinMode(greenLed_pin, OUTPUT);
   pinMode(yellowLed_pin, OUTPUT);
    
   // We initiate serial communication so that we can see the sensor value on the serial monitor:
   Serial.begin(9600);
    
   rtc.begin();       // Real Time Clock starting
   lcd.begin(16,2);   // Lcd screen starting

  // We expect the sensor to warm up for the first 5 seconds.
   methaneGasSensorPreparationProcess();
  }

//-----------------------------------------------------------------------------------------------------------------------------
  
  void loop() { 

    for(int i=0; i<numberOfCycles; i++){
      methaneGasDetector();
         // If the sensor value is higher than the threshold, we call the alarm () function:
        if (sensorValue >= threshold){
          Buzzer(100);
          digitalWrite(greenLed_pin, LOW);
          digitalWrite(yellowLed_pin, LOW);
        }
        else{
          pinMode(buzzer_pin, INPUT); // buzzer module as input by silencing
          Clock();
          digitalWrite(yellowLed_pin, LOW);
        }
        delay(200);
    }
    
    for(int i=0; i<numberOfCycles; i++){
      methaneGasDetector();
         // If the sensor value is higher than the threshold, we call the alarm () function:
        if (sensorValue >= threshold){
          Buzzer(100);
          digitalWrite(greenLed_pin, LOW);
          digitalWrite(yellowLed_pin, LOW);
        }
        else{
          pinMode(buzzer_pin, INPUT); // buzzer module as input by silencing
          temperatureAndHumidity();
          digitalWrite(greenLed_pin, LOW);
        }
        delay(1);
    }
  }

//-----------------------------------------------------------------------------------------------------------------------------
  
  void Clock(){
   lcd.setCursor(0,0);  //Lcd line = 0
   lcd.print("Time:   ");
   lcd.print(rtc.getTimeStr());
   lcd.setCursor(0,1);  //Lcd line = 0
   lcd.print("Date: ");
   lcd.print(rtc.getDateStr());

   // Serial port codes
   Serial.print("Time:");
   Serial.print(rtc.getTimeStr());
   Serial.print("\t");
   Serial.print("Date: ");
   Serial.println(rtc.getDateStr());

   digitalWrite(greenLed_pin, HIGH);
  }

//-----------------------------------------------------------------------------------------------------------------------------

    void temperatureAndHumidity(){
    // Checking if the sensor is read.
    int chk = DHT11_sensor.read(dht11_pin);
    digitalWrite(yellowLed_pin, HIGH);
    
    lcd.setCursor(0,0);
    lcd.print("  Temp : ");
    lcd.print((float)DHT11_sensor.temperature, 2);
    lcd.print("   ");
    lcd.setCursor(0,1);
    lcd.print("  Humi : ");
    lcd.print((float)DHT11_sensor.humidity, 2);
    lcd.print("   ");
        
    // We print the data from the sensor on the serial monitor.
    Serial.print("Humidity (%): ");
    Serial.print((float)DHT11_sensor.humidity, 2);
    Serial.print("\t");
   
    Serial.print("Temperature (Celcius): ");
    Serial.print((float)DHT11_sensor.temperature, 2);
    Serial.print(" °C \t");
   
    Serial.print("Temperature (Kelvin): ");
    Serial.print(DHT11_sensor.kelvin(), 2);
    Serial.print(" K \t\t");
    
    Serial.print("Temperature (Fahrenheit): ");
    Serial.print(DHT11_sensor.fahrenheit(), 2);
    Serial.println(" °F \t"); 
  }

//-----------------------------------------------------------------------------------------------------------------------------
  
  void methaneGasSensorPreparationProcess(){
    Serial.println("preparing methane gas sensor..."); 
    for(int i=0; i <= preheat_time; i = i+1000){
      lcd.setCursor(0,0);
      lcd.print("Preparing Methan");
      lcd.setCursor(0,1);
      lcd.print("  Gas Sensor..  ");
      
      digitalWrite(redLed_pin, HIGH);
      delay(500);
      digitalWrite(redLed_pin, LOW);
      delay(500);
    }
    delay(500);
  }

//-----------------------------------------------------------------------------------------------------------------------------
  
   void methaneGasDetector(){
    // With the analogRead() function we measure the sensor value and keep it in a variable:
    sensorValue = analogRead(mq4_AnalogPin);
    digitalWrite(redLed_pin, HIGH);
    
    // To see the sensor value from your computer, we write to the serial monitor:
    Serial.print("Methane Gas : ");
    Serial.print(sensorValue);
    Serial.print("\t"); 
    delay(5);
  }

//-----------------------------------------------------------------------------------------------------------------------------
  
  // We define our alarm function. This function takes the time that determines how often the buzzer will sound as a parameter.
  void Buzzer(unsigned int duration){
      
      lcd.setCursor(0,0);
      lcd.print("     ALERT...   ");
      lcd.setCursor(0,1);
      lcd.print("   Value : ");
      lcd.print(sensorValue);
      lcd.print("  ");
      
    // We want the buzzer to sound at 440Hz (La note):
    tone(buzzer_pin, 440);
    digitalWrite(redLed_pin, HIGH);
    digitalWrite(yellowLed_pin, HIGH);
    digitalWrite(greenLed_pin, HIGH);
    delay(duration);
    
    noTone(buzzer_pin);
    digitalWrite(redLed_pin, LOW);
    digitalWrite(yellowLed_pin, LOW);
    digitalWrite(greenLed_pin, LOW);
    delay(duration);
  }

```
