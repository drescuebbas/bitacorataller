# cosas que he hecho pal taller

Inicialmente usamos el programa original (original original) del colectivo Graffiti Research Lab, encontrado en el github de don "[LeonFedotov](https://github.com/LeonFedotov/L.A.S.E.R.-TAG-GRL)".

El programa al ser tan antiguo tenía un monton de errores por temas de optimización principalmente (se caía el programa por falta de memoria), por lo que me adentré en el mundo de la programación con p5.js y empecé a desarrollar un programa desde cero que cumpla con las funciones escenciales del proyecto original.

Paralelamente decidimos incorporar un potenciómetro que permitiera cambiar el grosor (o color) de la línea dibujada, por lo que nació el desafío de juntar un arduino con el programa de p5 (Y QUE FUNCIONARA INALÁMBRICAMENTE!!).

Primero hice un [programa](https://editor.p5js.org/dresCuevass/sketches/iAdq6f6w6) simple que permitía dibujar con el mouse y que al presionar la tecla "c" o "r" se cambiara el color o se borrara el dibujo, respectivamente.

Skarleth se enfocó en integrar el arduino con p5, por lo que modificó el código para que se comuniquen ambos, aparte de crear el código para el [arduino](https://app.notion.com/p/Arduino-IDE-36e4d2fe85d080ba9229d1a18913dfeb) mismo.

Todas las [demos](https://app.notion.com/p/Demo-de-dibujo-en-p5-36e4d2fe85d080f1a6cffcfe96c69d66)

Después modifiqué ambos códigos para que pudiésemos usar el potenciómetro de manera inalámbrica, con un ESP32 (un microcontrolador como los arduino, pero tiene un módulo de wifi integrado).
p5.js:
```
let ws;
let valorPot = 0;

function setup() {
    createCanvas(600, 700);
    background(20);
    colorLinea = color(60, 100, 120); // color inicial

    setTimeout(() => {
        ws = new WebSocket("ws://172.20.10.3:81");

        ws.onopen = () => console.log("Conectado al ESP32");
        ws.onclose = () => console.log("Desconectado");
        ws.onerror = (e) => console.error("Error WebSocket:", e);

        ws.onmessage = (event) => {
        let raw = event.data.trim();
        if (raw !== "") valorPot = int(raw);
        console.log("Recibido:", raw);
        };
    }, 1000);
}

function draw() {

    // if (!ws || ws.readyState !== WebSocket.OPEN) {
    // fill(255);
    // textAlign(CENTER);
    // text("Conectando...", width/2, height/2);
    // return;
    // }
//------------------------------------------------
  //elipseClaude();
  valorPote();
  dibujo();
  //cambiarColor();
  borrarFondo();
}

// FUNCIONES

function valorPote(){
    fill(0);
    rectMode(CENTER);
    rect(2, 30, 30);
    fill(255);
    textAlign(CENTER);
    textSize(16);
    text("Potenciómetro: " + valorPot, width / 2, 30);

}

function elipseClaude(){
    // Mapear 0-4095 (ADC 12 bits) a lo que necesites
    let diametro = map(valorPot, 0, 4095, 20, 350);
    fill(100, 200, 255);
    noStroke();
    ellipse(width / 2, height / 2, diametro, diametro);
}

function dibujo(){
    if (mouseIsPressed) {
        stroke(colorLinea); 
        strokeWeight(10);
        line(pmouseX, pmouseY, mouseX, mouseY);
    }
}

function borrarFondo() { 
    if (key === 'r') {
        background(20);
    }
}

function cambiarColor(){
    if (key === 'c' || key === 'C') {
        colorLinea = color(random(255), random(50), random(255)); // color aleatorio
    }
}
```

ESP32:
```
#include <WiFi.h>
#include <WebSocketsServer.h>

const char* ssid     = "dersuno";
const char* password = "2712rome";

const int potPin = 34;
WebSocketsServer webSocket(81);

void onWebSocketEvent(uint8_t client, WStype_t type, uint8_t* payload, size_t length) {
  // No necesitamos hacer nada cuando se conecta un cliente
}

void setup() {
  Serial.begin(115200);

  WiFi.begin(ssid, password);
  Serial.print("Conectando a WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConectado! IP: " + WiFi.localIP().toString());

  webSocket.begin();
  webSocket.onEvent(onWebSocketEvent);
  Serial.println("Servidor WebSocket iniciado en puerto 81");
}

void loop() {
  webSocket.loop();

  int valor = analogRead(potPin);
  String mensaje = String(valor);
  webSocket.broadcastTXT(mensaje);

  delay(20);
}


```

El programa actual, se compone de distintas funciones como detectar (a través de una webcam) el color del láser y en base a eso crear una máscara donde se ve únicamente el color detectado, también a partir de la máscara, generar el dibujo en un lienzo negro, el cual se muestra en una segunda ventana (proyector).

<!-- aún me queda por agregar, no se pierdan el próximo capítulo!! -->
