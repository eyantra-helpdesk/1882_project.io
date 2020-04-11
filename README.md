# 1882_project.io

#include<readiness_io.h><br/><br/>
#include<Ticker.h><br/><br/>
#include"config.h"<br/><br/><br/>
<br/><br/>
constint LED_PIN = 5; // The pin connecting the LED (D3)<br/><br/>
constint INTERRUPT1_PIN = 14; // The pin connects the test button (D5)<br/><br/>
constint INTERRUPT2_PIN = 12; // The pin connects the 2nd test button (D6)<br/><br/>
constint SOLENOID_PIN = 13; // The pin connects to the relay (D7)<br/><br/>
volatile byte interrupt = 0;<br/><br/>
readiness_io client(CHANNEL_ID, TOPIC, SENSOR_ID, VERSION, FORMAT);<br/><br/>
Ticker timer;<br/><br/>
<br/><br/>
voidbutton1Interrupt() {<br/><br/>
digitalWrite(LED_PIN, HIGH);<br/><br/>
digitalWrite(SOLENOID_PIN, HIGH);<br/><br/>
}<br/><br/>
<br/><br/>
voidbutton2Interrupt() {<br/><br/>
digitalWrite(LED_PIN, LOW);<br/><br/>
digitalWrite(SOLENOID_PIN, LOW);<br/><br/>
}<br/><br/>
/* Interrupt timer for collecting data to the Readiness.io server */<br/><br/>
voidreadFromServer(){<br/><br/>
interrupt++;<br/><br/>
}<br/><br/>
voidsetup() {<br/><br/>
pinMode(LED_PIN, OUTPUT);<br/><br/>
pinMode(BUILTIN_LED, OUTPUT);<br/><br/>
pinMode(SOLENOID_PIN, OUTPUT);<br/><br/>
digitalWrite(SOLENOID_PIN, HIGH);<br/><br/>
digitalWrite(BUILTIN_LED, HIGH); // internal LED is switched on when low - so we have to switch it off/<br/><br/>
Serial.begin(115200);<br/><br/>
Serial.setTimeout(2000);<br/><br/>
while(!Serial) { } // Wait for serial to initialize.<br/><br/>
Serial.println("Device Started");<br/><br/>
Serial.print("Connecting to ");<br/><br/>
Serial.println(WIFI_SSID);<br/><br/>
client.wifiConnection(WIFI_SSID, WIFI_PASS);<br/><br/>
pinMode(INTERRUPT1_PIN, INPUT);<br/><br/>
pinMode(INTERRUPT2_PIN, INPUT);<br/><br/>
attachInterrupt(digitalPinToInterrupt(INTERRUPT2_PIN), button2Interrupt, RISING); // Attach the interrupt.<br/><br/>
attachInterrupt(digitalPinToInterrupt(INTERRUPT1_PIN), button1Interrupt, RISING); // Attach the interrupt.<br/><br/>
timer.attach(UPDATE_RATE, readFromServer);<br/><br/>
client.testConnection();<br/><br/>
}<br/><br/>
voidloop() {<br/><br/>
if(interrupt>0){<br/><br/>
String result = client.subscribe(LISTEN_TOPIC);<br/><br/>
float reading = client.getReading(result);<br/><br/>
if (reading == 0) {<br/><br/>
digitalWrite(SOLENOID_PIN, HIGH);<br/><br/>
digitalWrite(LED_PIN,HIGH);<br/><br/>
} else {<br/><br/>
digitalWrite(SOLENOID_PIN, LOW);<br/><br/>
digitalWrite(LED_PIN,LOW);<br/><br/>
}<br/><br/>
interrupt=0;<br/><br/>
}<br/><br/>
}<br/><br/>
