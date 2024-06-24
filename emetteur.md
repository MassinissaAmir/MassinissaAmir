#include <SPI.h>
#include "RF24.h"

// PROGRAMME EMETTEUR

/*  PINOUT

-----  NRF  -----

CE    D9
CS    D10
SCK   D13
MOSI  D11
MISO  D12

----------------

LIEN DOC
https://nrf24.github.io/RF24/classRF24.html#

*/


#define CE_PIN 9
#define CSN_PIN 10

#define RF_POWER 3 //chiffre de 0 Ã  3 du minimum au maximum d'Ã©mission
#define NUM_RADIO 0 //dÃ©finit le numÃ©ro de la paire de NRF (ne pas utiliser encore)

//init radio
RF24 radio(CE_PIN, CSN_PIN);

// dÃ©finition des adresses possibles (des tunnels)
uint8_t address[][6] = { "Avi01", "Avi02" };

//DÃ©finition du Payload de 2 octets max
uint8_t payload[2];

void setup() {
  Serial.begin(9600);
  
  if (!radio.begin()) {
    Serial.println(F("Pas de rÃ©ponse du NRF"));
    while (1) {}  // hold in infinite loop
  }
  radio.setRadiation(RF_POWER,2,1); //puissance dÃ©finie utilisateur, dÃ©bit 250kb/s, ampli antenne ON
  radio.setPayloadSize(2); // Taille systÃ©matique de 2 octets : 0xXX commande, 0xXX valeur
  radio.enableAckPayload(); //Payload de vÃ©rification que l'avion a bien reÃ§u

  radio.openWritingPipe(address[0]);  // always uses pipe 0
  radio.openReadingPipe(1, address[1]);  // using pipe 1

  radio.setRetries(10,5); //10*250us=2.5ms entre chaque essai d'envoi, pour 5 essais

  payload[1]=0xBE;

  radio.stopListening();  //Mode emetteur
}

void loop() {

  payload[0]=0x44;  //code d'envoi (axe, bouton, etc)
  
  for(uint8_t i=0; i<180; i++){
    payload[1]= i;  //Envoi valeur apres code (angle, Ã©tat bouton)
    radio.write(&payload, sizeof(payload));
    delay(10);
  }

}
