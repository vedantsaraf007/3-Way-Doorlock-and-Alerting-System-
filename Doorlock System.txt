#include <Keypad.h>                 
#include <SPI.h>                    
#include <MFRC522.h>
#include <LiquidCrystal_I2C.h>
#include <Wire.h>

LiquidCrystal_I2C lcd(0x27, 20, 4);

#define SS_PIN 53                   
#define RST_PIN 5                   
MFRC522 mfrc522(SS_PIN, RST_PIN);   

int RedLed    = 15;                 
int GreenLed  = 16;              
int Blueled   = 17;
int lock      = 9;
int Buzzer    = 18;                
unsigned long Timer;               



const byte ROWS = 4;                
const byte COLS = 3;              


char Keys[ROWS][COLS] = {           
  {'1', '2', '3',},
  {'4', '5', '6',},
  {'7', '8', '9',},
  {'*', '0', '#',}
};

byte rowPins[ROWS] = {43, 41, 39, 37};
byte colPins[COLS] = {35, 33, 31};   
Keypad keypad = Keypad( makeKeymap(Keys), rowPins, colPins, ROWS, COLS); 

int RightCard;                     
int RightPinCode;                   
int WrongPinCode;                  
int PinCodeCounter;                

int Code1Correct;                   
int Code2Correct;                  
int Code3Correct;                 
int Code4Correct;                  
int Code5Correct;                  
int Code6Correct;                
int Reset;                          

int MifareCard1;                             
const int Code1MifareCard1 = '1';              
const int Code2MifareCard1 = '2';            
const int Code3MifareCard1 = '3';         
const int Code4MifareCard1 = '4';            
const int Code5MifareCard1 = '5';              
const int Code6MifareCard1 = '6';              

int MifareCard2;                               
const int Code1MifareCard2 = '6';            
const int Code2MifareCard2 = '5';            
const int Code3MifareCard2 = '4';              
const int Code4MifareCard2 = '3';            
const int Code5MifareCard2 = '2';              
const int Code6MifareCard2 = '1';             

void setup()

{
  Serial.begin(9600);                                    
  SPI.begin();                                            
  mfrc522.PCD_Init();                                     
  pinMode (RedLed, OUTPUT);                               
  pinMode (GreenLed, OUTPUT);                           
  pinMode (Blueled, OUTPUT);
  pinMode (lock, OUTPUT);
  pinMode (Buzzer, OUTPUT);
  lcd.backlight();
  lcd.init();
  lcd.setCursor(0, 0);
  lcd.print("    Welcome To");
  lcd.setCursor(0, 1);
  lcd.print(" JustDoElectronics");
  delay(5000);
  lcd.setCursor(1, 3);
  lcd.print(" Door lock :-   ");
  lcd.setCursor(13, 3);
  lcd.print("CLOSE");

}





