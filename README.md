<!--

author:   Konstantin Kirchheim

email:    konstantin.kirchheim@ovgu.de

version:  1.0.0

language: de

narrator:  Deutsch Female



@run_main
<script>
events.register("@0", e => {
		if (!e.exit)
    		send.lia("output", e.stdout);
		else
    		send.lia("eval",  "LIA: stop");
});

send.handle("input", (e) => {send.service("@0",  {input: e})});
send.handle("stop",  (e) => {send.service("@0",  {stop: ""})});


send.service("@0", {start: "CodeRunner", settings: null})
.receive("ok", e => {
		send.lia("output", e.message);

		send.service("@0", {files: {"main.c": `@input`}})
		.receive("ok", e => {
				send.lia("output", e.message);

				send.service("@0",  {compile: "gcc main.c -o a.out", order: ["main.c"]})
				.receive("ok", e => {
						send.lia("log", e.message, e.details, true);

						send.service("@0",  {execute: "./a.out"})
						.receive("ok", e => {
								send.lia("output", e.message);
								send.lia("eval", "LIA: terminal", [], false);
						})
						.receive("error", e => { send.lia("log", e.message, e.details, false); send.lia("eval", "LIA: stop"); });
				})
				.receive("error", e => { send.lia("log", e.message, e.details, false); send.lia("eval", "LIA: stop"); });
		})
		.receive("error", e => { send.lia("output", e.message); send.lia("eval", "LIA: stop"); });
})
.receive("error", e => { send.lia("output", e.message); send.lia("eval", "LIA: stop"); });

"LIA: wait";
</script>

@end







@sketch
<script>
events.register("mc_stdout", e => { send.lia("output", e); });
events.register("mc_start", e => { send.lia("eval", "LIA: terminal"); });


let compile = `arduino-builder -compile -logger=machine -hardware /usr/local/share/arduino/hardware -tools /usr/local/share/arduino/tools-builder -tools /usr/local/share/arduino/hardware/tools/avr -built-in-libraries /usr/local/share/arduino/libraries -libraries /usr/local/share/arduino/libraries -fqbn="Robubot Micro:avr:microbot" -ide-version=10807 -build-path $PWD/build -warnings=none -prefs=build.warn_data_percentage=100 -prefs=runtime.tools.avr-gcc.path=/usr/local/share/arduino/packages/arduino/tools/avr-gcc/4.8.1-arduino5/avr -prefs=runtime.tools.avr-gcc-4.8.1-arduino5.path=/usr/local/share/arduino/packages/arduino/tools/avr-gcc/4.8.1-arduino5/avr sketch/sketch.ino`;


send.service("@0arduino", {start: "CodeRunner", settings: null})
.receive("ok", e => {

		send.lia("output", e.message);
    send.service("@0arduino", {files: {"sketch/sketch.ino": `@input(0)`,
                                       "sketch/Distance.h": `@input(1)`,
                                       "sketch/Distance.cpp": `@input(2)`,
                                       "sketch/Display.h": `@input(3)`,
                                       "sketch/Display.cpp": `@input(4)`,
                                       "sketch/everytime.h": `@input(5)`,
                                       "sketch/IMU.h": `@input(6)`,
                                       "sketch/IMU.cpp": `@input(7)`,
                                       "sketch/IMURegisters.h": `@input(8)`,
                                       "sketch/Motor.h": `@input(9)`,
                                       "sketch/Motor.cpp": `@input(10)`,
                                       "build/": ""}})
		.receive("ok", e => {

				send.lia("output", e.message);
				send.service("@0arduino",  {compile: compile, order: ["sketch.ino", "Distance.h", "Distance.cpp", "Display.h", "Display.cpp", "everytime.h", "IMU.h", "IMU.cpp", "IMURegisters.h", "Motor.h", "Motor.cpp"]})
				.receive("ok", e => {

						send.lia("log", e.message, e.details, true);
            if(!window["bot_selected"]) { send.lia("eval", "LIA: stop"); }
            else {

              document.getElementById("button_"+window.bot_selected).disabled = true;
              send.service("c",
                { connect: [["@0arduino", {"get_path": "build/sketch.ino.hex"}], ["mc", {"upload": null, "target": window["bot_selected"]}]]
                }
              );

              send.handle("input", (e) => {
                send.service("mc",  {id: "bot_stdin",
                           action: "call",
                           params: {procedure: "com.robulab.target."+window["bot_selected"]+".send_input", args: [0,  String.fromCharCode(0)+btoa(e) ] }})});

              send.handle("stop",  (e) => {
                document.getElementById("button_"+window.bot_selected).disabled = false;

                send.service("mc", {id: "stdio0",
                                   action: "unsubscribe",
                                   params: {id: window["stdio0"], args: [] }});

                send.service("mc", {id: "stdio1",
                                   action: "unsubscribe",
                                   params: {id: window["stdio1"], args: [] }});

                send.service("mc",  {id: "bot_disconnect."+window["bot_selected"],
                           action: "call",
                           params: {procedure: "com.robulab.target.disconnect", args: [window["bot_selected"]] }})});


                delete window.stdio0;
                delete window.stdio1;
            }
				})
				.receive("error", e => { send.lia("log", e.message, e.details, false); send.lia("eval", "LIA: stop"); });
		})
		.receive("error", e => { send.lia("output", e.message); send.lia("eval", "LIA: stop"); });
})
.receive("error", e => { send.lia("output", e.message); send.lia("eval", "LIA: stop"); });

"LIA: wait";
</script>

<script>
function aduinoview_init() {

  let arduino_view_frame = document.getElementById("arduinoviewer");

  window["arduino_view_frame"] = arduino_view_frame;

  arduino_view_frame.onload = function () {

    arduino_view_frame.contentWindow.ArduinoView.init = function () {

      arduino_view_frame.contentWindow.ArduinoView.sendMessage = function(e) {

        send.service("mc",  {id: "bot_stdin2",
                   action: "call",
                   params: {procedure: "com.robulab.target."+window["bot_selected"]+".send_input", args: [1,  String.fromCharCode(0)+btoa(e) ] }});
      };

      arduino_view_frame.contentWindow.ArduinoView.onInputPermissionChanged(true);
    };
  };

  arduino_view_frame.contentWindow.ArduinoView.sendMessage = function(e) {

    send.service("mc",  {id: "bot_stdin2",
               action: "call",
               params: {procedure: "com.robulab.target."+window["bot_selected"]+".send_input", args: [1,  String.fromCharCode(0)+btoa(e) ] }});
  };

  arduino_view_frame.contentWindow.ArduinoView.onInputPermissionChanged(true);

}

function update() {
  if(!window["bot_selected"]) {
    for(let i=0; i<window.bot_list.length; i++) {
      let btn = document.getElementById("button_"+window.bot_list[i].target);

      if(btn === null) {
        let cmdi = document.getElementById("bot_list");

        btn = document.createElement("BUTTON");
        btn.id = "button_" + window.bot_list[i].target;
        btn.innerHTML = window.bot_list[i].name;
        btn.onclick = function() {
           let id = window.bot_list[i].target;

           send.service("mc", {id: "bot_connect."+id,
                      action: "call",
                      params: {procedure: "com.robulab.target.connect", args: [id] }});
         };

         cmdi.appendChild(btn);
         cmdi.appendChild(document.createTextNode(" "));
      }

      btn.style.backgroundColor = "";

      if(window.bot_list[i].owner == ""){
        btn.className = "lia-btn";
        btn.disabled = false;
      }
      else {
        btn.className = "lia-btn";
        btn.disabled = true;
      }
    }
  }
  else {
    for(let i=0; i<window.bot_list.length; i++) {
      let btn = document.getElementById("button_"+window.bot_list[i].target);
      if(window.bot_list[i].target == window["bot_selected"]){
        btn.className = "lia-btn";
        btn.disabled = false;
        btn.style.backgroundColor = "#ADFF2F";
      }
      else {
        btn.className = "lia-btn";
        btn.disabled = true;
      }
    }
  }
}


