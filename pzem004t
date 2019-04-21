#include <SoftwareSerial.h> // Arduino IDE <1.6.6
#include <SPI.h>
#include <Ethernet.h>
#include <PZEM004T.h>


//https://www.arduino.cc/en/Tutorial/SoftwareSerialExample


PZEM004T pzem(8,9);  // RX,TX
IPAddress ip(192,168,1,1);

IPAddress serverIp(192, 168, 0, 44);  // arduino static IP
byte mac[] = {
  0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED
};

EthernetServer server(80);
/*
 * PDAControl
 * Documentation PDAControl English:
 * http://pdacontrolen.com/meter-pzem-004t-with-arduino-esp32-esp8266-python-raspberry-pi/
 * Documentacion PDAControl Español:
 * http://pdacontroles.com/medidor-pzem-004t-con-arduino-esp32-esp8266-python-raspberry-pi/
 * Video Tutorial : https://youtu.be/qt32YT_1oH8
 * 
 */

void setup() {
  Serial.begin(9600);
  pzem.setAddress(ip);

  Ethernet.begin(mac, serverIp);

  // Check for Ethernet hardware present
  if (Ethernet.hardwareStatus() == EthernetNoHardware) {
    Serial.println("Ethernet shield was not found.  Sorry, can't run without hardware. :(");
    while (true) {
      delay(1); // do nothing, no point running without Ethernet hardware
    }
  }
  if (Ethernet.linkStatus() == LinkOFF) {
    Serial.println("Ethernet cable is not connected.");
  }
  
  // start the server
  server.begin();
  Serial.print("server is at ");
  Serial.println(Ethernet.localIP());
}

void loop() {

  float v = pzem.voltage(ip);   
  float i = pzem.current(ip);
  float p = pzem.power(ip);
  float e = pzem.energy(ip);

  if(v >= 0.0){ Serial.print(v);Serial.print("V; "); }
  if(i >= 0.0){ Serial.print(i);Serial.print("A; "); }
  if(p >= 0.0){ Serial.print(p);Serial.print("W; "); }
  if(e >= 0.0){ Serial.print(e);Serial.print("Wh; "); }  
  Serial.println("");
  
  // listen for incoming clients
  EthernetClient client = server.available();

  if (client) {
    Serial.println("new client");

    // an http request ends with a blank line
    boolean currentLineIsBlank = true;
    while (client.connected()) {
      if (client.available()) {
        char c = client.read();
        Serial.write(c);
        // if you've gotten to the end of the line (received a newline
        // character) and the line is blank, the http request has ended,
        // so you can send a reply

        if(v < 0.0){
          client.println("HTTP/1.1 500 Internal Server Error");
          Serial.println("responsed 500 Internal Server Error");
          break;
        } else {
        
          if (c == '\n' && currentLineIsBlank) {
            // send a standard http response header
            client.println("HTTP/1.1 200 OK");
            client.println("Content-Type: application/json;charset=utf-8");
            client.println("Connection: close");  // the connection will be closed after completion of the response
            client.println("Refresh: 5");  // refresh the page automatically every 5 sec
            client.println();
            
            client.print("{\"variables\" : {\"voltage\" : ");
            client.print(v);
            client.print(", \"current\" : ");
            client.print(i);
            client.print(", \"power\" : ");
            client.print(p);
            client.print(", \"energy\" : ");
            client.print(e);
            client.print("}, \"id\" : \"energymeter01\", \"name\" : \"energymeter01\", \"connected\" : true } ");
            client.println("");
  
            break;
          }
        }
        if (c == '\n') {
          // you're starting a new line
          currentLineIsBlank = true;
        } else if (c != '\r') {
          // you've gotten a character on the current line
          currentLineIsBlank = false;
        }
      }
    }
    // give the web browser time to receive the data
    delay(1);
    // close the connection:
    client.stop();
    Serial.println("client disconnected");
  }  
  delay(100);
}
