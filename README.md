```cpp
#include <LiquidCrystal.h> 
#include <stdio.h> 
LiquidCrystal lcd(6, 7, 5, 4, 3, 2); 
#include <SoftwareSerial.h> 
SoftwareSerial mySerial(8,9); 
unsigned char rcv; 
char pastnumber[11]; 
int units=0,amount=0; 
int sts1=0,sts2=0,sts3=0,sts4=0,sts5=0,sts6=0; 
int itk=0; 
char gsm_msg[10]; 
char res[130]; 
void serialFlush() 
{ 
while(Serial.available() > 0) 
{ 
char t = Serial.read(); 
} 
} 
void myserialFlush() 
{ 
while(mySerial.available() > 0) 
{ 
char t = mySerial.read(); 
} 
} 
char check(char* ex,int timeout) 
{ 
int i=0; 
int j = 0,k=0; 
while (1) 
{ 
sl: 
if(mySerial.available() > 0) 
{ 
res[i] = mySerial.read(); 
if(res[i] == 0x0a || res[i]=='>' || i == 100) 
{ 
i++; 
res[i] = 0;break; 
} 
i++; 
} 
j++; 
if(j == 30000) 
{ 
k++; 
// Serial.println("kk"); 
j = 0; 
} 
if(k > timeout) 
{ 
//Serial.println("timeout"); 
return 1; 
} 
}//while 1 
if(!strncmp(ex,res,strlen(ex))) 
{ 
// Serial.println("ok.."); 
return 0; 
} 
else 
{ 
// Serial.print("Wrong "); 
// Serial.println(res); 
i=0; 
goto sl; 
} 
} 
char buff[200],k=0; 
void upload(unsigned int s1,unsigned int s2,unsigned int s3); 
char readserver(void); 
void clearserver(void); 
const char* ssid = "iotserver"; 
const char* password = "iotserver123"; 
int sti=0; 
String inputString = ""; // a string to hold incoming data 
boolean stringComplete = false; // whether the string is complete 
int mtr = A0;int button1 = A1; 
int fan = 12; 
int bulb = 11; 
int socket = 10; 
char fan_string[10]="OFF"; 
char bulb_string[10]="OFF"; 
char socket_string[10]="OFF"; 
void okcheck() 
{ 
unsigned char rcr; 
do{ 
rcr = Serial.read(); 
}while(rcr != 'K'); 
} 
void setup() 
{ 
char ret; 
Serial.begin(1200);//serialEvent(); 
mySerial.begin(9600); 
pinMode(mtr, INPUT); 
pinMode(fan, OUTPUT);pinMode(bulb, OUTPUT);pinMode(socket, OUTPUT); 
pinMode(button1, INPUT_PULLUP); 
digitalWrite(fan, LOW);digitalWrite(bulb, LOW);digitalWrite(socket, LOW); 
//IOT smart Energymeter with GSM 
lcd.begin(16, 2);lcd.setCursor(0,0); 
lcd.print("IOT Smart Energy"); 
lcd.setCursor(0,1); 
lcd.print(" Meter with GSM"); 
delay(2500); 
wifiinit(); 
delay(2500); 
if(digitalRead(button1) == LOW) //GSM 
{ 
lcd.clear();lcd.print("GSM Initialising"); 
gsminit(); 
} 
digitalWrite(fan, HIGH);delay(1000); 
digitalWrite(bulb, HIGH);delay(1000);digitalWrite(socket, HIGH);delay(1000); 
lcd.clear(); 
lcd.setCursor(0,0); 
lcd.print("U:"); //2-3-4,0 
lcd.setCursor(5,0); 
lcd.print("A:"); //7-8-9,0 
lcd.setCursor(0,1); 
lcd.print("F:"); //2-3-4,1 lcd.setCursor(2,1); 
lcd.print("ON "); 
lcd.setCursor(5,1); 
lcd.print("B:"); //7-8-9,1 
lcd.print("ON "); 
lcd.setCursor(10,1); 
lcd.print("S:"); //12-13-14,1 
lcd.print("ON "); 
memset(fan_string,'\0',strlen(fan_string)); 
strcpy(fan_string,"ON"); 
memset(bulb_string,'\0',strlen(bulb_string)); 
strcpy(bulb_string,"ON"); 
memset(socket_string,'\0',strlen(socket_string)); 
strcpy(socket_string,"ON"); 
itk=0; 
//serialEvent(); 
} 
char bf3[50]; 
int g=0,f=0,count=0,lc=0; 
int cntmr=0; 
void loop() 
{ 
if(digitalRead(mtr) == LOW) 
{delay(200); 
while(digitalRead(mtr) == LOW); 
delay(500); 
units++; 
amount = (units * 4); 
lcd.setCursor(2,0); convertl(units); 
lcd.setCursor(7,0); convertl(amount); 
upload(units,amount,fan_string,bulb_string,socket_string); 
if(digitalRead(button1) == LOW) //GSM 
{ 
lcd.setCursor(15,0);lcd.print("U"); 
delay(4000);delay(4000);delay(4000);delay(4000); 
Serial.write("AT+CMGS=\""); 
Serial.write(pastnumber); 
Serial.write("\"\r\n"); delay(3000); 
Serial.write("U:"); 
Serial.print(units); 
Serial.write(" A:"); 
Serial.print(amount); 
Serial.write(0x1A); delay(4000);delay(4000);delay(4000);delay(4000); 
lcd.setCursor(15,0);lcd.print(" "); 
} 
} 
while(Serial.available()) 
{ 
char inChar = (char)Serial.read(); 
if(inChar == '*') 
{ 
sti=1; 
itk=0; 
memset(gsm_msg,'\0',strlen(gsm_msg)); 
} 
if(sti == 1) 
{ 
gsm_msg[itk] = inChar; 
itk++; 
} 
if(inChar == '#') 
{ 
sti=0; 
itk=0; 
stringComplete = true; 
} 
} 
if(stringComplete) 
{ 
lcd.setCursor(11,0);lcd.print(gsm_msg);lcd.print(" "); 
if(gsm_msg[1] == 's' || gsm_msg[1] == 'S') 
{ 
lcd.setCursor(15,0);lcd.print("R"); 
delay(4000);delay(4000);delay(4000);delay(4000); 
Serial.write("AT+CMGS=\""); 
Serial.write(pastnumber); 
Serial.write("\"\r\n"); delay(3000); 
Serial.write("U:"); 
Serial.print(units); 
Serial.write(" A:"); 
Serial.print(amount); 
Serial.write(0x1A); 
delay(4000);delay(4000); delay(4000);delay(4000); 
lcd.setCursor(15,0);lcd.print(" "); 
} 
if(gsm_msg[1] == '1') 
{ 
digitalWrite(fan, HIGH); 
lcd.setCursor(2,1);lcd.print("ON "); 
memset(fan_string,'\0',strlen(fan_string)); 
strcpy(fan_string,"ON"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
if(gsm_msg[1] == '2') 
{ 
digitalWrite(fan, LOW); 
lcd.setCursor(2,1);lcd.print("OFF"); 
memset(fan_string,'\0',strlen(fan_string)); 
strcpy(fan_string,"OFF"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
if(gsm_msg[1] == '3') 
{ 
digitalWrite(bulb, HIGH); 
lcd.setCursor(7,1);lcd.print("ON "); 
memset(bulb_string,'\0',strlen(bulb_string)); 
strcpy(bulb_string,"ON"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
if(gsm_msg[1] == '4') 
{ 
digitalWrite(bulb, LOW); 
lcd.setCursor(7,1);lcd.print("OFF"); 
memset(bulb_string,'\0',strlen(bulb_string)); 
strcpy(bulb_string,"OFF"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
if(gsm_msg[1] == '5') 
{ 
digitalWrite(socket, HIGH); 
lcd.setCursor(12,1);lcd.print("ON "); 
memset(socket_string,'\0',strlen(socket_string)); 
strcpy(socket_string,"ON"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
if(gsm_msg[1] == '6') 
{ 
digitalWrite(socket, LOW); 
lcd.setCursor(12,1);lcd.print("OFF"); 
memset(socket_string,'\0',strlen(socket_string)); 
strcpy(socket_string,"OFF"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
memset(gsm_msg,'\0',strlen(gsm_msg)); 
inputString = ""; 
stringComplete = false; 
} 
delay(100); 
/* 
if(digitalRead(button1) == HIGH) //WIFI 
{ 
cntmr++; 
} 
*/ 
cntmr++; 
if(cntmr >= 1000) 
{cntmr=0; 
lcd.setCursor(15, 1);lcd.print("R"); 
char ctrl = readserver(); 
if(ctrl == '1' && sts1 == 0) 
{sts1=1;sts2=0,sts3=0,sts4=0,sts5=0,sts6=0; 
digitalWrite(fan, HIGH); 
lcd.setCursor(2,1);lcd.print("ON "); 
memset(fan_string,'\0',strlen(fan_string)); 
strcpy(fan_string,"ON"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
if(ctrl == '2' && sts2 == 0) 
{sts1=0;sts2=1,sts3=0,sts4=0,sts5=0,sts6=0; 
digitalWrite(fan, LOW); 
lcd.setCursor(2,1);lcd.print("OFF"); 
memset(fan_string,'\0',strlen(fan_string)); 
strcpy(fan_string,"OFF"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
if(ctrl == '3' && sts3 == 0) 
{sts1=0;sts2=0,sts3=1,sts4=0,sts5=0,sts6=0; 
digitalWrite(bulb, HIGH); 
lcd.setCursor(7,1);lcd.print("ON "); 
memset(bulb_string,'\0',strlen(bulb_string)); 
strcpy(bulb_string,"ON"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
if(ctrl == '4' && sts4 == 0) 
{sts1=0;sts2=0,sts3=0,sts4=1,sts5=0,sts6=0; 
digitalWrite(bulb, LOW); 
lcd.setCursor(7,1);lcd.print("OFF"); 
memset(bulb_string,'\0',strlen(bulb_string)); 
strcpy(bulb_string,"OFF"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
if(ctrl == '5' && sts5 == 0) 
{sts1=0;sts2=0,sts3=0,sts4=0,sts5=1,sts6=0; 
digitalWrite(socket, HIGH); 
lcd.setCursor(12,1);lcd.print("ON "); 
memset(socket_string,'\0',strlen(socket_string)); 
strcpy(socket_string,"ON"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
if(ctrl == '6' && sts6 == 0) 
{sts1=0;sts2=0,sts3=0,sts4=0,sts5=0,sts6=1; 
digitalWrite(socket, LOW); 
lcd.setCursor(12,1);lcd.print("OFF"); 
memset(socket_string,'\0',strlen(socket_string)); 
strcpy(socket_string,"OFF"); 
upload(units,amount,fan_string,bulb_string,socket_string); 
} 
lcd.setCursor(15, 1);lcd.print(" "); 
} 
} 
/* 
void serialEvent() 
{ 
while (Serial.available()) 
{ 
char inChar = (char)Serial.read(); 
if(inChar == '*') 
{sti=1; 
inputString += inChar; 
} 
if(sti == 1) 
{ 
inputString += inChar; 
} 
if(inChar == '#') 
{sti=0; 
stringComplete = true; 
} 
} 
} 
*/ 
int readSerial(char result[]) 
{ 
int i = 0; 
while (1) 
{ 
while (Serial.available() > 0) 
{ 
char inChar = Serial.read(); 
if (inChar == '\n') 
{ 
result[i] = '\0'; 
Serial.flush(); 
return 0; 
} 
if (inChar != '\r') 
{ 
result[i] = inChar; 
i++; 
} 
} 
} 
} 
void gsminit() 
{ 
Serial.write("AT\r\n"); okcheck(); 
Serial.write("ATE0\r\n"); okcheck(); 
Serial.write("AT+CMGF=1\r\n"); okcheck(); 
Serial.write("AT+CNMI=1,2,0,0\r\n"); okcheck(); 
Serial.write("AT+CSMP=17,167,0,0\r\n"); okcheck(); 
lcd.clear(); 
lcd.print("SEND MSG STORE"); 
lcd.setCursor(0,1); 
lcd.print("MOBILE NUMBER"); 
do{ 
rcv = Serial.read(); 
}while(rcv != '*'); 
readSerial(pastnumber); 
lcd.clear(); 
lcd.print(pastnumber); 
pastnumber[10]='\0'; 
Serial.write("AT+CMGS=\""); 
Serial.write(pastnumber); 
Serial.write("\"\r\n"); delay(3000); 
Serial.write("Mobile no. registered\r\n"); 
Serial.write(0x1A); delay(4000); 
} 
char bf2[50]; 
void upload(unsigned int unt,unsigned int amt,const char *s3,const char *s4,const char *s5) 
{ 
delay(2000); 
lcd.setCursor(15, 1);lcd.print("U"); 
myserialFlush(); 
mySerial.println("AT+CIPSTART=4,\"TCP\",\"projectsfactoryserver.in\",80"); 
delay(8000); 
memset(buff,0,strlen(buff)); 
sprintf(buff,"GET 
http://projectsfactoryserver.in/storedata.php?name=iot1597&s1=%u&s2=%u&s3=%s&s4=% 
s&s5=%s\r\n\r\n",unt,amt,s3,s4,s5); 
myserialFlush(); 
sprintf(bf2,"AT+CIPSEND=4,%u",strlen(buff)); 
mySerial.println(bf2); 
delay(5000); 
myserialFlush(); 
mySerial.print(buff); 
delay(2000); 
mySerial.println("AT+CIPCLOSE"); 
lcd.setCursor(15, 1);lcd.print(" "); 
} 
char readserver(void) 
{ 
char t; 
delay(2000); 
lcd.setCursor(15, 1);lcd.print("R"); 
myserialFlush(); 
mySerial.println("AT+CIPSTART=4,\"TCP\",\"projectsfactoryserver.in\",80"); 
//http://projectsfactoryserver.in/last.php?name=amvi001L 
delay(8000); 
memset(buff,0,strlen(buff)); 
sprintf(buff,"GET http://projectsfactoryserver.in/last.php?name=iot1597L\r\n\r\n"); 
myserialFlush(); 
sprintf(bf2,"AT+CIPSEND=4,%u",strlen(buff)); 
mySerial.println(bf2); 
delay(5000); 
myserialFlush(); 
mySerial.print(buff); 
//read status 
while(1) 
{ 
while(!mySerial.available()); 
t = mySerial.read(); 
// Serial.print(t); 
if(t == '*' || t == '#') 
{ 
if(t == '#')return 0; 
while(!mySerial.available()); 
t = mySerial.read(); 
// Serial.print(t); 
delay(1000); 
myserialFlush(); 
return t; 
} 
} 
delay(2000); 
mySerial.println("AT+CIPCLOSE"); 
lcd.setCursor(15, 1);lcd.print(" "); 
delay(2000); 
return t; 
} 
void clearserver(void) 
{ 
delay(2000); 
lcd.setCursor(15, 1);lcd.print("C"); 
myserialFlush(); 
mySerial.println("AT+CIPSTART=4,\"TCP\",\"projectsfactoryserver.in\",80"); 
//sprintf(buff,"GET 
http://projectsfactoryserver.in/storedata.php?name=iot1&s10=0\r\n\r\n"); 
delay(8000); 
memset(buff,0,strlen(buff)); 
sprintf(buff,"GET 
http://projectsfactoryserver.in/storedata.php?name=iot1597&s10=0\r\n\r\n"); 
myserialFlush(); 
sprintf(bf2,"AT+CIPSEND=4,%u",strlen(buff)); 
mySerial.println(bf2); 
delay(5000); 
myserialFlush(); 
mySerial.print(buff); 
delay(2000); 
myserialFlush(); 
mySerial.println("AT+CIPCLOSE"); 
lcd.setCursor(15, 1);lcd.print(" "); 
delay(2000); 
} 
void wifiinit() 
{ 
char ret; 
st: 
mySerial.println("ATE0"); 
ret = check((char*)"OK",50); 
mySerial.println("AT"); 
ret = check((char*)"OK",50); 
if(ret != 0) 
{ 
delay(1000); 
goto st; 
} 
lcd.clear();lcd.setCursor(0, 0);lcd.print("CONNECTING"); 
mySerial.println("AT+CWMODE=1"); 
ret = check((char*)"OK",50); 
cagain: 
myserialFlush(); 
mySerial.print("AT+CWJAP=\""); 
mySerial.print(ssid); 
mySerial.print("\",\""); 
mySerial.print(password); 
mySerial.println("\""); 
if(check((char*)"OK",300))goto cagain; 
mySerial.println("AT+CIPMUX=1"); 
delay(1000); 
lcd.clear();lcd.setCursor(0, 0);lcd.print("WIFI READY"); 
} 
void converts(unsigned int value) 
{ 
unsigned int a,b,c,d,e,f,g,h; 
a=value/10000; 
b=value%10000; 
c=b/1000; 
d=b%1000; 
e=d/100; 
f=d%100; 
g=f/10; 
h=f%10; 
a=a|0x30; 
c=c|0x30; 
e=e|0x30; 
g=g|0x30; 
h=h|0x30; 
Serial.write(a); 
Serial.write(c); 
Serial.write(e); 
Serial.write(g); 
Serial.write(h); 
} 
void convertl(unsigned int value) 
{ 
unsigned int a,b,c,d,e,f,g,h; 
a=value/10000; 
b=value%10000; 
c=b/1000; 
d=b%1000; 
e=d/100; 
f=d%100; 
g=f/10; 
h=f%10; 
a=a|0x30; 
c=c|0x30; 
e=e|0x30; 
g=g|0x30; 
h=h|0x30; 
// lcd.write(a); 
// lcd.write(c); 
lcd.write(e); 
lcd.write(g); 
lcd.write(h); 
} 
```
