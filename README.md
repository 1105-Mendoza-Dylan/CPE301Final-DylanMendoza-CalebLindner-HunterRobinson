# CPE301Final, Dylan Mendoza, Caleb Lindner, Hunter Robinson

//CPE301 Final Project
//Dylan Mendoza
//Caleb Lindner
//Hunter Robinson

#include "DHT.h"
#include <LiquidCrystal.h>

#define DHTTYPE DHT11
#define DHTPIN 12

DHT dht(DHTPIN, DHTTYPE);

const int rs = 11;
const int en = 10;
const int d4 = 22;
const int d5 = 23;
const int d6 = 24;
const int d7 = 25;

const int fan =13;
const int power = 2;
const int error = 5;
const int on = 6;
const int off = 4;
const int level = A0;

float temp;
float hum;
int powerStateCurrent = 0;
int powerStateLast;
int systemState = 1;
float water;

LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

void setup() {
  // put your setup code here, to run once:
pinMode(fan, OUTPUT); //register
pinMode(power, INPUT); // register

Serial.begin(9600); //register
lcd.begin(16, 2);
dht.begin();
}

void loop() {

  water = analogRead(level); //collect water level in cooler, register

  //LCD Display Portion, takes readings and then writes temp and hummidity to lcd
  temp = dht.readTemperature();
  hum = dht.readHumidity();
  lcd.setCursor(0,0);
  lcd.print("Temp C: ");
  lcd.print(temp,2);
  lcd.setCursor(0,1);
  lcd.print("Humidity: ");
  lcd.print(hum, 2);  

  //turns fan on or off depending on read temperature from the above portion
  if(temp>24){
    digitalWrite(fan,HIGH); //register
  }else if(temp<20){
    digitalWrite(fan,LOW); //register
  }
  

  //error portion, if there is an error happening, it is detected here
  if(isnan(temp) || isnan(hum)|| water<180){
   Serial.println("Error"); //register
   digitalWrite(error, HIGH); //register
   digitalWrite(on, LOW); //regsiter
   digitalWrite(off, LOW); //register
   digitalWrite(fan,LOW); //register
   return;
  }

  //System switch on and off
  powerStateLast = powerStateCurrent;
  powerStateCurrent = digitalRead(power);
if(powerStateCurrent == HIGH && powerStateLast == LOW){ //using a pull down resistor button to toggle the system state
  systemState = !systemState;  
}

if(systemState == 0){
  digitalWrite(off,HIGH); //register
  digitalWrite(on,LOW); //register
  digitalWrite(fan,LOW); //register
  lcd.noDisplay();
}
else{
  digitalWrite(on,HIGH); //register
  digitalWrite(off,LOW); //register
  lcd.display();
  return;
}

}