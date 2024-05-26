# Simulation Intelligente d'un Système Feu de Passage Piéton

| | |
|-|-|
|`Author` | Herve PHANZU

## Description
Ensemble de feu rouge pour passage piéton et route.
## Motivation
Ce projet vise à promouvoir la sécurité des piétons, à éduquer sur les règles de circulation et à développer des compétences en ingénierie logicielle et matérielle afin d'eduquer les jeunes generations.
## Architecture
Le projet utilise un Arduino Nano comme cerveau central pour contrôler le système de feu de passage piéton. Il intègre un bouton, un capteur de proximité pour les voitures, des LEDs et un buzzer piezo pour fournir une expérience réaliste et sécurisée.

Lorsqu'un piéton souhaite traverser, il appuie sur le bouton. Ce signal est capté par l'Arduino Nano, qui déclenche alors le changement du feu de circulation. Les LEDs simulant les feux de circulation passent alors au vert pour les piétons, tandis que les feux pour les voitures passent au rouge.

Pendant la phase de passage piéton, l'Arduino Nano surveille également en continu le capteur de proximité pour les voitures. Si des voitures sont détectées à proximité du passage piéton, le système raccourci la phase de feu rouge pour les voitures afin de fluidifier le trafic.

Pour renforcer la perception visuelle et auditive des changements de feux, le buzzer piezo émet des signaux sonores synchronisés avec les changements de feux. Ces signaux aident à alerter les piétons du début et de la fin de leur phase de passage, ainsi que les conducteurs du changement d'état du feu de circulation.

Après la fin de la séquence de passage piéton, l'Arduino Nano retourne à son état initial pour attendre une nouvelle demande de passage ou détecter la présence de voitures, assurant ainsi un fonctionnement continu et adaptatif du système feu de passage piéton.

### Block diagram

<!-- Make sure the path to the picture is correct -->
![Block Diagram](Block-diagramme.png)

### Schematic

![Schematic](Schema.png)

### Components


<!-- This is just an example, fill in with your actual components -->

| Device | Usage | Price |
|--------|--------|-------|
| Arduino | Microcontrolleur |[25 RON](https://www.optimusdigital.ro/ro/compatibile-cu-arduino-nano/1686-placa-de-dezvoltare-compatibila-cu-arduino-nano-atmega328p-i-ch340.html?search_query=Arduino+Nano&results=22) |
| Bouton Push | Button | [1 RON](https://www.optimusdigital.ro/ro/butoane-i-comutatoare/1119-buton-6x6x6.html?search_query=buton&results=222) |
| Jumper Wires | Connecting components | [7 RON](https://www.optimusdigital.ro/ro/fire-fire-mufate/884-set-fire-tata-tata-40p-10-cm.html?search_query=set+fire&results=110) |
| Breadboard | Project board | [5 RON](https://www.optimusdigital.ro/ro/prototipare-breadboard-uri/44-breadboard-400-points.html) |
|Buzzer Piezo| bruit | [2 Ron](https://www.optimusdigital.ro/ro/audio-buzzere/634-buzzer-pasiv-de-5-v.html?search_query=buzzer&results=62) |
| LED | lumiere | 1 RON [vert](https://www.optimusdigital.ro/ro/optoelectronice-led-uri/697-led-verde-de-3-mm-cu-lentile-difuze.html?search_query=led&results=818) [rouge](https://www.optimusdigital.ro/ro/optoelectronice-led-uri/697-led-verde-de-3-mm-cu-lentile-difuze.html?search_query=led&results=818) |
|HC SR04| Capteur proximite| [7 RON](https://www.optimusdigital.ro/ro/senzori-senzori-ultrasonici/9-senzor-ultrasonic-hc-sr04-.html?gad_source=1&gclid=CjwKCAjw0YGyBhByEiwAQmBEWuIeN6JU2Yfu_L0-K-zlkALkLRz3k7O205eIvhzLc3jk6mt59dOX4xoCFT8QAvD_BwE) |

### Libraries
<!-- This is just an example, fill in the table with your actual components -->

| Library | Description | Usage |
|---------|-------------|-------|
| [capteur proximite](https://github.com/gamegine/HCSR04-ultrasonic-sensor-lib) | HCSR04 is an Arduino library HCSR04 Sensors | Librairie pour le capteur de proximite  |

Code 
```C

#define RED_PED 12     
#define GREEN_PED 11

#define RED_CAR 2
#define YELLOW_CAR 3
#define GREEN_CAR 4

#define BUZZER A0
#define BUTTON A5


void setup() {

  pinMode(BUZZER, OUTPUT);  
  digitalWrite(BUZZER, LOW);
  
  pinMode(BUTTON, INPUT_PULLUP);

  pinMode(RED_CAR, OUTPUT);
  pinMode(YELLOW_CAR, OUTPUT);
  pinMode(GREEN_CAR, OUTPUT);
  digitalWrite(YELLOW_CAR, LOW);
  digitalWrite(RED_CAR, LOW);
  digitalWrite(GREEN_CAR, HIGH);

  pinMode(GREEN_PED, OUTPUT);
  digitalWrite(GREEN_PED, LOW);
  pinMode(RED_PED, OUTPUT);
  digitalWrite(RED_PED, HIGH);
  

}

long readUltrasonicDistance(int triggerPin, int echoPin)
{
  pinMode(triggerPin, OUTPUT);  
  digitalWrite(triggerPin, LOW);
  delayMicroseconds(2);
 
  digitalWrite(triggerPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(triggerPin, LOW);
  pinMode(echoPin, INPUT);
 
  return pulseIn(echoPin, HIGH);
}


bool carprior = true;
unsigned long lastchange = 0;

void greenblink(int reps, int dl, int fr)
{
  for (int i = 0; i < reps ; i++ )
  {
      digitalWrite(GREEN_PED, HIGH);
      tone(BUZZER, fr);
      delay(dl);
      digitalWrite(GREEN_PED, LOW);
      noTone(BUZZER);
      delay(dl);
  }
}


void loop() { 
  
  if (carprior == true) 
  {

    if (digitalRead(BUTTON) == LOW)
    {
      while((millis() - lastchange) < 4000){delay(1);}

      delay(500);

      digitalWrite(GREEN_CAR, LOW);
      digitalWrite(YELLOW_CAR, HIGH); 
      delay(1500);

      digitalWrite(YELLOW_CAR, LOW);
      digitalWrite(RED_CAR, HIGH);
      delay (1000);

      digitalWrite(RED_PED, LOW);
      digitalWrite(GREEN_PED, HIGH);
        

      carprior = false;
      lastchange = millis();
    }

  }

  else
  {
    
    if ( ( (millis() - lastchange) > 10000) || (  ( (0.01723 * readUltrasonicDistance(5, 5)) <= 20) && ( (millis() - lastchange) > 4000)  )  )
    {

      greenblink(6, 500, 2000);

      delay(60);
      digitalWrite(RED_PED, HIGH);
      digitalWrite(RED_CAR, LOW);
      digitalWrite(GREEN_CAR, HIGH);
      carprior = true;
      lastchange = millis();

    } 

    delay(100);


  }
  

}
```

## Log

<!-- write every week your progress here -->

### Week 6 - 12 May

### Week 7 - 19 May

### Week 20 - 26 May


## Reference links

<!-- Fill in with appropriate links and link titles -->
