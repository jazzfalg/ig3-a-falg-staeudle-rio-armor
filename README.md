### Setup

In den entsprechenden Ordner wechseln (frontend/backend)

    cd <foldername>
    npm install

### Develop

    npm run dev

### Build

    npm run build

# Hardware Interaction (Hardware communication)

Diese Beispiele zeigen wie eine Kommunikation zwischen einem Microcontroller und App stattfinden kann.
Dabei wird die **Serialport-library** zur Kommunikation mit dem Controller und **Websocket** zur Kommunikation zwischen frontend und backend genutzt.

![Datenarchitektur](readmebilder/Datenfluss.png)

## Serialport

### Software

`Serialport.js` ist eine Library zur seriellen Kommunikation. Seriel bedeutet, dass einzelne Bits nacheinander übertragen werden.

    import { SerialPort } from "serialport";
    const port = new SerialPort({ path: devicePort, baudRate: 9600 });

Um aus Bits die am Port ankommen eine Nachricht zu machen, wird er mit einem **parser** veknüpft. Der liest bis zu einem Umbruch und wandelt die Daten zu einer lesbaren Nachricht um.

    import { ReadlineParser } from "serialport";

    let parser = port.pipe(new ReadlineParser());

Da der serielle Port nur auf Clientseite existiert, führen wir den Prozess auf dem Client aus. Über einen **WebSocket** werden die Daten dann an das frontend geschickt.

- [SerialPort Docs](https://serialport.io/docs/)


### Frontend

Dieser Code ist eine visuelle Darstellung einer Bizepsübung, bei dem die Bewegung des Arms des Benutzers auf eine Figur im Spiel abgebildet wird.

Der Code wurde in JavaScript mit der p5.js-Bibliothek und dem Svelte-Framework geschrieben und erfordert die Importierung der Pakete 'p5-svelte' und '$app/environment'.

Das Spiel verfügt über folgende Funktionen:

- Visuelle Darstellung der Arm-Bewegung
- Timer, der die Dauer der Übung verfolgt
- Anzeige des Pulses (Herzfrequenz) und der verbrannten Kalorien
- Geschwindigkeitsdetektor, der den Benutzer warnt, wenn er zu schnell bewegt
- Das Spiel verwendet Bilder für den Hintergrund, den Körper, die Bizepse und den Unterarm. Die Bilder werden mit der p5.loadImage()-Funktion geladen.
- Die Bewegung des Arms des Benutzers wird im Spiel durch den Rotationswinkel des Unterarms dargestellt, der auf den Winkel des Potentiometer-Werts (Potentiometer-Wert) im echten Leben abgebildet wird. Der Timer startet, wenn der Potentiometer-Wert größer als 120 ist. Der Puls- und Kalorienanzeige werden in regelmäßigen Abständen aktualisiert, und der -Geschwindigkeitsdetektor warnt den Benutzer, wenn er zu schnell bewegt.

Das Spiel wird mit der p5.draw()-Funktion und der segment()-Funktion gezeichnet, die den Unterarm mit den p5.translate()- und p5.rotate()-Funktionen zeichnet. Die Hintergrund-, Körper- und Bizepsbilder des Spiels werden ebenfalls in der p5.draw()-Funktion gezeichnet.

### Hardware

Auf Seite des Arduinos lesen und schreiben wir über die serielle Schnittstelle#

**⚠️ Achtung:** während der Port belegt ist, bspw. durch das serialport-script, kann nichts auf das board geladen werden

    String inputString;

    void setup() {
        Serial.begin(9600);
    }
    void loop() {
        //Empfangen:
        while (Serial.available()) {
            inputString = Serial.readStringUntil('\n');
        }

        //Senden
        Serial.println("Hello from Arduino");
    }

---

## WebSocket

WebSocket ist ein Netzwerkprotokoll, das eine Verbindung zwischen **Webanwendung** (Client) und einem **WebSocket-Server** ermöglicht.

- [ws-library Docs](https://github.com/websockets/ws)
- [WebSocket Wikipedia](https://de.wikipedia.org/wiki/WebSocket)

### Server

Um den Server zu erstellen, verwenden wir die `ws-library`.

    import { WebSocketServer } from "ws";
    const wss = new WebSocketServer({ port: 8080 });

**Senden:** Dieser Server sendet in unserem Fall, jedes mal wenn am Port eine Nachricht ankommt, diese an den Client weiter.

    parser.on("data", (data) => {
        if (websocketClient !== undefined) {
            websocketClient.send(JSON.stringify({ connected: true, message: data }));
        }
    });

**Empfangen:** Wenn der Server eine Nachricht vom Client bekommt, wird diese an den Port geschrieben.

    ws.on("message", function message(data) {
            if (port) port.write(data + "\n");
    });

### Client

Auf Clientseite nutzen wir das JS-native WebSocket Objekt

    let socket = new WebSocket('ws://localhost:8080')

**Empfangen:**

    socket.addEventListener('message', (event) => {
        const data = JSON.parse(event.data);
        //do something with the data
    }

**Senden:**

    socket.send(light);