void loop() {


  if (Reset == 1)                                           
    RightCard = 0;
    MifareCard1 = 0;
    MifareCard2 = 0;
    RightPinCode = 0;
    WrongPinCode = 0;
    Code1Correct = 0;
    Code2Correct = 0;
    Code3Correct = 0;
    Code4Correct = 0;
    Code5Correct = 0;
    Code6Correct = 0;
    PinCodeCounter = 0;
    delay (50);
    Reset = 0;
  }

  if (millis() - Timer > 7000 && RightCard == 1)          
  { 
    Reset = 1;
    Serial.println("CardAccesOff");
  }

  if   (mfrc522.PICC_IsNewCardPresent() &&
        mfrc522.PICC_ReadCardSerial())
  {

    if
    (mfrc522.uid.uidByte[0] == 0x61   &&                
        mfrc522.uid.uidByte[1] == 0x7E   &&                
        mfrc522.uid.uidByte[2] == 0x1F   &&               
        mfrc522.uid.uidByte[3] == 0x69)                    

    {
      RightCard = 1;                                     
      MifareCard1 = 1;                                   
      digitalWrite (Buzzer, HIGH);                       
      delay (150);                                       
      digitalWrite (Buzzer, LOW);                        

      PinCodeCounter = 0;                               
      Timer =  millis();                                 
      Serial.println("CardAccesOn");                     
      delay (200);                                       
    }
    if
    (mfrc522.uid.uidByte[0] == 0xD1   &&                 
        mfrc522.uid.uidByte[1] == 0x91   &&                 
        mfrc522.uid.uidByte[2] == 0x1E   &&                 
        mfrc522.uid.uidByte[3] == 0x69)                   

    {
      RightCard = 1;                                       
      MifareCard2 = 1;
      digitalWrite (Buzzer, HIGH);                         
      delay (150);                                         
      digitalWrite (Buzzer, LOW);                          

      PinCodeCounter = 0;                                  
      Timer =  millis();                                   
      Serial.println("CardAccesOn");                       
      delay (200);                                         
    }

  }

  if (Code6Correct == 1 && RightCard == 1)                   
  {
    RightPinCode = 1;                              
    lcd.clear();
    lcd.setCursor(1, 2);
    lcd.print(" Door lock :-   ");
    lcd.setCursor(13, 2);
    lcd.print("OPEN");
    digitalWrite (GreenLed, HIGH);                         
    digitalWrite (Blueled, HIGH);
    digitalWrite (lock, HIGH);
    delay (2000);                                            
    digitalWrite (Buzzer, HIGH);                            
    delay (150);                                            
    digitalWrite (Buzzer, LOW);                             
    delay (50);                                             
    digitalWrite (Buzzer, HIGH);                            
    delay (150);                                            
    digitalWrite (Buzzer, LOW);                             
    delay (500);                                           
    digitalWrite (GreenLed, LOW);
    digitalWrite (Blueled, LOW);
    digitalWrite (lock, LOW); //
    lcd.clear();
    lcd.setCursor(6, 1);
    lcd.print("Plz Visit ");
    lcd.setCursor(8, 2);
    lcd.print("Again");
    delay(5000);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("    Welcome To");
    lcd.setCursor(0, 1);
    lcd.print(" JustDoElectronics");
    lcd.setCursor(1, 3);
    lcd.print(" Door lock :-   ");
    lcd.setCursor(13, 3);
    lcd.print("CLOSE");

    Serial.println("Correct PinCode");                      
    Reset = 1;                                            
  }


  if ((Code6Correct == 0) && (PinCodeCounter >= 6) && (RightCard == 1))
  {
    WrongPinCode = 1;                                                       
    Serial.println("WrongCode");                                             
    Reset = 1;                                                              
  }


  if ((WrongPinCode == 1) || (millis() - Timer > 7000 && RightCard == 1))    
  {
    digitalWrite (Buzzer, HIGH);                                            
    digitalWrite (RedLed, HIGH);                                           
    delay(1500);                                                            
    digitalWrite (Buzzer, LOW);                                             
    digitalWrite (RedLed, LOW);                                             

    Serial.println("WrongCode or Timer expired");                           
    Reset = 1;                                                              
  }


  char KeyDigit = keypad.getKey();                                         

  if ((RightCard == 1) &&                                                    
      ((KeyDigit == '1') ||
       (KeyDigit == '2')  ||
       (KeyDigit == '3')  ||
       (KeyDigit == '4')  ||
       (KeyDigit == '5')  ||
       (KeyDigit == '6')  ||
       (KeyDigit == '7')  ||
       (KeyDigit == '8')  ||
       (KeyDigit == '9')  ||
       (KeyDigit == '0')  ||
       (KeyDigit == '*')  ||
       (KeyDigit == '#')))

  {
    PinCodeCounter++;                                                       
    digitalWrite (Buzzer, HIGH);                                            
    delay (50);                                                             
    digitalWrite (Buzzer, LOW);                                             
  }


  if ((KeyDigit == Code1MifareCard1) && (RightCard == 1) && (Code1Correct == 0) && (MifareCard1 == 1))           
    Code1Correct = 1;                                                                                        
    return;                                                                                                  
  }

  if ((KeyDigit == Code2MifareCard1) && (Code1Correct == 1) && (Code2Correct == 0) && (MifareCard1 == 1))       
    Code2Correct = 1;                                                                                        
    return;                                                                       
  }

  if ((KeyDigit == Code3MifareCard1) && (Code2Correct == 1) && (Code3Correct == 0) && (MifareCard1 == 1))        
  {
    Code3Correct = 1;                                                                                        
    return;                                                                                                   

  if ((KeyDigit == Code4MifareCard1) && (Code3Correct == 1) && (Code4Correct == 0) && (MifareCard1 == 1))       
  {
    Code4Correct = 1;                                                                                       
    return;                                                                                                  
  }
  if ((KeyDigit == Code5MifareCard1) && (Code4Correct == 1) && (Code5Correct == 0) && (MifareCard1 == 1))         
    Code5Correct = 1;                                                                                         
    return;                                                                                                   
  }

  if ((KeyDigit == Code6MifareCard1) && (Code5Correct == 1) && (Code6Correct == 0) && (MifareCard1 == 1))     
  {
    Code6Correct = 1;                                                                                         
    return;                                                             
  }





  if ((KeyDigit == Code1MifareCard2) && (RightCard == 1) && (Code1Correct == 0) && (MifareCard2 == 1))              
  {
    Code1Correct = 1;                                                                                           
    return;                                                                                                     
  }

  if ((KeyDigit == Code2MifareCard2) && (Code1Correct == 1) && (Code2Correct == 0) && (MifareCard2 == 1))          
  {
    Code2Correct = 1;                                                                                          
    return;                                                                                                    
  }

  if ((KeyDigit == Code3MifareCard2) && (Code2Correct == 1) && (Code3Correct == 0) && (MifareCard2 == 1))           
  {
    Code3Correct = 1;                                                                                          
    return;                                                                                                     
  }

  if ((KeyDigit == Code4MifareCard2) && (Code3Correct == 1) && (Code4Correct == 0) && (MifareCard2 == 1))          
  {
    Code4Correct = 1;                                                                                        
    return;                                                                                                    
  }
  if ((KeyDigit == Code5MifareCard2) && (Code4Correct == 1) &&  (Code5Correct == 0) && (MifareCard2 == 1))         
  {
    Code5Correct = 1;                                                                                          
    return;                                                                                                    
  }

  if ((KeyDigit == Code6MifareCard2) && (Code5Correct == 1) &&  (Code6Correct == 0) && (MifareCard2 == 1))        
  {
    Code6Correct = 1;                                                                                          
    return;                                                                                                    
  }

}