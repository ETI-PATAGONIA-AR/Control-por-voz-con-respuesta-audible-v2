Bien, como sabrán, si bien podemos grabar y almacenar hasta 255 frases, una de las limitantes del VR3 es que solo podemos trabajar con un grupo de tan solo 7 frases precargadas al iniciar nuestro programa. Si bien es una limitante el no poder trabajar con todos los comandos de voz almacenados libremente, no es un impedimento poder acceder a ellos (los 255) cuando los necesitemos. Para ello, tenemos que recurrir a tan solo unas lineas de programacion extra para limpiar los comandos seleccionados al inciciar, y luego cargar los nuevos 7, 6 u la cantidad de comandos que quieran usar.
Si vieron el video con el procedimiento de como grabar nuestros mensajes de voz, creo que ya estamos a la altura para seguir explayando como sacarle mas jugo a este modulo.
Los espacios almacenados que implemente con SISTRAIN "n" fueron los siguientes:

```
///////////////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////////////
// GRABACIONES:
// 0 Hey Roberta
//1 alarma
//2 Cocina
//3 patio
//4 comedor
//5 living
//6 jardin
//7 temperatura
//8 riego
//9 encender
//10 apagar
//11 apagar todo
//12 tiempo
//13 5 minutos
//14 10 minutos
//15 20 minutos
//16 30 minutos
//17 panico
//18 basta
//19 luces
///////////////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////////////
```

En el setup, en la seccion donde elegimos que espacios de memoria precargar, elegimos cuales de los siete espacios queremos usar en nuestra primera parte del programa, ejemplo:

```
myVR.load(uint8_t (0));   //Hey Roberta esta guardado en la posicion 0
 myVR.load(uint8_t (1));  //1 alarma
 myVR.load(uint8_t (7));  //7 temperatura
 myVR.load(uint8_t (8));  //8 riego
 myVR.load(uint8_t (11)); //11 apagar todo
 myVR.load(uint8_t (17)); //17 panico
 myVR.load(uint8_t (19)); //19 luces
```

Con estos comandos, inicio el programa (Como no me gustas Alexa, la llame Roberta) y a medida que el VR3 va reconociendo comandos, puedo ejecutar funciones donde puedo volver a cargar otros 7 espacios de mensajes y así sucesivamente para lograr no tener la limitante de tener un programa de tan solo 7 comandos de voz.... Les muestro un extracto de mi Proyecto: "ROBERTA", que básicamente es un asistente offline para automatizar / domotizar algunas funciones de nuestros hogares...

```
///////////////////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////////////////
void loop()
    {
     // - Mapa de grabaciones -
     //0  Hey Roberta
     //1  alarma      -> 9 encender / 10 apagar   (SALIDA= Sirena)
     //7  temperatura -> LM35 + DFPLAYER mini
     //8  riego       -> 12 tiempo /13 5 minutos / 14 10 minutos / 15 20 minutos / 16 30 minutos / 9 encender /10 apagar (SALIDA= Riego)
     //17 panico      -> 18 basta   (SALIDA= Sirena)
     //19 luces       -> 2 Cocina -> 9 encender / 10 apagar  (SALIDA= Cocina)
     //               -> 3 patio  -> 9 encender / 10 apagar  (SALIDA= Patio)
     //               -> 4 comedor-> 9 encender / 10 apagar  (SALIDA= Comedor)
     //               -> 5 living -> 9 encender / 10 apagar  (SALIDA= Living)
     //               -> 6 jardin -> 9 encender / 10 apagar  (SALIDA= Jardin)
     //11 apagar todo   -> Ponemos en estado bajo todos los pines / funciones
 
     int ret;
     ret = myVR.recognize(buf, 50);
     if(ret>0)
       {
        if(buf[1] == 0)
          {
           Apagar_todo();
           digitalWrite(HEY_ROBERTA, HIGH);
           HEY_ROBERTA = true;
           Serial.println("Hey Roberta");
           digitalWrite(Te_Escucho, HIGH);
           Serial.println("Esperando orden");
           TE_ESCUCHO = true;
           myVR.clear();            //Primero debemos borrar los comandos anteriores
                                           // y luego cargar los comandos a implementar
           myVR.load(uint8_t (1));  //1 alarma
           myVR.load(uint8_t (7));  //7 temperatura
           myVR.load(uint8_t (8));  //8 riego
           myVR.load(uint8_t (11)); //11 apagar todo
           myVR.load(uint8_t (17)); //17 panico
           myVR.load(uint8_t (19)); //19 luces
          }
       if(HEY_ROBERTA)
         {
         //El buffer inicial con los espacios de mensajes precargados es de 7 posiciones (0 a 6)
          if(buf[1] == 1)    // Si VR3 escucha "Alarma", envia el numero de espacio que reconocio ("1")
            {
             ...bla bla bla
             TE_ESCUCHO = true;
            }
 
      if(buf[1] == 7)      //Temperatura
        {
         decirTemperatura();
         delay(1000); // Antirebote básico 
        }
   
     if(buf[1] == 17)                                   //Reconoce la palabra y nos envia el valor del espacio almacenado
        {                          
        ....bla bla bla
        }
     if(buf[1] == 19)                                   //Reconoce la palabra y nos envia el valor del espacio almacenado
        {                          
        ....bla bla bla
       }
      if(buf[1] == 8){                              //Reconoce la palabra y nos envia el valor del espacio donde esta  
     //guardada. Precargamos 5 comandos para poder establecer el set de tiempo para programar el tiempo
    //de riego
    
        Serial.println("Que tiempo desea regar?");
        SET_TIEMPO_RIEGO = true;
        myVR.clear();
        myVR.load(uint8_t (12));                     //Seteamos 5minuto
        myVR.load(uint8_t (13));                     //Seteamos 10 minutos
        myVR.load(uint8_t (14));                     //Seteamos 15 minutos
        myVR.load(uint8_t (15));                     //Seteamos 20 minutos
        myVR.load(uint8_t (16));                     //Seteamos 25 minutos
        myVR.load(uint8_t (9));                      //encender
        myVR.load(uint8_t (10));                     //apagar

        if ((buf[1] == 12)
          {
            tiempo=5;                 //variable para el temporizador
            BanderaRIEGO=1;
            Regar();
          }
 
      if ((buf[1] == 13)
          {
            tiempo=10;                 //variable para el temporizador
            BanderaRIEGO=1;
            Regar();
          }
 
    }
//cuando ejecuta el salto a la funcion "Regar", como cargamos anteriormente los espacios
//12 tiempo /13 5 minutos / 14 10 minutos / 15 20 minutos / 16 30 minutos / 9 encender /10 apagar, ahora espera que le digamos el tiempo y de ahi saltara a otra funcion para indicarle que inicie (SALIDA= Riego)
void Regar()
      {
       Serial.println("Iniciar/Apagar Riego?");
  
 if ((buf[1] == 9)
          {
          // aca ponemos en alto el pin de salida del riego
           }
 if ((buf[1] == 10)
          {
          //aca ponemos en bajo el pin de salida del riego y precargamos los espacios del inicio
          }
     //la bandera es para que sepa donde esta y a donde volver
       }
    }
```

