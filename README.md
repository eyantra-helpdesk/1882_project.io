# 1882_project.io

#include<readiness_io.h>
#include<Ticker.h>
#include"config.h"

constint LED_PIN = 5; // The pin connecting the LED (D3)
constint INTERRUPT1_PIN = 14; // The pin connects the test button (D5)
constint INTERRUPT2_PIN = 12; // The pin connects the 2nd test button (D6)
constint SOLENOID_PIN = 13; // The pin connects to the relay (D7)
volatile byte interrupt = 0;
readiness_io client(CHANNEL_ID, TOPIC, SENSOR_ID, VERSION, FORMAT);
Ticker timer;

voidbutton1Interrupt() {
digitalWrite(LED_PIN, HIGH);
digitalWrite(SOLENOID_PIN, HIGH);
}

voidbutton2Interrupt() {
digitalWrite(LED_PIN, LOW);
digitalWrite(SOLENOID_PIN, LOW);
}
/* Interrupt timer for collecting data to the Readiness.io server */
voidreadFromServer(){
interrupt++;
}
voidsetup() {
pinMode(LED_PIN, OUTPUT);
pinMode(BUILTIN_LED, OUTPUT);
pinMode(SOLENOID_PIN, OUTPUT);
digitalWrite(SOLENOID_PIN, HIGH);
digitalWrite(BUILTIN_LED, HIGH); // internal LED is switched on when low - so we have to switch it off/
Serial.begin(115200);
Serial.setTimeout(2000);
while(!Serial) { } // Wait for serial to initialize.
Serial.println("Device Started");
Serial.print("Connecting to ");
Serial.println(WIFI_SSID);
client.wifiConnection(WIFI_SSID, WIFI_PASS);
pinMode(INTERRUPT1_PIN, INPUT);
pinMode(INTERRUPT2_PIN, INPUT);
attachInterrupt(digitalPinToInterrupt(INTERRUPT2_PIN), button2Interrupt, RISING); // Attach the interrupt.
attachInterrupt(digitalPinToInterrupt(INTERRUPT1_PIN), button1Interrupt, RISING); // Attach the interrupt.
timer.attach(UPDATE_RATE, readFromServer);
client.testConnection();
}
voidloop() {
if(interrupt>0){
String result = client.subscribe(LISTEN_TOPIC);
float reading = client.getReading(result);
if (reading == 0) {
digitalWrite(SOLENOID_PIN, HIGH);
digitalWrite(LED_PIN,HIGH);
} else {
digitalWrite(SOLENOID_PIN, LOW);
digitalWrite(LED_PIN,LOW);
}
interrupt=0;
}
}