function subscriptions() {
    events.register("mc", e => {
       if(typeof(e) === "undefined")
          return;

       if(!!e.subscription) {
         if (e.subscription.startsWith("com.robulab.livestream")) {
            //console.log("vid", e.parameters.args[0].data);
            window.cam.PutData(new Uint8Array(e.parameters.args[0].data));
         }
         else if (e.subscription == "com.robulab.target.changed") {
           let args = e.parameters.args;
           for(let i=0; i<args.length; i++) {
             for(let j=0; j<window.bot_list.length; j++) {
               if(args[i].target == window.bot_list[j].target) {
                 window.bot_list[j] = Object.assign(window.bot_list[j], args[i])
               }
             }
           }
           update();
         } else if (e.subscription.endsWith(".0.raw_out")) {
           events.dispatch("mc_stdout", String.fromCharCode.apply(this, e.parameters.args[0].data));
         } else if (e.subscription.endsWith(".1.raw_out")) {
           window["arduino_view_frame"].contentWindow.ArduinoView.onArduinoViewMessage(
             String.fromCharCode.apply(this, e.parameters.args[0].data) );
         }
         else {
          //console.log("problem", e);
         }
       }
       else if(e.id == "bot_list") {
         window["bot_list"] = e.ok;
         let cmdi = document.getElementById("bot_list");

         for (let i = 0; i< window.bot_list.length; i++) {
            let btn = document.createElement("BUTTON");
            btn.id = "button_" + window.bot_list[i].target;
            btn.innerHTML = window.bot_list[i].target;
            btn.onclick = function() {
              let id = window.bot_list[i].target;

              send.service("mc", {id: "bot_connect."+id,
                           action: "call",
                          params: {procedure: "com.robulab.target.connect", args: [id] }});
              };

              cmdi.appendChild(btn);
              cmdi.appendChild(document.createTextNode(" "));

              send.service("mc", {id: "bot_name."+i,
                           action: "call",
                           params: {procedure: "com.robulab.target."+ window.bot_list[i].target +".get_name", args: [] }});
             }
           update();
         }
         else if( e.id.startsWith("bot_flash1") ) {
           let [cmd, target, file] = e.id.split(" ");
           send.service("mc", {upload: file, target: target, id: e.ok});
         }
         else if( e.id.startsWith("bot_flash2") ) {
           let [cmd, target, id] = e.id.split(" ");
           send.service("mc", {finish: parseInt(id), target: target});
         }
         else if ( e.id.startsWith("bot_flash3") && !e.ok ) {
           aduinoview_init();
           let [cmd, target, id] = e.id.split(" ");

           send.service("mc", {id: "bot_stdio.0.target",
                               action: "subscribe",
                               params: {topic: "com.robulab.target."+target+".0.raw_out", args: [] }});

           send.service("mc", {id: "bot_stdio.1.target",
                               action: "subscribe",
                               params: {topic: "com.robulab.target."+target+".1.raw_out", args: [] }});

           events.dispatch("mc_start", "");


        }

        else if (e.id.startsWith("bot_stdio")) {
          let [cmd, stdio, target] = e.id.split(".");
          window["stdio"+stdio] = e.ok.id;
        }

        else if( e.id.startsWith("bot_name") ) {
          let [cmd, id] = e.id.split(".")
          window.bot_list[id]["name"] = e.ok;
          document.getElementById("button_"+window.bot_list[id].target).innerHTML = e.ok;
        }

        else if( e.id.startsWith("bot_connect") ) {
          let [cmd, target] = e.id.split(".");
          window["bot_selected"] = target;
          let btn = document.getElementById("button_"+target);
          btn.onclick = function() {
              send.service("mc", {id: "bot_disconnect."+target,
                         action: "call",
                         params: {procedure: "com.robulab.target.disconnect", args: [target] }});
          };

          send.service("mc", {id: "bot_stream."+target,
                     action: "call",
                     params: {procedure: "com.robulab.target."+target+".get_stream", args: [target] }});

          update();

          window["cam"] = window.decoder(document.getElementById("bot_show"), ([w, h]) => {
              let canvas = document.getElementById("bot_show");

              canvas.height = h;
              canvas.width = w;
          });
        }
        else if( e.id.startsWith("bot_disconnect") ) {
          let [cmd, target] = e.id.split(".");
          delete window["bot_selected"];

          let btn = document.getElementById("button_"+target)
          btn.onclick = function() {
              send.service("mc", {id: "bot_connect."+target,
                         action: "call",
                         params: {procedure: "com.robulab.target.connect", args: [target] }});
          };

          send.service("mc", {id: "cam_disconnect",
                             action: "unsubscribe",
                             params: {id: window["streaming_id"], args: [] }});

          update();
        }
        else if ( e.id.startsWith("bot_stream") ) {
          let [cmd, target] = e.id.split(".");
          send.service("mc",
                       {id: "streaming",
                        action: "subscribe",
                        params: {topic: e.ok.url, args: [] }});
        }
        else if (e.id == "streaming") {
          window["streaming_id"] = e.ok.id;
        }

        else if ( e.id == "cam_disconnect") {
          delete window.cam;
        }

        else {
          console.log("not handled", e);
      }
      });

      send.service("mc", {id: "bot_list",
                            action: "call",
                            params: {procedure: "com.robulab.target.get-online", args: [] }});

      send.service("mc", {id: "bot_changes",
                            action: "subscribe",
                            params: {topic: "com.robulab.target.changed", args: [] }})

      send.service("mc", {id: "bot_changes",
                            action: "subscribe",
                            params: {topic: "com.robulab.target.reregister", args: [] }})


      window["mc_subscribed"] = true;

};

function login(silent=true) {
    let cmdi = document.getElementById("mcInterface");

    send.service("mc", {start: "MissionControl", settings: null})
      .receive("ok", (e) => {
          console.log("user-connected:", e);
          cmdi.hidden = false;
          window["mc_logged_in"] = true;
          subscriptions();
      })
      .receive("error", (e) => {
          cmdi.hidden = true;
          console.log("Error user-connected:", e);
          alert("Fail: Please check your login!");
      });
};

if (!window.mc_logged_in) {
  setTimeout((e) => { login() }, 300);
}
else {
  document.getElementById("mcInterface").hidden = false;
  update();
}

window.addEventListener("beforeunload", function (event) {

    if ( window.stdio1 ) {
          send.service("mc", {id: "stdio1",
                       action: "unsubscribe",
                       params: {id: window["stdio1"], args: [] }});
    }

    if ( window.stdio0 ) {
          send.service("mc", {id: "stdio0",
                       action: "unsubscribe",
                       params: {id: window["stdio0"], args: [] }});
    }

    if(window.bot_selected) {
        send.service("mc", {id: "cam_disconnect",
                           action: "unsubscribe",
                           params: {id: window["streaming_id"], args: [] }});

        send.service("mc",  {id: "bot_disconnect."+window["bot_selected"],
                   action: "call",
                   params: {procedure: "com.robulab.target.disconnect", args: [window["bot_selected"]] }});
    }

});

</script>


<div id="mcInterface" hidden="true">
  <span style="border-style: solid; width: 49.5%; float: left; min-width: 480px;">
    <span id="bot_list" ></span>
    <br>
    <iframe id="arduinoviewer" style="margin-left: 3px; width: 99%; max-height: 432px;" src="https://elab.ovgu.robulab.com/arduinoview"></iframe>
  </span>
  <span style="border-style: solid; width: 49.5%; height: 480px; float: right; min-width: 480px; overflow: auto">
    <canvas id="bot_show" style="width: calc(16 * 10vw); height: calc(9 * 10vw);"></canvas>
  </span>
</div>
@end


@init_clear
<script>
  let cmdi = document.getElementById("mcInterface");
  if(cmdi)
    cmdi.hidden = true;

  let viewer = document.getElementById("arduinoviewer");
  if(viewer)
    try {
    //  viewer.contentWindow.document.body.innerHTML = ""
    } catch(e) {}

  if ( window.stdio1 ) {
          send.service("mc", {id: "stdio1",
                       action: "unsubscribe",
                       params: {id: window["stdio1"], args: [] }});
  }

  if ( window.stdio0 ) {
          send.service("mc", {id: "stdio0",
                       action: "unsubscribe",
                       params: {id: window["stdio0"], args: [] }});
  }

  if(window.bot_selected) {
        send.service("mc", {id: "cam_disconnect",
                           action: "unsubscribe",
                           params: {id: window["streaming_id"], args: [] }});

        send.service("mc",  {id: "bot_disconnect."+window["bot_selected"],
                   action: "call",
                   params: {procedure: "com.robulab.target.disconnect", args: [window["bot_selected"]] }});
  }
</script>


@end




-->

# Einführung

@init_clear

Willkommen zurück im eLearning-System *eLab*.

    --{{0}}--