Bien, entiendo que simplifique bastante el programa :silbando: , pero me gustaria que salga de ustedes un poco de creatividad tambien :no: .... Si pongo todo el programa, como ya vi en anteriores ocasiones, copian y pegan en otro lugar sin mencionar quien fue el salamin con patas que los desarrollo :facepalm: ...
Bien, la primera parte estrella de esta nueva actualizacion ya la comente; Ahora vamos a la segunda parte estrella, que basicamente fue mejorar la manera en que contestaba con audios las ordenes impartidas... Migre todo a un pequeño modulo reproductor de mp3, el "dfPLAYER mini" y como veia que contestarme cosas basicas como "Luces encendidas / Luces apagadas" ya me parecía pre-arcaico, decidí darle mas funciones para sacarle jugo a la nueva adquisición: "Le di a Roberta la opción de poder decirme la temperatura con voz!!! "
Les muestro como y ustedes podran modificarlo a futuro para sus proyectos... Recordemos las siguientes lineas en el loop el programa anterior:

```
if(buf[1] == 7)      //Temperatura
        {
         decirTemperatura();
         delay(1000); // Antirebote básico
        }
```

Bien, cuando reconoce la palabra, nos envia el numero de espacio donde esta almacenado y saltamos a la funcion "decirTemperatura();"
En esta funcion, lo que vamos hacer es leer el sensor LM35 de Temperatura, vamos a extraer esos valores (enteros y decimales) y luego vamos a reproducir los audios para que diga la temperatura...parece facil, peeerooooo o_O ...

```
void decirTemperatura()
{
  int lectura_analogica = analogRead(tempPin);  // Leemos el valor del sensor
  float voltaje = (lectura_analogica * 5.0) / 1023.0;
  float temperatura_celsius = voltaje * 100.0;  // LM35: 10mV/°C → 100°C por volt

  // Separamos parte entera y decimal
  int tempEntero = (int)temperatura_celsius;
  int tempDecimal = (int)((temperatura_celsius - tempEntero) * 100);
  if (tempDecimal < 0) tempDecimal *= -1; // Asegurar positivo

  // Limitamos rango
  if (tempEntero < -1) tempEntero = -1;
  if (tempEntero > 50) tempEntero = 50;
  if (tempDecimal > 99) tempDecimal = 99;

  // --- Reproducción por DFPlayer ---
  // "Hacen"
  myDFPlayer.play(1);
  delay(1000);

  // Parte entera de la temperatura (-1 a 50)
  if (tempEntero == -1) {
    myDFPlayer.play(2); // menos un grado
  } else {
    myDFPlayer.play(3 + tempEntero); // 0003 = 0 grados, 0004 = 1 grado...
  }
  delay(1200);

  // Parte decimal (0 a 99)
  myDFPlayer.play(100 + tempDecimal);
  delay(1200);
}
```

