# projet-robot
 /*******************************************************************************
 Projektname:       ACS I.cprj
 Benötigte Libs´s:  IntFunc_lib.cc
 Routinen:          ASC_I.cc
 Autor:             UlliS
 Datum:             03.02.2009

 Funktion:          PRO-BOT128 fährt umher, und weicht dabei Hindernisse über
                    das ACS (Anti Collisions System) aus.
                    Der Roboter wird in diesem Demo immer eine leichte Kurve
                    fahren, da hier keine Motor-Gleichlaufregelung vorhanden ist.
                    Sollte PRO-BOT128 Hindernisse nicht erkennen, oder das ACS System
                    ständig auch ohne Hindernisse regieren, muss ACS_Init()
                    verändert werden! (Bereich von 1 bis 20)
                    Hier sollte man die Parameter Schritt für Schritt auf die
                    richtigen Werte trimmen.

*******************************************************************************/

#define PWM_IR 35
#define IR_left 27
#define IR_right 29
#define TSOP 26

#define FLL 19
#define FLR 18
#define BLL 17
#define BLR 16

#define Motor_Enable 15

byte ACS_R, ACS_L;


void main(void)
{

    ACS_Init(5);        //ACS setup / sensitivity 1 To 20 / 1=near / 20 =far
    LED_Init();         //Flash LEDs
    Motor_Init();       //Motor setup
    AbsDelay(1000);     //Wait 1Sec.

    do                  //Endless Loop
     {
       // Check_Left();   //Check IR Sensor "ACS" left
        //Check_Right();  //Check IR Sensor "ACS" right
         Forward();
        Status_LEDS();  //Switsch Status LEDs8

        //Drive behaviour
       // if ((ACS_L == 1) && (ACS_R == 1)) Forward();
        //if ((ACS_L == 0) && (ACS_R == 0)) Backward();
       // if ((ACS_L == 1)&& (ACS_R == 0)) Turn_Left();
        if ((ACS_L == 0) && (ACS_R == 1)) Turn_Right();

     } while (1);

}



void Motor_Init(void)
{
   Timer_T1PWMX(256,128,128,PS_8);               //Setting up PWM channel A und B Timer1
    Port_DataDirBit(Motor_Enable,PORT_OUT);      //Port Enable Motor = Output
    Port_WriteBit(Motor_Enable,1);               //Port = High +5V
    Timer_T1PWA(128);                            //Motor stop!
    Timer_T1PWB(128);                            //Motor stop!
}


void LED_Init(void)
{
    Port_DataDirBit(FLL,PORT_OUT);               //Port PC.0 = Output
    Port_DataDirBit(FLR,PORT_OUT);               //Port PC.1 = Output
    Port_DataDirBit(BLL,PORT_OUT);               //Port PC.2 = Output
    Port_DataDirBit(BLR,PORT_OUT);               //Port PC.3 = Output
    Port_Write(2,0x00);                          //All LEDs "ON"
    AbsDelay(500);                               //Wait 1Sec.
    Port_Write(2,0x1F);                          //All LEDs "OFF"
    AbsDelay(500);                               //Wait 1Sec.
    Port_Write(2,0x00);                          //All LEDs "ON"
    AbsDelay(500);                               //Wait 1Sec.
    Port_Write(2,0x1F);                          //All LEDs "OFF"
}


void ACS_Init(byte sensitivity)
{
    Port_DataDirBit(IR_left,PORT_OUT);
    Port_DataDirBit(IR_right,PORT_OUT);
    Port_DataDirBit(TSOP,PORT_IN);

    //Calculating the pulse width modulation
    //Timer_T3PWM(Par1,Par2,PS);
    //Period=Par1*PS/FOSC (51*8/14,7456MHz=27,66 µs)  = 36Khz
    //Pulse=Par2*PS/FOSC (25*8/14,7456MHz=13,56 µs) On Time

    //Timer_T3PWM(Word period,Word PW0,Byte PS)  --> 36Khz
    //Timer_T3PWM(51,sensitivity,PS_8);   //with Par1, Par2 can reach altered!
                                        //Responding To the ACS must be sensitive To these parameters are screwed!
}


void Check_Right(void)
{
    Port_WriteBit(IR_left,PORT_OFF);     //Check right side
    Port_WriteBit(IR_right,PORT_ON);
    AbsDelay(5);
    ACS_R = Port_ReadBit(TSOP);
    Port_WriteBit(IR_right,PORT_OFF);
}


void Check_Left(void)
{
    Port_WriteBit(IR_right,PORT_OFF);     //Check left side
    Port_WriteBit(IR_left,PORT_ON);
    AbsDelay(5);
    ACS_L = Port_ReadBit(TSOP);
    Port_WriteBit(IR_left,PORT_OFF);
}


void Forward(void)                        //Drive forward
{
    Timer_T1PWA(220);
    Timer_T1PWB(220);
    AbsDelay(250);
}


void Backward(void)                       //Drive backward
{
    Timer_T1PWA(180);
    Timer_T1PWB(30);
    AbsDelay(1000);
}


void Turn_Left(void)                      //Turn left
{
    Timer_T1PWA(30);
    Timer_T1PWB(180);
    AbsDelay(150);
}


void Turn_Right(void)                     //Turn right
{
    Timer_T1PWA(180);
    Timer_T1PWB(30);
    AbsDelay(150);
}


void Status_LEDS(void)                    //Change Status LEDs
{
    if (ACS_L == 0)
      {
       Port_WriteBit(BLL,PORT_ON);
      }
      else Port_WriteBit(BLL,PORT_OFF);

    if (ACS_R == 0)
      {
       Port_WriteBit(BLR,PORT_ON);
      }
      else Port_WriteBit(BLR,PORT_OFF);
}

