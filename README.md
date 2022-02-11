# Magda-projekt-UMG# Magda-projekt-UMG
* ## CZUJNIK PARKOWANIA
* #### I) UŻYTE ELEMENTY: 
* ESP32/Arduino Nano/Uno 
 * Ultrdźw. czujnik odległości 
* Buzzer głośniejszy 
* Taśma led wyświetlająca odległość
* #### II) KOD:
``` c 
// ESP8266 D1 Mini Wemos
//  FizPIN  ARDUINO     FUNCTION
//  D1      5           echo
//  D2      4           trig
//  D5      14          buzzer
//  D4      2           pasek led
#include <Adafruit_NeoPixel.h>
 
#define MAX_BRIGHTNESS 160 //maksymalna jasnosc pojedynczej diody led 0 - 255 
 
//1s - 1000ms - 1 000 000us\
// 1m - 100cm
// cm/us (343.0 * 1) / 10000 343.0 /10000 = 0.0343
#define SOUND_SPEED 0.0343
#define BUZZER_PIN 14
#define LED_PIN 2
#define ECHO_PIN 5
#define TRIGGER_PIN 4
 
// inicjalizacja paska led (ilosc diod, PIN, (USTAWIENIA, czestotliwosc itp))
Adafruit_NeoPixel pasek_led(8, LED_PIN, NEO_GRB + NEO_KHZ800);
 
// Odpowiednik odleglosci do czestotliwosci pipczenia buzzera oraz ilosci zapalonych diod
int ranges[8] = { 220,185,150,120,95,65,45,30 };
unsigned long poprzedniCzasBuzzera = 0;
unsigned long poprzedniCzasEcho = 0;
// Jeżeli jest >0 oznacza, że buzzer będzie co taki czas wydawał dźwięk
unsigned long okresBuzzera = 0;
float odleglosc = -1;
bool noweDane=false;
 
void updateByDistance(float distance) {
    int index = -1;
    for (int i = 0; i < 8; i++) {
        if (distance < ranges[i]) {
            pasek_led.setPixelColor(i, pasek_led.Color(MAX_BRIGHTNESS, 0, 0));
            index = i;
        }
        else {
            pasek_led.setPixelColor(i, pasek_led.Color(0, MAX_BRIGHTNESS, 0));
        }
    }
    pasek_led.show();
    if (index >= 7) {
        digitalWrite(BUZZER_PIN, LOW);
        okresBuzzera = 0;
    }
    else if (index <= -1) {
        digitalWrite(BUZZER_PIN, HIGH);
        okresBuzzera = 0;
    }
    else {
        if(index==0) okresBuzzera = 1200;
        else okresBuzzera = 1000 / index;
    }
}
 
void updateBuzzer() {
    if (okresBuzzera>0 && poprzedniCzasBuzzera + okresBuzzera < millis()) {
        if (digitalRead(BUZZER_PIN) == HIGH) {
            digitalWrite(BUZZER_PIN, LOW);
        }
        else {
            digitalWrite(BUZZER_PIN, HIGH);
        }
        poprzedniCzasBuzzera = millis();
    }
}
 
// należy zastosować atrybut icache_ram_attr
void ICACHE_RAM_ATTR interrupt(){
    if (digitalRead(ECHO_PIN) == HIGH) {
        // zapis aktualnego czasu (moment w ktorym zostala wygenerowana fala)
        poprzedniCzasEcho = micros();
    }
    else {
        // obliczenie roznicy w czasie pomiedzy wygenerowaniem fali, a informacji o powrocie
        // podzielone przez 2 ze wzgledu na to, ze fala przebyla droge do obiektu po czym wrocila
        // fala wykonala te sama trase dwukrotnie zmienil sie jedynie jej zwrot
        unsigned long deltaTime = (micros() - poprzedniCzasEcho) / 2;
 
        // v = s/t, s = v*t
        if (odleglosc < 0)
            odleglosc = SOUND_SPEED * deltaTime;
        else //filtr IIR dolnoprzepustowy 
            odleglosc = 0.9*odleglosc + (SOUND_SPEED * deltaTime)*0.1;
        // powiadomienie o gotowosci do dalszego pingowania
        noweDane = true;
    }
}
 
void trig() {
    // noweDane przechowuje czy aktualnie czekamy na powrót fali
    if (!noweDane) return;
    noweDane = false;
    digitalWrite(TRIGGER_PIN, LOW);
    delayMicroseconds(2);
    digitalWrite(TRIGGER_PIN, HIGH);
    delayMicroseconds(10);
    digitalWrite(TRIGGER_PIN, LOW);
}
 
void setup() {
    //INICJALIZACJA PORTU SZEREGOWEGO
    Serial.begin(115200);
    delay(10);
    Serial.print("\n\n\n\n\n\n\n\nStarting...");
 
    //INICJALIZACJA ULTRADZWIEKOWEGO CZUJNIKA ODLEGLOSCI
    pinMode(TRIGGER_PIN, OUTPUT);
    digitalWrite(TRIGGER_PIN, LOW);
    pinMode(ECHO_PIN, INPUT);
    //przerwanie na pin ECHO_PIN stan CHANGE
    attachInterrupt(ECHO_PIN, interrupt, CHANGE);
 
    //INICJALIZACJA PASKA LED
    pasek_led.begin();
    pasek_led.show();
    delay(10);
    for (int i = 0; i < 7; i++) {
        pasek_led.setPixelColor(i, pasek_led.Color(0, MAX_BRIGHTNESS, 0));
    }
    pasek_led.show();
 
    //INICJALIZACJA BUZZERA
    pinMode(BUZZER_PIN, OUTPUT);
 
    //sterowanie masą
    digitalWrite(BUZZER_PIN, HIGH);
}
 
void loop() {
    if(noweDane) {
        updateByDistance(odleglosc);
    }
    
    trig();
    updateBuzzer();
}
```
* #### ZDJĘCIA:
![This is an image]
