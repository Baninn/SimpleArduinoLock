#include <SPI.h>
#include <MFRC522.h>
#include <Stepper.h>
#define RST_PIN         9          // Configurable, see typical pin layout above
#define SS_1_PIN        10         // Configurable, take a unused pin, only HIGH/LOW required, must be diffrent to SS 2
#define SS_2_PIN        8          // Configurable, take a unused pin, only HIGH/LOW required, must be diffrent to SS 1
#define NR_OF_READERS   2
#define STEPS 64
#define Buzzer 4
#define CM 1      //Centimeter
#define INC 0     //Inch
#define TP 2      //Trig_pin
#define EP 3      //Echo_pin
byte ssPins[] = {SS_1_PIN, SS_2_PIN};
Stepper stepper(STEPS, 8, 6, 7, 5);
//MFRC522 mfrc522[1];   // Create MFRC522 instance.
MFRC522 mfrc522[NR_OF_READERS]; 
unsigned long lockOn=5000,on=0;
void setup() 
{
  pinMode(TP,OUTPUT);       // set TP output for trigger  
  pinMode(EP,INPUT);        // set EP input for echo
  pinMode(Buzzer,OUTPUT);  //设置蜂鸣器输出引脚
  stepper.setSpeed(210);
  Serial.begin(9600); // Initialize serial communications with the PC
  while (!Serial);    // Do nothing if no serial port is opened (added for Arduinos based on ATMEGA32U4)
  SPI.begin();        // Init SPI bus
  Serial.println("Start");
}

void loop() 
{
   
  for (uint8_t reader = 0; reader < NR_OF_READERS; reader++) //RFID
  {
    if (mfrc522[reader].PICC_IsNewCardPresent() && mfrc522[reader].PICC_ReadCardSerial()) 
    {
      Serial.print(F("Card UID:"));
      dump_byte_array(mfrc522[reader].uid.uidByte, mfrc522[reader].uid.size);
      mfrc522[reader].PICC_HaltA();//错误的时候只读一次
    } 
  } 
  
  while(Serial.available())//蓝牙
  {
    char c=Serial.read();
      if(c=='1')
      {
        Serial.println("BT is ready!"); // 返回到手机调试程序上  
        Serial.write("顺时针");
        stepper.step(1024);
        lockOn=millis()+10000;
      }
     if(c=='2')
     {
       Serial.write("逆时针");
       stepper.step(-1024);
     }
  } 
  long microseconds = TP_init();//超声波
  long distacne_cm = Distance(microseconds, CM);
  unsigned long nowtime=millis();
  if(distacne_cm>=10)
  {   
    if(nowtime>lockOn)
    { 
       lockOn=nowtime+5000;             //记录当前时间长度，第一次为5000ms,赋值给lockOn
       //lockOff=nowtime+5000;  
       Serial.print("门未关好");
       Serial.print("Distacne_CM = ");
       Serial.println(distacne_cm);
       //Buzzer_Di();
    }
      //lockOn=lockOn+5000;
  }
}

void dump_byte_array(byte *buffer, byte bufferSize) 
{
  String card;
  for (byte i = 0; i < bufferSize; i++) 
  {
    Serial.print(buffer[i] < 0x10 ? " 0" : " ");
    Serial.print(buffer[i], HEX);   
    card.concat(buffer[i]); 
    card.concat(" ");   
  }
  Serial.print("\n");
  Serial.println(card);

  if(card=="37 253 250 55 "||card=="21 36 26 107 ")
  {
    if(on%2==0)
    {
      Serial.println("open"); 
      //stepp();
      stepper.step(3072);
    }
    if(on%2==1)
    {
      Serial.println("close"); 
      //stepp();
      stepper.step(-3072);
    }
    on++;
    lockOn=millis()+10000;
   }
   else
   {
    Serial.println("NO"); 
    Buzzer_Di();
    //蜂鸣器响  
   }
}
/*
void stepp()//步进电机
{
     Serial.println("顺开");
     stepper.step(1024);

     Serial.println("逆关");
     stepper.step(-1024);
}
*/
void Buzzer_Di()
{
    digitalWrite(Buzzer,HIGH); //蜂鸣器响
    delay(200);     //延时
    digitalWrite(Buzzer,LOW); //蜂鸣器关闭
    //delay(200);     //延时
}

long Distance(long time, int flag)
{
  long distacne;
  if(flag)
    distacne = time /29 / 2  ;     // Distance_CM  = ((Duration of high level)*(Sonic :340m/s))/2
  else
    distacne = time / 74 / 2;      // INC
  return distacne;
}

long TP_init()
{                     
  digitalWrite(TP, LOW);                    
  delayMicroseconds(2);
  digitalWrite(TP, HIGH);                 // pull the Trig pin to high level for more than 10us impulse 
  delayMicroseconds(10);
  digitalWrite(TP, LOW);
  long microseconds = pulseIn(EP,HIGH);   // waits for the pin to go HIGH, and returns the length of the pulse in microseconds
  return microseconds;                    // return microseconds
}
