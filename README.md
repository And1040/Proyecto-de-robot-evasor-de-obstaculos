# Proyecto-de-robot-evasor-de-obstaculos
Realizacion de un robot que evade obstaculos
#include <Servo.h>
// -------- Puente H --------
const int IN1 = 4, IN2 = 2, ENA = 3;
const int IN3 = 7, IN4 = 6, ENB = 5;
// -------- Ultrasonico --------
const int trigPin = 13;
const int echoPin = 12;
// -------- Servo --------
const int servoPin = 9;
Servo servo;
// -------- Estados --------
enum Estado {
AVANZAR,
PARAR,
RETROCEDER,
MIRAR_IZQ,
MIRAR_DER,
DECIDIR,
GIRAR
};
Estado estado = AVANZAR;
unsigned long tEstado = 0;
// -------- Variables --------
long dFrente = 0, dIzq = 0, dDer = 0;
char giro = 'D';
// -------- Motores --------
void mover(int a1, int a2, int b1, int b2, int v) {
digitalWrite(IN1, a1); digitalWrite(IN2, a2);
digitalWrite(IN3, b1); digitalWrite(IN4, b2);
analogWrite(ENA, v); analogWrite(ENB, v);
}
void adelante(int v){ mover(LOW,HIGH,LOW,HIGH,v); }
void atras(int v){ mover(HIGH,LOW,HIGH,LOW,v); }
void izquierda(int v){ mover(HIGH,LOW,LOW,HIGH,v); }
void derecha(int v){ mover(LOW,HIGH,HIGH,LOW,v); }
void parar() {
analogWrite(ENA,0); analogWrite(ENB,0);
digitalWrite(IN1,LOW); digitalWrite(IN2,LOW);
digitalWrite(IN3,LOW); digitalWrite(IN4,LOW);
}
// -------- Distancia --------
long medirDistancia() {
digitalWrite(trigPin, LOW);
delayMicroseconds(2);
digitalWrite(trigPin, HIGH);
delayMicroseconds(10);
digitalWrite(trigPin, LOW);
long d = pulseIn(echoPin, HIGH, 20000);
long cm = d * 0.034 / 2;
if (cm == 0) cm = 100;
return cm;
}
// -------- Setup --------
void setup() {
pinMode(IN1,OUTPUT); pinMode(IN2,OUTPUT); pinMode(ENA,OUTPUT);
pinMode(IN3,OUTPUT); pinMode(IN4,OUTPUT); pinMode(ENB,OUTPUT);
pinMode(trigPin,OUTPUT); pinMode(echoPin,INPUT);
servo.attach(servoPin);
servo.write(90);
}
// -------- Loop (FSM real) --------
void loop() {
switch (estado) {
case AVANZAR:
dFrente = medirDistancia();
adelante(130);
if (dFrente < 25) {
estado = PARAR;
tEstado = millis();
}
break;
case PARAR:
parar();
if (millis() - tEstado > 200) {
estado = RETROCEDER;
tEstado = millis();
}
break;
case RETROCEDER:
atras(120);
if (millis() - tEstado > 300) {
parar();
servo.write(150);
estado = MIRAR_IZQ;
tEstado = millis();
}
break;
case MIRAR_IZQ:
if (millis() - tEstado > 300) {
dIzq = medirDistancia();
servo.write(30);
estado = MIRAR_DER;
tEstado = millis();
}
break;
case MIRAR_DER:
if (millis() - tEstado > 300) {
dDer = medirDistancia();
servo.write(90);
estado = DECIDIR;
}
break;
case DECIDIR:
giro = (dIzq > dDer) ? 'I' : 'D';
estado = GIRAR;
tEstado = millis();
break;
case GIRAR:
if (giro == 'I') izquierda(130);
else derecha(130);
if (millis() - tEstado > 450) {
parar();
estado = AVANZAR;
}
break;
}
}
