# Elec1601-report
int shieldPairNumber = 5;
boolean ConnStatusSupported = true;
String slaveName = "Slave";
String masterNameCmd = "\r\n+STNA=Master";
String connectCmd = "\r\n+CONN=";
int nameIndex = 0;
int addrIndex = 0;
String recvBuf;
String slaveAddr;
String retSymb = "+RTINQ=";   
SoftwareSerial blueToothSerial(RxD,TxD);
//JoyStick setup
const int VRxPin = A0;
const int VRyPin = A1;
const int SWPin = 2;
int VRx = 0;
int VRy = 0;
int SW = 0;
void setup()
{

    Serial.begin(9600);
    blueToothSerial.begin(38400);

    pinMode(RxD, INPUT);
    pinMode(TxD, OUTPUT);
    pinMode(ConnStatus, INPUT);



    if(ConnStatusSupported) Serial.println("Checking Master-Slave connection status.");

    if(ConnStatusSupported && digitalRead(ConnStatus)==1)
    {
        Serial.println("Already connected to Slave - remove USB cable if reboot of Master Bluetooth required.");
    }
    else
    {
        Serial.println("Not connected to Slave.");

        setupBlueToothConnection();     // Set up the local (master) Bluetooth module
        getSlaveAddress();              // Search for (MAC) address of slave
        makeBlueToothConnection();      // Execute the connection to the slave

        delay(1000);                    // Wait one second and flush the serial buffers
        Serial.flush();
        blueToothSerial.flush();
    }

    //Joystick Setup
    pinMode(VRxPin, INPUT);
    pinMode(VRyPin, INPUT);
    pinMode(SWPin, INPUT_PULLUP);
}


void loop()
{
    char recvChar;
    char test = 'S';

    while(1)
    {
        if(blueToothSerial.available())   // Check if there's any data sent from the remote Bluetooth shield
        {
            recvChar = blueToothSerial.read();
            Serial.print(recvChar);
        }
          VRx = analogRead(VRxPin);
          VRy = analogRead(VRyPin);
          SW = digitalRead(SWPin);
          if (VRx > 900) {
            test = 'R';
          } else if (VRx < 50) {
            test = 'L';
          } else if (VRy < 550) {
            test = 'B';
          } else if (VRy > 700) {
            test = 'F';
          } else if (SW == 0) {
            test = 'M';
          }
          else {
            test = 'S';
          }
          blueToothSerial.print(test);
          Serial.println(test);
          delay(250);
          Serial.println(VRx);
          Serial.println(VRy);
          Serial.println(SW);
    }
}


void setupBlueToothConnection()
{
    Serial.println("Setting up the local (master) Bluetooth module.");

    masterNameCmd += shieldPairNumber;
    masterNameCmd += "\r\n";

    blueToothSerial.print("\r\n+STWMOD=1\r\n");
    blueToothSerial.print(masterNameCmd);
    blueToothSerial.print("\r\n+STAUTO=0\r\n");


    blueToothSerial.flush();
    delay(2000);

    blueToothSerial.print("\r\n+INQ=1\r\n");

    blueToothSerial.flush();
    delay(2000);

    Serial.println("Master is inquiring!");
}


void getSlaveAddress()
{
    slaveName += shieldPairNumber;

    Serial.print("Searching for address of slave: ");
    Serial.println(slaveName);

    slaveName = ";" + slaveName;   

    char recvChar;



    while(1)
    {
        if(blueToothSerial.available())
        {
            recvChar = blueToothSerial.read();
            recvBuf += recvChar;

            nameIndex = recvBuf.indexOf(slaveName);   

            if ( nameIndex != -1 )   // ie. if slaveName was found
            {
                addrIndex = (recvBuf.indexOf(retSymb,(nameIndex - retSymb.length()- 18) ) + retSymb.length());
                slaveAddr = recvBuf.substring(addrIndex, nameIndex);

                Serial.print("Slave address found: ");
                Serial.println(slaveAddr);

                break;  // Only breaks from while loop if slaveName is found
            }
        }
    }
}


void makeBlueToothConnection()
{
    Serial.println("Initiating connection with slave.");

    char recvChar;

    // Having found the target slave address, now form the full connection command

    connectCmd += slaveAddr;
    connectCmd += "\r\n";

    int connectOK = 0;       // Flag used to indicate succesful connection
    int connectAttempt = 0;  // Counter to track number of connect attempts

    // Keep trying to connect to the slave until it is connected (using a do-while loop)

    do
    {
        Serial.print("Connect attempt: ");
        Serial.println(++connectAttempt);

        blueToothSerial.print(connectCmd);



        recvBuf = "";

        while(1)
        {
            if(blueToothSerial.available())
            {
                recvChar = blueToothSerial.read();
                recvBuf += recvChar;

                if(recvBuf.indexOf("CONNECT:OK") != -1)
                {
                    connectOK = 1;
                    Serial.println("Connected to slave!");
                    blueToothSerial.print("Master-Slave connection established!");
                    break;
                }
                else if(recvBuf.indexOf("CONNECT:FAIL") != -1)
                {
                    Serial.println("Connection FAIL, try again!");
                    break;
                }
            }
        }
    } while (0 == connectOK);

}

