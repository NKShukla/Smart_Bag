# Smart_Bag
#include <TinyGPS.h>
#include <SoftwareSerial.h>
#include <Servo.h>   //servo library
  
SoftwareSerial gpsSerial(10,11);//rx,tx
TinyGPS gps; // create gps object
Servo servo1,servo2,servo3;

float lat = 28.54708,lon = 77.27344; // create variable for latitude and longitude object
int trigPin = 5;
int echoPin = 6;
int servoPin1 = 7;
int servoPin2 = 8;
int servoPin3 = 9;
long duration, dist, average;   
long aver[3];   //array for average
const int led = 12;
char data1='b';
int switch1=0;
int switch2=0;


void setup() {
  Serial.begin(9600);
  gpsSerial.begin(9600); // connect gps sensor
  pinMode(trigPin, OUTPUT);  
  pinMode(echoPin, INPUT);
}

void measure()
{  
  digitalWrite(10,HIGH);
  digitalWrite(trigPin, LOW);
  delayMicroseconds(5);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(15);
  digitalWrite(trigPin, LOW);
  pinMode(echoPin, INPUT);
  duration = pulseIn(echoPin, HIGH);
  dist = (duration/2) / 29.1;    //obtain distance
}  

void loop()
{
  char data=data1;
  int switch2=0;
  int switch3=0;
  for (int i=0;i<=2;i++){   //average distance
    measure();               
    aver[i]=dist;            
    delay(10);              //delay between measurements
  }
 dist=(aver[0]+aver[1]+aver[2])/3;    
  int light = analogRead(A0);
  if (Serial.available()>0)
    data=Serial.read();
  while(gpsSerial.available())
  { // check for gps data
    if(gps.encode(gpsSerial.read()))// encode gps data
      gps.f_get_position(&lat,&lon); // get latitude and longitude
  }
  String latitude = String(lat,6);
  String longitude = String(lon,6);
  Serial.println(latitude+','+longitude);
  delay(1000);
  if(data=='a')
  {
    switch1=1;
    data1='a';
  }
  else if(data=='b')
  {
    switch1=0;
    data1='b';
  }
  else if(data=='c')
    switch2=1;
  else if(data=='d')
    switch2=2;
  else if(data=='e')
    switch3=1;
  else if(data=='f')
    switch3=2;
  if (light < 50 or switch1)
      digitalWrite(led,HIGH);
  else
     digitalWrite(led,LOW);
  if (switch2==1)
  {
    servo1.attach(servoPin1);
    servo1.write(0);  
    delay(800);
    servo1.detach();
  }
  else if (switch2==2)
  {
    servo1.attach(servoPin1);
    servo1.write(180);
    delay(800);
    servo1.detach();
  }
  else if ( dist<10 ) {
  //Change distance as per your need
    servo1.attach(servoPin1);
    servo1.write(0);
    delay(800);
    servo1.detach();
    delay(5000);   
    servo1.attach(servoPin1);
    servo1.write(180);  
    delay(800);
    servo1.detach();
  }
  if (switch3==1)
  {
    servo2.attach(servoPin2);
    servo3.attach(servoPin3);
    servo2.write(0);
    servo3.write(0);
    delay(10000);
    servo2.detach();
    servo3.detach();
  }
  else if (switch3==2)
  {
    servo2.attach(servoPin2);
    servo3.attach(servoPin3);
    servo2.write(180);
    servo3.write(180);
    delay(10000);
    servo2.detach();
    servo3.detach();
  }
}