El problema del DFPlayer Mini, es que numera los archivos en el orden en que están grabados, no por nombre alfabético.
Cuando tengamos la miniSD, es MUY importante copiarlos uno por uno en el orden correcto.
En mi caso use nombres de archivo tipo 0001.mp3, 0002.mp3, etc; y evite carpetas (los deje a todos en la raíz de la tarjeta). Yo se que es tedioso, mas cuando me tome el trabajo de usar un TTS para generar todos los audios por que tengo una voz de Roberta media ronca :ROFLMAO:
En fin; El orden de mensajes que estableci para esta parte es:

_0001.mp3 → “Hacen” (Palabra inicial)_

Las Temperaturas enteras del –1 al 50:

_0002.mp3 → “menos un grado”_
_0003.mp3 → “cero grados”_
_0004.mp3 → “un grado”_
_0005.mp3 → “dos grados”_
_bla bla bla …_
_0053.mp3 → “cincuenta grados”_

Con esto tenemos –1 hasta 50 cubierto.
El tema decimales del 0 al 99 lo resolvi de esta manera:

_0100.mp3 → “con cero centésimas”_
_0101.mp3 → “con una centésima”_
_0102.mp3 → “con dos centésimas”_
_bla bla bla …_
_0199.mp3 → “con noventa y nueve centésimas”_

No quiero ser reiterativo, pero ya se que es tediosa esta parte y por eso quiero enfatizar en algunas cosas... Al que le gusta el durazno, se banca la pelusa... Veamos mas detenidamente los llamados y luego un ejemplo de como esta misma parte la podemos usar para decir la hora u otro dato que quieran...

**El entero se llama con:**

```
if (tempEntero == -1)
   {
    myDFPlayer.play(2);   // menos un grado
   }
   else
   {
   myDFPlayer.play(3 + tempEntero); // 0003 = cero grados, 0004 = 1 grado, ...
   }
```

**El decimal se llama con:**

```
myDFPlayer.play(100 + tempDecimal); // 0100 = cero centésimas, ...
```
o sea, citemos un ejemplo par ser mas practicos:

Si el sensor marca 23.45 °C

_Reproduce 0001.mp3 → “Hacen”_
_Reproduce 0026.mp3 → “veintitrés grados”_
_Reproduce 0145.mp3 → “con cuarenta y cinco centésimas”_

Si el sensor marca –1.20 °C

_Reproduce 0001.mp3 → “Hacen”_
_Reproduce 0002.mp3 → “menos un grado”_
_Reproduce 0120.mp3 → “con veinte centésimas”_

Para el caso de implementar una RTC para decir la hora, podria ser:

```
#include <Wire.h>
#include "RTClib.h"
#include "SoftwareSerial.h"
#include "DFRobotDFPlayerMini.h"

RTC_DS1307 rtc;
SoftwareSerial mySerial(10, 11); // RX, TX para DFPlayer
DFRobotDFPlayerMini myDFPlayer;

#define BUTTON_PIN 2   //para este ejemplo usamos un pulsador para que diga la hora

void setup() {
  pinMode(BUTTON_PIN, INPUT_PULLUP); // Pulsador a GND
  Serial.begin(9600);
  mySerial.begin(9600);

  if (!rtc.begin()) {
    Serial.println("No se encuentra RTC");
    while (1);
  }
  if (!rtc.isrunning()) {
    Serial.println("RTC detenido, ajustando fecha/hora...");
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  }

  if (!myDFPlayer.begin(mySerial)) {
    Serial.println("Error DFPlayer Mini");
    while (1);
  }
  myDFPlayer.volume(25); // Volumen (0-30)
}

void loop() {
  if (digitalRead(BUTTON_PIN) == LOW) { // Pulsador presionado
    decirHora();
    delay(1000); // Antirebote básico
  }
}

void decirHora() {
  DateTime now = rtc.now();
  int hora = now.hour();
  int minuto = now.minute();
  // "Son las"
  myDFPlayer.play(1);
  delay(1200);
  // Hora
  myDFPlayer.play(2 + hora);  // porque 0002.mp3 es "cero horas"
  delay(1500);
  // "y"
  myDFPlayer.play(30);
  delay(800);
  // Minutos
  myDFPlayer.play(31 + minuto); // porque 0031.mp3 es "cero minutos"
  delay(1500);
}
```

Bien, les comparto a modo adelanto un VIDEO ( https://youtu.be/N6YvQgb3dJE?si=L2i9OlHFikIc32-q )

[![CtrlVOZ_VR3_v2](https://img.youtube.com/vi/N6YvQgb3dJE/0.jpg)](https://youtu.be/N6YvQgb3dJE)

El codigo de fuente completo no lo comparto abiertamente. Puede solicitarlo via correo electronico personalmente