//slave part code
#include <SoftwareSerial.h>
#include <Servo.h>
#define RxD 7
#define TxD 6
#define ConnStatus A1
#define DEBUG_ENABLED  1
int shieldPairNumber = 5;
boolean ConnStatusSupported = true;
String slaveNameCmd = "\r\n+STNA=Slave";
SoftwareSerial blueToothSerial(RxD,TxD);
int VRx = 0;
int VRy = 0;
int SW = 0;
bool isManual = true;
Servo servoLeft;
Servo servoRight;
void setup()
{

    pinMode(A4, OUTPUT);
  pinMode(A5, INPUT);


  pinMode(A0, OUTPUT);
  pinMode(A1, INPUT);


  pinMode(A2, OUTPUT);
  pinMode(A3, INPUT);

    servoLeft.attach(13);
    servoRight.attach(12);
    Serial.begin(9600);
    blueToothSerial.begin(38400);

    pinMode(RxD, INPUT);
    pinMode(TxD, OUTPUT);
    pinMode(ConnStatus, INPUT);



    if(ConnStatusSupported) Serial.println("Checking Slave-Master connection status.");

    if(ConnStatusSupported && digitalRead(ConnStatus)==1)
    {
        Serial.println("Already connected to Master - remove USB cable if reboot of Master Bluetooth required.");
    }
    else
    {
        Serial.println("Not connected to Master.");

        setupBlueToothConnection();

        delay(1000);
        Serial.flush();
        blueToothSerial.flush();
    }
}


void loop()
{
    char recvChar;

    while(1)
    {
        if(blueToothSerial.available())
        {
            recvChar = blueToothSerial.read();
            Serial.println(recvChar);
            if (recvChar == 'M') {
                changeDriveMode();
            }
            if (isManual) {
                manualDriveHandler(recvChar);
            } else {
                autoDriveHandler();
            }
        }
    }
}

void changeDriveMode () {
    if (isManual) {
        isManual = false;
    } else if (!isManual) {
        isManual = true;
    }
}

void manualDriveHandler(char input) {
    if (input == 'F') {

        servoLeft.writeMicroseconds(1700);
        servoRight.writeMicroseconds(1300);
    } else if (input == 'B') {

        servoLeft.writeMicroseconds(1300);
        servoRight.writeMicroseconds(1700);
    } else if (input == 'L') {

        servoLeft.writeMicroseconds(1300);
        servoRight.writeMicroseconds(1300);
    } else if (input == 'R') {

        servoLeft.writeMicroseconds(1700);
        servoRight.writeMicroseconds(1700);
    } else {
        servoLeft.writeMicroseconds(1500);
        servoRight.writeMicroseconds(1500);
    }
}

void autoDriveHandler() {
  if (!readFrontSensor()) {
            servoLeft.writeMicroseconds(1700);
        servoRight.writeMicroseconds(1300);
  }
    else
  {
    decideDirection();
        servoLeft.writeMicroseconds(1500);
        servoRight.writeMicroseconds(1500);
  }
}


void setupBlueToothConnection()
{
    Serial.println("Setting up the local (slave) Bluetooth module.");

    slaveNameCmd += shieldPairNumber;
    slaveNameCmd += "\r\n";

    blueToothSerial.print("\r\n+STWMOD=0\r\n");
    blueToothSerial.print(slaveNameCmd);
    blueToothSerial.print("\r\n+STAUTO=0\r\n");
    blueToothSerial.print("\r\n+STOAUT=1\r\n");



    blueToothSerial.flush();
    delay(2000);

    blueToothSerial.print("\r\n+INQ=1\r\n");

    blueToothSerial.flush();
    delay(2000);

    Serial.println("The slave bluetooth is inquirable!");
}






boolean readFrontSensor() {
  long duration, cm;
  digitalWrite(A4, LOW);
  delayMicroseconds(5);
  digitalWrite(A4, HIGH);
  delayMicroseconds(10);
  digitalWrite(A4, LOW);


  pinMode(A5, INPUT);
  duration = pulseIn(A5, HIGH);

  cm = (duration/2) / 29.1;

  if (cm < 15) {
    return true;
  }
  return false;
}



void decideDirection() {
        long rduration, rcm;
    digitalWrite(A0, LOW);
    delayMicroseconds(5);
    digitalWrite(A0, HIGH);
    delayMicroseconds(10);
    digitalWrite(A0, LOW);
    pinMode(A1, INPUT);
    rduration = pulseIn(A1, HIGH);
    rcm = (rduration/2) / 29.1;

    long lduration, lcm;
    digitalWrite(A2, LOW);
    delayMicroseconds(5);
    digitalWrite(A2, HIGH);
    delayMicroseconds(10);
    digitalWrite(A2, LOW);
    pinMode(A3, INPUT);
    lduration = pulseIn(A3, HIGH);
    lcm = (lduration/2) / 29.1;     

    if (lcm > rcm) {
        servoLeft.writeMicroseconds(1300);
        servoRight.writeMicroseconds(1300);
    } else {
              servoLeft.writeMicroseconds(1700);
        servoRight.writeMicroseconds(1700);
      }
      delay(1000);

}