Während der Fokus der vorhergehenden Aufgaben auf den aktorischen
Hardwarekomponenten unserer Roboterplattform lag, wollen wir in dieser Aufgabe,
die ersten Sensoren in Betrieb nehmen. Mit ihnen wird es uns möglich sein,
bestimmte Eigenschaften der Umgebung unseres Roboters zu überwachen. Die
erhaltenen Umgebungsinformationen können wir in der nächsten Aufgabe nutzen, um
eine intelligente Bewegungsstrategie umzusetzen.

     {{0-2}}
![bender](https://media2.giphy.com/media/28gR4lnnoJVaU/giphy.gif)<!-- style="width: 80%; max-width: 600px" -->


    --{{1}}--
Bei den Sensoren, die wir in dieser Aufgabe ansteuern und auslesen werden,
handelt es sich um zwei Infrarot-Distanzsensoren, sowie eine intertiale
Messeinheit (engl. _inertial measurement unit (IMU)_).

    --{{2}}--
Mit Hilfe der Distanzsensoren können wir den Nah- und Fernbereich der
Vorderseite unseres Roboters überwachen. Diese Informationen sind zum Beispiel
zur Implementierung einer Kollisionsvermeidung hilfreich.

      {{2}}
![ir](https://upload.wikimedia.org/wikipedia/commons/2/27/Sharp_GP2Y0A21YK_IR_proximity_sensor_cropped.jpg)<!-- style="width: 200px" -->

    --{{3}}--
Mit den Informationen der IMU, die sich aus einem 3-Achsen-Gyroskop,
3-Achsen-Beschleunigungssensor, 3-Achsen-Kompass und einem Temperatursensor
zusammensetzt, kann eine Überwachung der Positions- und Orientierungsänderungen
unseres Roboters umgesetzt werden. Wir können also, mit einer gewissen
Unsicherheit, diese Informationen nutzen um die aktuelle Position des Roboters
relativ zu seiner intertialen Position zu bestimmen.

      {{3}}
![imu](https://upload.wikimedia.org/wikipedia/commons/5/54/Flight_dynamics_with_text.png)<!-- style="width: 200px" -->


    --{{4}}--
Bevor ihr diese *high-level* Informationen jedoch generieren könnt,
implementiert ihr in dieser Aufgabe zunächst die Funktionalität zum Auslesen der
Sensorwerte.

## Themen und Ziele

@init_clear

    --{{0}}--
Entsprechend der bereits angesprochenen Sensorik (IR-Distanzsensoren, IMU)
bilden deren Messverfahren und speziellen Ausführungen ein Kernthema dieser
Aufgabe. Allerdings unterscheidet sich je nach Sensorik, dessen Ausführung und
Anbindung an den Mikrocontroller, das Vorgehen zum Auslesen aktueller
Sensorwerte.

    --{{1}}--
Im Allgemeinen geben Sensoren zunächst analoge Spannungswerte aus, die mit dem
Wert der gemessenen Umgebungsgröße funktional korrelieren. Diese müssen mit
Hilfe eines Analog-Digital-Wandlers *(engl. analog-digital converter (ADC))*
zunächst in digitale Werte umgesetzt werden. In Abhängigkeit von der konkreten
Ausführung des Sensors, kann dies bereits innerhalb des Sensors geschehen, oder
muss durch einen externen ADC implementiert werden.

    --{{2}}--
Anschließend an die Digitalisierung der Spannungswerte müssen diese noch
entsprechend der Sensorkennlinie in die eigentlich zu messende Umgebungsgröße
umgerechnet werden.

**Themen:**

    {{0}}
* Infrarot Distanzsensoren
* Inertiale Messeinheit (engl. *inertial measurement unit (IMU)*)
* {{1}} Analog-Digital-Converter (ADC)
* {{2}} Interpretation von Spannungswerten anhand von Kennlinien


    --{{3}}--
Mit Hilfe dieses Vorgehens und der angesprochenen Sensorik, ist das Ziel dieser
Aufgabe, Entfernungen vor dem Roboter, sowie dessen Positions- und
Orientierungsänderungen zu messen. Ausgewählte Werte sollen durch die
8-Segment-Anzeigen ausgegeben werden. Letztlich sollen die Informationen so
verarbeitet werden, dass ein Positionstracking des Roboters möglich ist.

                               {{3}}
*******************************************************************************

**Ziel(e):**

* Messen von Entfernungen durch Infrarot-Sensorik
* Messen von Beschleunigungen und Orientierungsänderungen
* Interpretation der Sensorwerte über die Zeit (Geschwindigkeit, Orientierung)

*******************************************************************************

## Weitere Informationen

@init_clear

                             --{{0}}--
Wie schon durch die Vorlesung bekannt, sind die physikalischen Prinzipien der
einzelnen Sensoren interessant für diese Aufgabe.


                               {{0}}
*******************************************************************************

**Sensoren:**

* [Entfernungssensor](https://en.wikipedia.org/wiki/Proximity_detector)
* [Gyroskope](http://sensorwiki.org/doku.php/sensors/gyroscope)
* [Beschleunigungssensoren](http://www.sensorwiki.org/doku.php/sensors/accelerometer)
* [Magnetometer](https://en.wikipedia.org/wiki/Magnetometer)

*******************************************************************************

                             --{{1}}--
Darüber hinaus sind auch die verschiedenen Varianten von Analog-Digital-Wandler
von Interesse. Informiert euch auch darüber, welche Varianten uns der
[AVR ATmega32U4](http://www.atmel.com/Images/Atmel-7766-8-bit-AVR-ATmega16U4-32U4_Datasheet.pdf)
bietet.

                               {{1}}
*******************************************************************************

**ADC:**

* [Analog-Digital-Wandler](https://en.wikipedia.org/wiki/Analog-to-digital_converter)
* Allgemeine Erklärungen zur Analog-Digital-Wandler:

  !?[ADC](https://www.youtube.com/embed/_gZjBx9cdro)<!--
    width: 560px;
    height: 315px;
  -->

*******************************************************************************



                             --{{2}}--
Neben der von uns hauptsächlich genutzten Anbindung peripherer
Hardwarkomponenten an die Mikrocontroller, existieren noch leistungsfähigere
Bus-Systeme. Hier seien Beispielhaft
[I2C](https://en.wikipedia.org/wiki/I%C2%B2C) und
[SPI](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus) genannt.
Interessant ist vor allem der *I^2^C*-Bus, da über eine solche Variante die IMU
an unseren Mikrocontroller angeschlossen ist.
**Hinweis: im Datenblatt des Mikrocontrollers wird hier auch von *TWI* gesprochen!**


                               {{2}}
*******************************************************************************

**Serielle Bus-Protokolle:**

* [I2C](https://en.wikipedia.org/wiki/I%C2%B2C)
* [SPI](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus)

**Arduino-Projekt:**

* Ein Arduino-Projekt mit einem Sharp Distanzsensor:

  !?[ArduinoSharp](https://www.youtube.com/embed/DTUUKM--PyQ)<!--
    width: 560px;
    height: 315px;
  -->

*******************************************************************************


                                --{{3}}--
Letztlich findet ihr wie immer die Datenblätter und Dokumentation für diese
Aufgabe unter dem Punkt **PKeS**.


                                  {{3}}
*******************************************************************************

**PKeS:**

* [Datenblatt IMU (MPU-9255)](https://stanford.edu/class/ee267/misc/MPU-9255-Datasheet.pdf)
* [Datenblatt IMU (MPU-9255) Register Map](https://stanford.edu/class/ee267/misc/MPU-9255-Register-Map.pdf)
* [Datenblatt infrarot Distanzsensor Sharp GP2Y0A41SK0F](http://sharp-world.com/products/device/lineup/data/pdf/datasheet/gp2y0a41sk_e.pdf)
* [Datenblatt infrarot Distanzsensor Sharp GP2Y0A02YK0F](http://sharp-world.com/products/device/lineup/data/pdf/datasheet/gp2y0a02yk_e.pdf)
* [Datenblatt Motortreiber](http://www.st.com/content/ccc/resource/technical/document/datasheet/59/d0/ce/41/56/bb/4b/10/DM00034699.pdf/files/DM00034699.pdf/jcr:content/translations/en.DM00034699.pdf)
* [Schaltbelegungsplan](https://github.com/liaScript/PKeS0/blob/master/materials/robubot_stud.pdf?raw=true)
* [Arduinoview](https://github.com/fesselk/Arduinoview/blob/master/doc/Documetation.md)
* [Datenblatt des AVR ATmega32U4](http://www.atmel.com/Images/Atmel-7766-8-bit-AVR-ATmega16U4-32U4_Datasheet.pdf)

*******************************************************************************


# Aufgabe 3

@init_clear

    --{{0}}--
Zu Beginn dieser Aufgabe müssen die von den Sensoren gelieferten Werte zunächst
ausgelesen und in den Mikrocontroller übertragen werden.

    --{{1}}--
Wie bereits angedeutet, liefern einige Sensoren lediglich analoge Spannungswerte
als Repräsentation der gemessenen Umgebungsinformation. Dies trifft auf die
beiden Sharp-Distanzsensoren zu. Daher müssen in einem ersten Schritt die
entsprechenden Analog-Digital-Wandler (einer für jeden Sensor) konfiguriert
werden. So erhaltet ihr die digitalisierten Informationen der Spannungswerte.
Diese müssen dann für jeden der beiden Distanzsensoren individuell in
Entfernungswerte konvertiert werden.

    --{{2}}--
Im Gegensatz zu den Distanzssensoren überliefert die IMU bereits digitale
linearisierte Repräsentationen der Sensorwerte. Allerdings ist die IMU über
einen I^2^C-Bus angeschlossen, den ihr nutzen müsst, um die IMU entsprechend zu
konfigurieren und aktuelle Sensorwerte auszulesen. Dies soll in der zweiten
Teilaufgabe implementiert werden. Ähnlich zu den Distanzsensoren müssen auch
hier die von dem Sensor gelieferten Werte konvertiert werden.

    --{{3}}--
Letztlich sollen die gemessenen Sensorwerte nun noch durch das
Arduinoview-Interface, wie auch durch die 8-Segment-Anzeigen, dargestellt
werden. Diese Darstellung sollt ihr in der letzten Teilaufgabe implementieren.
Darüber hinaus sollen die Informationen der IMU genutzt werden, um die
Orientierung des Roboters zu verfolgen und seine aktuelle Geschwindigkeit
anzuzeigen.

**Teilaufgaben**

* {{1}} Auslesen der Distanzsensoren und Konvertieren der Sensorwerte.
* {{2}} Auslesen der IMU und Konvertieren der Sensorwerte.
* {{3}} Visualisierung der Sensorwerte, Anzeige auf den 8-Segment-Anzeigen und
        Implementierung einer Orientierungs- und Geschwindigkeitsüberwachung.
* {{4}} **Bonus:** Berechnen der Roboterposition relativ zur Startposition.


    --{{4}}--
Im Gegensatz zu den bisherigen Aufgaben greifen wir diesesmal auf eure
bisherigen Implementierungen zurück. Dies betrifft die Ansteuerung der
8-Segment-Anzeigen, sowie der Motoren. Während die 8-Segment-Anzeige direkt zur
Bearbeitung der Aufgabe notwendig ist, könnt ihr die Motoren und den zur
Ansteuerung bekannten *Joystick* nutzen, um den Roboter zu bewegen und so eure
Sensorik zu testen.

    --{{5}}--
Damit ihr eure Lösungen aus den vorherigen Aufgaben weiter nutzen könnt, müsst
ihr sie aus den entsprechenden Projekten kopieren. Wie ihr sicherlich bemerkt
habt, befindet sich das eLab noch in einem jungen Stadium, sodass wir noch keine
Gelegenheit hatten, einen automatisierten Austausch des Quellcodes zwischen
Projekten/Aufgaben zu realisieren. Wir bitten das zu entschuldigen.

    --{{6}}--
In den Vorlagen zu dieser Aufgabe findet ihr außerdem eine zusätzliche
Bibliothek: `everytime`, mit ihr könnt ihr Programmteile periodisch ausführen
(zum Beispiel alle 300ms), ohne dazu die Programmausführung unterbrechen zu
müssen, wie es mit `delay` der Fall wäre.

    {{6}}
> **Hinweis:**
>
> Verwendet bei der Bearbeitung der Aufgabe keine Funktionen aus der
> Arduino-Bibliothek. Lediglich die Funktionen der `Serial`-Klasse zur
> Ansteuerung der seriellen Schnittstelle, der Funktion `millis()` zur einfachen
> Zeitmessung, sowie der Funktionen aus `Wire.h` für die I^2^C Kommunikation
> können genutzt werden.

## Aufgabe 3.1

@init_clear

    --{{0}}--
Das Ziel dieser Aufgabe ist es, die Distanzsensoren an der Vorderseite unserer
Roboterplattform in Betrieb zu nehmen. Implementiert dazu die Funktionen in
`Distance.{h,cpp}`.


**Ziel:**
Implementiert das Auslesen der Sensorwerte beider Distanzsensoren und konvertiert die Spannungswerte in Distanzwerte in Millimeter (mm).

**Teilschritte:**

    --{{1}}--
Der erste Schritt dazu ist, den entsprechenden Analog-Digital-Konverter für die
Verwendung mit den Distanzsensoren zu konfigurieren. Dies soll in der Funktion
`configADC` geschehen.

    --{{2}}--
Damit das Auslesen der Sensorwerte zyklisch geschehen kann, implementiert ihr im
nächsten Schritt die Funktion `readADC`, die den aktuellen, digitalisierten
Spannungswert des ausgewählten Sensors in mV liefern soll.

    --{{3}}--
Im letzten Schritt muss der digitalisierte Spannungswert entsprechend der
Kennlinie des jeweiligen Distanzsensors in Milimeter konvertiert werden.
Implementiert dazu die Funktionen `linearizeLR` und `linearizeSR`, wobei *LR*
für *Long Range* und *SR* für *Short Range* steht. Ihr könnt dazu die
entsprechenden Kennlinien aus den Datenblättern der Sensoren nutzen. Beachtet,
dass wir die Distanzwerte in Millimeter (mm) darstellen wollen!


* {{1}} Konfiguriert Analog-Digital-Wandler für die Verwendung mit den
        Distanzsensoren.
* {{2}} Implementiert Funktion `readADC(uint8_t channel)` zum Auslesen der
        Distanzsensoren (digitalisierte Spannungswerte in mV).
* {{3}} Implementiert Funktionen `linearizeLR(uint16_t voltage)` und
        `linearizeSR(uint16_t voltage)` zur Transformation der Spannungswerte in
        Distanzwerte (mm) für beide Sensoren.

## Aufgabe 3.2

@init_clear

    --{{0}}--
Nachdem wir nun die Ansteuerung der Distanzsensoren erfolgreich implementiert
haben, wollen wir auch die Daten der inertialen Messeinheit auslesen und
konvertieren.

    --{{1}}--
Die IMU ist, im Gegensatz zu den Distanzsensoren, per I^2^C an den
Mikrocontroller angeschlossen. Daher haben wir euch eine Implementierung zur
Kommunikation mit der IMU vorgegeben. Um diese zu Initialisieren, müsst ihr
zunächst `initializeIMU` aufrufen. Anschließend könnt ihr mit Hilfe der
I^2^C-Kommunikation die aktuellen Sensorwerte aus der IMU auslesen. Dort sind
vorallem die Beschleunigungen und Drehraten für uns von interesse. Daher füllt
die Funktion `readIMU` die Struktur `IMUdata`.

    --{{2}}--
Wie schon bei den Distanzsensoren erhaltet ihr von der IMU lediglich Werte die
proportional zu den Beschleunigungen bzw. Drehraten sind. Daher müsst ihr auch
diese Werte in $\frac{m}{s^2}$ bzw. $\frac{rad}{s}$ konvertieren. Implementiert
dazu die Funktion `readIMUscaled`.

**Ziel:**

Implementiert das Auslesen und Transformieren der Beschleunigungs- und
Drehratenwerte der IMU. Beschleunigungen sollen in $\frac{m}{s^2}$ und Drehraten
in $\frac{rad}{s}$ repräsentiert werden.

**Teilschritte:**

Implementiert das Auslesen und Transformieren der Beschleunigungs- und
Drehratenwerte der IMU.

## Aufgabe 3.3

@init_clear

    --{{0}}--
Da wir nun die Umgebung unseres Roboters mit Hilfe unserer Sensoren beobachten
können, wollen wir die erhaltenen Werte natürlich noch entsprechend darstellen.
Daher besteht das Ziel dieser Aufgabe darin, die Sensorwerte mit Hilfe von
Arduinoview zu visualisieren und ein Orientierungs- und
Geschwindigkeitsüberwachung zu implementieren.

**Ziel:** Visualisiert die Sensordaten mit Diagrammen in Arduinoview und zeigt
die Geschwindigkeit ($v~in~[\frac{mm}{s}]$) und Orientierung ($\theta~in~[rad]$)
des Roboters relativ zu seiner Startposition an.


> **Hinweis:**
>
> Da durch die Anzahl der Diagramme die Darstellung in Arduinoview
> unübersichtlich werden kann, könnt ihr jeweils die Werte auch in textueller
> Form darstellen und periodisch aktualisieren.


    --{{1}}--
Im ersten Schritt solltet ihr eurem Arduinoview ein Diagramm hinzufügen, in dem
ihr die aktuellen Werte der Distanzsensoren visualisiert.

    --{{2}}--
Äquivalent zum vorhergehen Schritt könnt ihr nun ein weiteres Diagramm dem
Arduinoview-Interface hinzufügen, in dem ihr die Beschleunigungswerte der
3-Achsen der IMU darstellt.

    --{{3}}--
Da uns die IMU auch Drehraten für die 3 Achsen bereitstellt, sollen auch diese
Werte in einem separaten Diagramm dargestellt werden.

    --{{4}}--
Da uns durch die Aufgabe 1 nun auch die 8-Segment-Anzeigen zur Darstellung von
Zeichen zur Verfügung steht, wollen wir diese natürlich auch nutzen. Gebt daher
jeweils den kleinsten der beiden Distanzwerte auf der Anzeige aus. Falls
lediglich ein Distanzsensor valide Werte liefert, gebt ihr diesen aus. Falls
beide Distanzsensoren keine validen Werte liefern, gebt ein '---' auf dem
Display aus.

    --{{5}}--
Zuletzt möchten wir die Daten der IMU zur Überwachung der Orientierung und
Geschwindigkeit des Roboters nutzen. Lest dazu sowohl die Beschleunigungs- als
auch die Drehraten der IMU über die gesamte Ausführungszeit aus und konvertiert
sie so, dass ihr eine Geschwindigkeits- und Orientierungsangabe relativ zu der
Startposition des Roboters erhaltet. Gebt letztlich die Geschwindigkeit
($v~in~[\frac{mm}{s}]$) und Orientierung ($\theta~in~[rad]$) über das
Arduinoview-Interface kontinuierlich aus.


**Teilschritte:**

* {{1}} Stellt die aktuellen Werte beider Distanzsensoren in einem Diagramm dar.
* {{2}} Stellt die aktuellen Beschleunigungswerte aller 3 Achsen in einem
        Diagramm dar.
* {{3}} Stellt die aktuellen Drehraten aller 3 Achsen in einem Diagramm dar.
* {{4}}
  *****************************************************************************
  Messwertdarstellung auf dem Display
  * über Knöpfe auf dem Roboter soll sich Messwerte direkt anzeigen lassen
    (_Knopf 1_: LR Distanz, _Knopf 2_: SR Distanz, _Knopf 3_: Drehwinkelwinkel
    des Roboters um die Senkrechte, _Knopf 4_: Rücksetzen des Drehwinkels)
  * wenn kein knopf gedrück wird soll einer der beiden Distanzwerte ausgegeben
    werden
  * bestimmt die Qualität der Messungen unterschiedlicher Distanzen
    (experimentel) und implemetiert eine Auswahl des Sensors
  *****************************************************************************
* {{5}} Konvertiert die Beschleunigungs- und Drehratenwerte in eine
        Robotergeschwindigkeit und -Orientierung, d.h. $v$ und $\theta$.

## Bonusaufgabe

@init_clear

    --{{0}}--
Neben der Geschwindigkeitsüberwachung erlauben uns die Beschleunigungswerte in
$x$ und $y$ Richtung, auch die Implementierung einer Positionsüberwachung. Daher
ist das Ziel dieser Bonusaufgabe, die Berechnung der Position ($x~in~[mm]$,
$y~in~[mm]$).

**Ziel:**
Berechnet die Roboterposition ($x$,$y$) mit Hilfe der IMU Daten.

    --{{1}}--
Wie stabil ist die Positionsangabe? Welche Gründe könnte die auftretende
Unsicherheit haben und wie könnten die Unsicherheit reduziert werden?

# **Programmierung**

``` cpp     sketch.ino
#include <FrameStream.h>
#include <Frameiterator.h>
#include <avr/io.h>
#include "IMU.h"

#define OUTPUT__BAUD_RATE 9600
FrameStream frm(Serial1);

// Forward declarations
void InitGUI();

// hierarchical runnerlist, that connects the gui elements with
// callback methods
declarerunnerlist(GUI);

// First level will be called by frm.run (if a frame is recived)
beginrunnerlist();
fwdrunner(!g, GUIrunnerlist); //forward !g to the second level (GUI)
callrunner(!!, InitGUI);
fwdrunner(ST, stickdata);
endrunnerlist();

// GUI level
beginrunnerlist(GUI);
callrunner(es, CallbackSTOP);
callrunner(ms, CallbackSTART);
endrunnerlist();

int8_t left;
int8_t right;

void stickdata(char* str, size_t length)
{
    // str contains a string of two numbers ranging from -127 to +127
    // indicating left and right motor values
    // TODO: interprete str, set motors
}

void CallbackSTOP()
{
    // call deactivateMotors();
}


void CallbackSTART()
{
    // call activateMotors
}

/*
 * @brief initialises the GUI of ArduinoView
 *
 * In this function, the GUI, is configured. For this, Arduinoview shorthand,
 * HTML as well as embedded JavaScript can be used.
 */
void InitGUI()
{
    delay(500);

    frm.print(F("!h<h1>PKeS Exercise 2</h1>"));
    frm.end();

    frm.print(F("!SbesvSTOP"));
    frm.end();

    frm.print(F("!SbmsvSTART"));
    frm.end();
    //this implements a joystick field using HTML SVG and JS
    frm.print(F("!H"
      "<div><style>svg * { pointer-events: none; }</style>\n"
      "<div style='display:inline-block'> <div id='state'> </div>\n"
      "<svg id='stick' width='300' height='300' viewBox='-150 -150 300 300' style='background:rgb(200,200,255)' >\n"
      "<line id='pxy' x1='0' y1='0' x2='100' y2='100' style='stroke:rgb(255,0,0);stroke-width:3' />\n"
      "<circle id='cc' cx='0' cy='0' r='3'  style='stroke:rgb(0,0,0);stroke-width:3' />\n"
      "</svg></div><div style='display:inline-block'><div id='info0'></div><div id='info1'></div><div id='info2'></div></div></div>\n"
      "<script>\n"
        "var getEl=function(x){return document.getElementById(x)};\n"
        "var setEl=function(el,attr,val){(el).setAttributeNS(null,attr,val)};\n"
        "var stick=getEl('stick');\n"
        "function sticktransform(x,y){\n"
          "x = x-150;\n"
          "y = -(y-150);\n"
          "setstick(x,y);\n"
        "}\n"
        "function setstick(x,y){\n"
          "setpointer(x,y);\n"
          "l = Math.floor(Math.min(127,Math.max(-127,y + x/2)));\n"
          "r = Math.floor(Math.min(127,Math.max(-127,y - x/2)));\n"
          "setStateDisplay(x,y,l,r);\n"
          "sendframe(\"ST\"+l +\",\"+r);\n"
        "}\n"
        "function setStateDisplay(x,y,l,r){\n"
          "msg=getEl('state');\n"
          "msg.innerHTML= 'x= '+ x +' y= '+ y +' l= '+ l +' r= '+ r ;\n"
        "}\n"
        "function setpointer(x,y){\n"
          "pxy=getEl('pxy');\n"
          "setEl(pxy,'x2',x);\n"
          "setEl(pxy,'y2',-y);\n"
        "}\n"
        "stick.onmousemove=function(e){\n"
          "if( e.buttons == 1 ){\n"
            "sticktransform(e.offsetX,e.offsetY);\n"
        "}};\n"
        "stick.onmousedown=function(e){\n"
          "if( e.buttons == 1 ){\n"
            "sticktransform(e.offsetX,e.offsetY);\n"
        "}};\n"
        "stick.onmouseup=function(e){\n"
          "setstick(0,0);\n"
        "};\n"
        "stick.onmouseleave=function(e){\n"
          "setstick(0,0);\n"
        "};\n"
        "function sendframe(msg){\n"
          "ArduinoView.sendMessage(ArduinoView._createSerialFrame(msg))\n"
        "}\n"
        "setstick(0,0);\n"
      "</script>\n"));
    frm.end();
}

/*
 * @brief Initialisation
 *
 * Implement basic initialisation of the program.
 */
IMUdataf imuoffset;

void setup()
{
    delay(1000);

    //prepare Serial interfaces
    Serial.begin(OUTPUT__BAUD_RATE);
    Serial1.begin(OUTPUT__BAUD_RATE);

    Serial.println(F("Willkommen zur PKeS Übung"));

    //request reset of GUI
    frm.print("!!");
    frm.end();

    delay(500);

    // initialise Motors, 8-Segment-Display, ADCs and IMU
}

/*
 *  @brief Main loop
 *
 *  This function will be called repeatedly and shall therefore implement the
 *  main behavior of the program.
 */

void loop()
{
    // read & run ArduinoView Frames
    while(frm.run());

    /* everytime.h example
    every(500){
        frm.print(F("!jgetEl('info2').innerHTML='"));
        frm.print(millis());
        frm.print("';");
        frm.end();
    }
    */
}
```
``` h Distance.h
#ifndef ESSIRSENSOR_H
#define ESSIRSENSOR_H

#include <inttypes.h>

void     configADC();
uint16_t linearizeSR(uint16_t );
uint16_t linearizeLR(uint16_t );
int16_t readADC(uint8_t );

#endif
```
``` cpp Distance.cpp
#include "Distance.h"
#include <Arduino.h>
#include <avr/io.h>

uint16_t ADC_mV_Faktor;

/**
 * @brief Configures the ADC related to mode, clock speed, gain
 *        resolution and refrence voltage
 *
 *
 * chose the reference voltage carfully related to the output
 * range of the sensor.
 */
void configADC()
{
}

/**
 * @brief Starts a single conversion and receives a result return in mV
 */
int16_t readADC(uint8_t channel)
{
    int16_t mV;
    return mV;
}

/**
 * @brief Maps the digital voltage information on a distance
 *        in mm
 */
uint16_t linearizeSR(uint16_t distmV)
{
}

/**
 * @brief Maps the digital voltage information on a distance
 *        in mm
 */
uint16_t linearizeLR(uint16_t distmV)
{
}
```
``` h Display.h
#ifndef DISPLAY_H
#define DISPLAY_H

#include <inttypes.h>

//TODO: create a Display.cpp that implements this interface

// you may create as much helper functions inside your Display.cpp as you like
// ADVICE: all values are displayed as decimal (Base 10)
// if you like you may ADD functions that display values as hexadecimal (Base 16)

//initialises Display
//Setup Data Direction
//write some default pattern or empty to Display

void initDisplay();

// writes 3 data-bytes to Display
// these bytes/bits should represent the 7 Segments and Dot
// may be used to implement the pattern or clearing of initDisplay()
// and the writeValue Functions
// prepare to discuss your bitorder
void writeToDisplay(uint8_t data[3]);

//writes an integer value to display
void writeValueToDisplay(int value);

//writes float value to display
//display as many significant digits as possible
//eg. 5.87 ; 14.5; 124.; -12.4
void writeValueToDisplay(float value);

#endif
```
``` cpp Display.cpp
#include "Display.h"

// TODO: Write a Driver for the LED-Display
```
``` h  -everytime.h
#ifndef EVERYTIMEH24b0433
#define EVERYTIMEH24b0433
/* Copyright (c) 2017, Karl Fessel, lib everytime, All rights reserved.*/

// these macros run a command or block in regular intervals

/**
 * these macros depend on unspecified but very common behavior of C, C++ and
 * the target architectur:
 * - integer overflow for subtraction and addition
 * - if the return type of the time function is bigger than the storage
 *   the lower bits schould be stored
 *   (this behavior is specified for values that fit into the storrage type
 *    but is unspecified for values that are bigger)
 */

/* simple versions for easy reading:
 * #define every(X) static uint16_t before = 0; uint16_t now = millis();\
 *               if((now - before >= X) && (before = now,1) )
 *
 *     static uint32_t before = 0;
 *     unsigned long now = millis();
 *     if (now - before > 1000){
 *     before = now;
 */


//this make the variable names more unique by adding the current line number
#define IDCAT3( A, B) A ## B
#define IDCAT2( A, B) IDCAT3( A, B)
#define IDCAT(X) IDCAT2(X, __LINE__)

/**
 * these are gereric variants to create the mor specifc ones
 * X      is the time betwen two runs
 * START  is the time of the first start or -1 to wait for the first perion after time 0 (-1 saves memory and flash)
 * STARTO is the time of the first start with offset from first test (now) (0 to start now)
 * F      is the function that returns some kind of time
 *  e.g.: Arduinos millis() and micros()
 *  !! millis() do not include all milliseconds since its incremented in
 *  !!   256 / 250 kHz steps and are compensated every ~42 milliseconds by adding another millisecond !!
 *  !! micros() function itself takes more than a microsecond to execute !!
 * T      is the type of the storage variable the type should behave as said above
 *
 * everySTF(X, START, F, T)
 *  start with offset from time 0, -1 -> first intevall
 *
 * everyNTF(X, STARTO, F, T)
 *  start with offset from now
 */
#ifndef ET_DRIFT
// Dirft Mode
// 1 : drift if miss
//      e.g.: trigger every 5, miss at 10 execution, starts at 11 next trigger will be 16
// 2 : never drift but may loose its function if execution takes to long
//      e.g.: trigger every 5, miss at 10 and 15 -> double trigger
// 3 : no drift until big miss then drift to now
//      e.g.: trigger every 5, miss at 10 and 15 -> trigger once at 16 next trigger will be 21
// 4 : try to stay in messure, if big miss advance in steps
//      e.g.: trigger every 5, miss at 10 and 15 trigger once at 16 next trigger will be 20
#define ET_DRIFT 3
#endif

#if ET_DRIFT == 1
// 34 Byte for everySTF(X, -1, millis(), uint16_t )
// drift if miss
// non dynamic initialisation of static variable saves space in memory and flash 0 saves even more
#define everySTF(X, START, F, T) \
            T IDCAT(now) = (F);\
            static T IDCAT(before) = ((START) < 0) ? 0 : (0 - (X) + (START));\
            if( (IDCAT(now) - IDCAT(before) >= (X)) \
             && ((IDCAT(before) = IDCAT(now)),1) )

// start with offset from now
#define everyNTF(X, STARTO, F, T) \
            T IDCAT(now) = (F);\
            static (T) IDCAT(before) = IDCAT(now) - (X) + (STARTO);\
            if( (IDCAT(now) - IDCAT(before) >= (X)) \
             && ((IDCAT(before) = IDCAT(now)),1) )

#endif
#if ET_DRIFT == 2
// 38 Byte for everySTF(X, -1, millis(), uint16_t )
// no drift but may lose its function if miss is to big
// non dynamic initialisation of static variable saves space in memory and flash 0 saves even more
#define everySTF(X, START, F, T) \
            T IDCAT(now) = F;\
            static T IDCAT(before) = (START < 0) ? 0 : (0 - X + START);\
            if( (IDCAT(now) - IDCAT(before) >= X) \
             && ((IDCAT(before) += X),1) )

// start with offset from now
#define everyNTF(X, STARTO, F, T) \
            T IDCAT(now) = F;\
            static T IDCAT(before) = IDCAT(now) - X + STARTO;\
            if( (IDCAT(now) - IDCAT(before) >= X) \
             && ((IDCAT(before) += X),1) )
#endif
#if ET_DRIFT == 3
// 46 Byte for everySTF(X, -1, millis(), uint16_t )
// drift when big miss (two intervals) else no drift
// non dynamic initialisation of static variable saves space in memory and flash 0 saves even more
#define everySTF(X, START, F, T) \
            T IDCAT(now) = F;\
            static T IDCAT(before) = (START < 0) ? 0 : (0 - X + START);\
            if( (IDCAT(now) - IDCAT(before) >= X) \
              && ((IDCAT(before) = (IDCAT(now) - IDCAT(before) > 2 * X)?IDCAT(now):IDCAT(before) + X),1) )

// start with offset from now
#define everyNTF(X, STARTO, F, T) \
            T IDCAT(now) = F;\
            static T IDCAT(before) = IDCAT(now) - X + STARTO;\
            if( (IDCAT(now) - IDCAT(before) >= X) \
             && ((IDCAT(before) = (IDCAT(now) - IDCAT(before) > 2 * X)?IDCAT(now):IDCAT(before) + X),1) )
#endif
#if ET_DRIFT == 4
// 68 Bytes for everySTF(X, -1, millis(), uint16_t )
// this uses lambda functions for the caluculation of the increment
// lambda functions are part of c++11
#define everySTF(X, START, F, T) \
            T IDCAT(now) = F;\
            static T IDCAT(before) = (START < 0) ? 0 : (0 - X + START);\
            T IDCAT(dist) = IDCAT(now) - IDCAT(before);\
            if( (IDCAT(dist) >= X) \
             && (IDCAT(before) += [&](){ T step = 0; do{ step += X; }while( (IDCAT(dist) > step + X) && (step + X > step) ); return step; }() ,1  ))

// start with offset from now
#define everyNTF(X, STARTO, F, T) \
            T IDCAT(now) = F;\
            static T IDCAT(before) = IDCAT(now) - X + STARTO;\
            T IDCAT(dist) = IDCAT(now) - IDCAT(before);\
            T IDCAT(step) = 0;\
            if( (IDCAT(dist) >= X) \
             && (IDCAT(before) += [&](){ T step = 0; do{ step += X; }while( (IDCAT(dist) > step + X) && (step + X > step) ); return step; }() ,1  ))
#endif

// execute every X milliseconds (8 bit)
#define everyb(X)            everySTF(X, -1, millis(), uint8_t )
#define everynowb(X)         everyNTF(X,  0, millis(), uint8_t )

// execute every X milliseconds (16 bit)
#define every(X)            everySTF(X, -1, millis(), uint16_t )
#define everynow(X)         everyNTF(X,  0, millis(), uint16_t )

// execute every X milliseconds (32 bit)
#define everyl(X)            everySTF(X, -1, millis(), uint32_t )
#define everynowl(X)         everyNTF(X,  0, millis(), uint32_t )

// execute every X microseconds (16 bit)
// !! loop cycle may be longer on most arduinos than the resolution of this !!
// (30 microseconds with this just toggeling a LED using arduino method digitalWrite)
#define everyu(X)            everySTF(X, -1, micros(), uint16_t )
#define everynowu(X)         everyNTF(X,  0, micros(), uint16_t )

// execute every X microseconds (32 bit)
#define everyul(X)            everySTF(X, -1, micros(), uint32_t )
#define everynowul(X)         everyNTF(X,  0, micros(), uint32_t )

#endif
```
``` h IMU.h
#ifndef IMU_H
#define IMU_H

#include <inttypes.h>

#define IMU_ADDR  0x69

struct IMUdata
{
    int16_t accel_x;
    int16_t accel_y;
    int16_t accel_z;
    int16_t gyro_x;
    int16_t gyro_y;
    int16_t gyro_z;
};

struct IMUdataf
{
    float accel_x;
    float accel_y;
    float accel_z;
    float gyro_x;
    float gyro_y;
    float gyro_z;
};

void initializeIMU();
void readIMU(struct IMUdata& d);
uint8_t IMUWhoAreYou();
void readIMUscale(struct IMUdataf& d);
#endif
```
``` cpp IMU.cpp
#include <Arduino.h>
#include <Wire.h>
#include "IMU.h"
#include "IMURegisters.h"

#define IMU_Addr     0b1101001
#define I2C_clock    400000UL

#define ACCEL_FS_SEL 00
#define GYRO_FS_SEL  00

uint8_t IMU_WhoAmI;

void initializeIMU()
{
    uint8_t ret;
    //I2C Master
    Wire.begin();

    Wire.setClock(I2C_clock);

    //read WhoAmI
    Wire.beginTransmission(IMU_ADDR);
    Wire.write(IMU_WHO_AM_I);
    ret = Wire.endTransmission(false);

    if(ret)
    {
        IMU_WhoAmI=ret;
        return;
    }

    Wire.requestFrom(IMU_ADDR,1);
    IMU_WhoAmI=Wire.read();

    //disable powersave modes
    Wire.beginTransmission(IMU_ADDR);
    Wire.write(IMU_PWR_MGMT_1);
    Wire.write(0);
    //and _PWR_MGMT_2
    Wire.write(0);
    Wire.endTransmission();
//configuration
    Wire.beginTransmission(IMU_ADDR);
    Wire.write(IMU_GYRO_CONFIG);
    Wire.write(0|GYRO_FS_SEL<<3);
//  (IMU_ACCEL_CONFIG);
    Wire.write(0|ACCEL_FS_SEL<<3);
    Wire.endTransmission();
    //selftest ?
}

void readIMU(struct IMUdata& d)
{
    //read accelleromenter, temprature and gyroscope
    Wire.beginTransmission(IMU_ADDR);
    Wire.write(IMU_ACCEL_XOUT_H);
    Wire.endTransmission(false);
    Wire.requestFrom(IMU_ADDR,14);
    d.accel_x = Wire.read() << 8;
    d.accel_x |= Wire.read();
    d.accel_y = Wire.read() << 8;
    d.accel_y |= Wire.read();
    d.accel_z = Wire.read() << 8;
    d.accel_z |= Wire.read();
    //skip Temperature
    Wire.read();
    Wire.read();
//     Wire.beginTransmission(IMU_ADDR);
//     Wire.write(IMU_GYRO_XOUT_H);
//     Wire.endTransmission(false);
//     Wire.requestFrom(IMU_ADDR,6);
    d.gyro_x = Wire.read() << 8;
    d.gyro_x |= Wire.read();
    d.gyro_y = Wire.read() << 8;
    d.gyro_y |= Wire.read();
    d.gyro_z = Wire.read() << 8;
    d.gyro_z |= Wire.read();
}

void readIMUscale(struct IMUdataf& d)
{
}

uint8_t IMUWhoAreYou()
{
    return IMU_WhoAmI;
}
```
``` h -IMURegisters.h
#pragma once

#define IMU_SELF_TEST_X_GYR    0x00 /*0*/
#define IMU_SELF_TEST_Y_GYR    0x01 /*1*/
#define IMU_SELF_TEST_Z_GYR    0x02 /*2*/
#define IMU_SELF_TEST_X_ACC    0x0D /*13*/
#define IMU_SELF_TEST_Y_ACC    0x0E /*14*/
#define IMU_SELF_TEST_Z_ACC    0x0F /*15*/
#define IMU_XG_OFFSET_H        0x13 /*19*/
#define IMU_XG_OFFSET_L        0x14 /*20*/
#define IMU_YG_OFFSET_H        0x15 /*21*/
#define IMU_YG_OFFSET_L        0x16 /*22*/
#define IMU_ZG_OFFSET_H        0x17 /*23*/
#define IMU_ZG_OFFSET_L        0x18 /*24*/
#define IMU_SMPLRT_DIV         0x19 /*25*/
#define IMU_CONFIG             0x1A /*26*/
#define IMU_GYRO_CONFIG        0x1B /*27*/
#define IMU_ACCEL_CONFIG       0x1C /*28*/
#define IMU_ACCEL_CONFIG_2     0x1D /*29*/
#define IMU_LP_ACCEL_ODR       0x1E /*30*/
#define IMU_WOM_THR            0x1F /*31*/
#define IMU_FIFO_EN            0x23 /*35*/
#define IMU_I2C_MST_CTRL       0x24 /*36*/
#define IMU_I2C_SLV0_ADDR      0x25 /*37*/
#define IMU_I2C_SLV0_REG       0x26 /*38*/
#define IMU_I2C_SLV0_CTRL      0x27 /*39*/
#define IMU_I2C_SLV1_ADDR      0x28 /*40*/
#define IMU_I2C_SLV1_REG       0x29 /*41*/
#define IMU_I2C_SLV1_CTRL      0x2A /*42*/
#define IMU_I2C_SLV2_ADDR      0x2B /*43*/
#define IMU_I2C_SLV2_REG       0x2C /*44*/
#define IMU_I2C_SLV2_CTRL      0x2D /*45*/
#define IMU_I2C_SLV3_ADDR      0x2E /*46*/
#define IMU_I2C_SLV3_REG       0x2F /*47*/
#define IMU_I2C_SLV3_CTRL      0x30 /*48*/
#define IMU_I2C_SLV4_ADDR      0x31 /*49*/
#define IMU_I2C_SLV4_REG       0x32 /*50*/
#define IMU_I2C_SLV4_DO        0x33 /*51*/
#define IMU_I2C_SLV4_CTRL      0x34 /*52*/
#define IMU_I2C_SLV4_DI        0x35 /*53*/
#define IMU_I2C_MST_STATUS     0x36 /*54*/
#define IMU_INT_PIN_CFG        0x37 /*55*/
#define IMU_INT_ENABLE         0x38 /*56*/
#define IMU_INT_STATUS         0x3A /*58*/
#define IMU_ACCEL_XOUT_H       0x3B /*59*/
#define IMU_ACCEL_XOUT_L       0x3C /*60*/
#define IMU_ACCEL_YOUT_H       0x3D /*61*/
#define IMU_ACCEL_YOUT_L       0x3E /*62*/
#define IMU_ACCEL_ZOUT_H       0x3F /*63*/
#define IMU_ACCEL_ZOUT_L       0x40 /*64*/
#define IMU_TEMP_OUT_H         0x41 /*65*/
#define IMU_TEMP_OUT_L         0x42 /*66*/
#define IMU_GYRO_XOUT_H        0x43 /*67*/
#define IMU_GYRO_XOUT_L        0x44 /*68*/
#define IMU_GYRO_YOUT_H        0x45 /*69*/
#define IMU_GYRO_YOUT_L        0x46 /*70*/
#define IMU_GYRO_ZOUT_H        0x47 /*71*/
#define IMU_GYRO_ZOUT_L        0x48 /*72*/
#define IMU_EXT_SENS_DATA_00   0x49 /*73*/
#define IMU_EXT_SENS_DATA_01   0x4A /*74*/
#define IMU_EXT_SENS_DATA_02   0x4B /*75*/
#define IMU_EXT_SENS_DATA_03   0x4C /*76*/
#define IMU_EXT_SENS_DATA_04   0x4D /*77*/
#define IMU_EXT_SENS_DATA_05   0x4E /*78*/
#define IMU_EXT_SENS_DATA_06   0x4F /*79*/
#define IMU_EXT_SENS_DATA_07   0x50 /*80*/
#define IMU_EXT_SENS_DATA_08   0x51 /*81*/
#define IMU_EXT_SENS_DATA_09   0x52 /*82*/
#define IMU_EXT_SENS_DATA_10   0x53 /*83*/
#define IMU_EXT_SENS_DATA_11   0x54 /*84*/
#define IMU_EXT_SENS_DATA_12   0x55 /*85*/
#define IMU_EXT_SENS_DATA_13   0x56 /*86*/
#define IMU_EXT_SENS_DATA_14   0x57 /*87*/
#define IMU_EXT_SENS_DATA_15   0x58 /*88*/
#define IMU_EXT_SENS_DATA_16   0x59 /*89*/
#define IMU_EXT_SENS_DATA_17   0x5A /*90*/
#define IMU_EXT_SENS_DATA_18   0x5B /*91*/
#define IMU_EXT_SENS_DATA_19   0x5C /*92*/
#define IMU_EXT_SENS_DATA_20   0x5D /*93*/
#define IMU_EXT_SENS_DATA_21   0x5E /*94*/
#define IMU_EXT_SENS_DATA_22   0x5F /*95*/
#define IMU_EXT_SENS_DATA_23   0x60 /*96*/
#define IMU_I2C_SLV0_DO        0x63 /*99*/
#define IMU_I2C_SLV1_DO        0x64 /*100*/
#define IMU_I2C_SLV2_DO        0x65 /*101*/
#define IMU_I2C_SLV3_DO        0x66 /*102*/
#define IMU_I2C_MST_DELAY_CTRL 0x67 /*103*/
#define IMU_SIGNAL_PATH_RESET  0x68 /*104*/
#define IMU_MOT_DETECT_CTRL    0x69 /*105*/
#define IMU_USER_CTRL          0x6A /*106*/
#define IMU_PWR_MGMT_1         0x6B /*107*/
#define IMU_PWR_MGMT_2         0x6C /*108*/
#define IMU_FIFO_COUNTH        0x72 /*114*/
#define IMU_FIFO_COUNTL        0x73 /*115*/
#define IMU_FIFO_R_W           0x74 /*116*/
#define IMU_WHO_AM_I           0x75 /*117*/
#define IMU_XA_OFFSET_H        0x77 /*119*/
#define IMU_XA_OFFSET_L        0x78 /*120*/
#define IMU_YA_OFFSET_H        0x7A /*122*/
#define IMU_YA_OFFSET_L        0x7B /*123*/
#define IMU_ZA_OFFSET_H        0x7D /*125*/
#define IMU_ZA_OFFSET_L        0x7E /*126*/
```
``` h Motor.h
#pragma once

#include <inttypes.h>

// initialise timer(s) here
void initMotors();
// control the motor-power, pay attention to the direction of rotation
void setMotors(int8_t left, int8_t right);

// you may use this to set Motor-DDRs
void deactivateMotors();
void activateMotors();
```
``` cpp Motor.cpp

```
@sketch

# Quizze

@init_clear

    --{{0}}--
Wie auch in der letzten Aufgabe haben wir noch ein paar kurze Fragen an euch.

## Analog-Digital-Wandler

@init_clear

                      --{{0}}--
Welche Vorteile hat ein 8-Bit Flash-ADC gegenüber einem 8-Bit
Analog-Digital-Wandler, der nach dem Wägeverfahren arbeitet?

                       {{0-1}}
    [[X]] Er benötigt keinen Digital-Analog-Wandler
    [[ ]] Er benötigt weniger Komparatoren
    [[X]] Er benötigt weniger Taktzyklen
    [[ ]] Er hat (im Allgemeinen) geringere Anschaffungskosten


                      --{{1}}--
Welche Schaltung wird zur Umsetzung eines Analog-Digital-Wandler benötigt?

                       {{1-2}}
    [( )] Eine H-Brücke
    [( )] Ein Schwingkreis
    [(X)] Eine Sample-And-Hold Schaltung
    [( )] Ein Gleichrichter

                      --{{2}}--
Wie viele Komparatoren benötigt ein 10-Bit Flash-ADC?

                        {{2}}
    [( )] 1021
    [( )] 1022
    [(X)] 1023
    [( )] 1024


## Distanzsensoren

@init_clear

                      --{{0}}--
Welche Vorgehen zur lichtbasierten Distanzmessung können unterschieden werden?

                       {{0-1}}
    [[X]] Time-of-Flight
    [[X]] Laufzeitmessung
    [[X]] Triangulation
    [[ ]] Quadratur
    [[X]] Phasenverschiebung
    [[ ]] Parallelverschiebung

                      --{{1}}--
Auf welchem Vorgehen zur Distanzmessung basiert der SharpGP2Y0A41SK0F?

                        {{1}}
    [( )] Time-of-Flight
    [( )] Laufzeitmessung
    [(X)] Triangulation
    [( )] Quadratur
    [( )] Phasenverschiebung
    [( )] Parallelverschiebung

## Inertiale Messeinheit

@init_clear

                      --{{0}}--
Welche Sensoren enthält die IMU (MPU-9255)?

                       {{0-1}}
    [( )] Temperatursensor, Distanzsensor
    [( )] Gyroskop, Beschleunigungssensor, Distanzsensor
    [(X)] Magnetometer, Gyroskop, Beschleunigungssensor, Temperatursensor
    [( )] Magnetometer, Beschleunigungssensor, Distanzsensor


                      --{{1}}--
Wie viele *Degrees of freedom* hat die auf dem Roboter verbaute IMU (MPU-9255)?

                       {{1-2}}
    [( )] 6
    [(X)] 9
    [( )] 10


                      --{{2}}--
Welche Größen werden für eine Richtungsachse innerhalb der IMU (MPU-9255)
gemessen?

                        {{2}}
    [(X)] magnetsiche Flussdichte, Beschleunigung, Winkeländerung
    [( )] magnetische Flussdichte, Geschwindigkeit, Drehwinkelbeschleunigung
    [( )] magnetische Flussdichte, Geschwindigkeit, Orientierung
    [( )] magnetische Feldstärke, Beschleunigung, Orientierung
