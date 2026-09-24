<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Clap'n'Wake</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: #f3f4f6;
      color: #111827;
    }

    .app {
      max-width: 480px;
      margin: auto;
      padding: 25px 18px 40px;
    }

    h1 {
      font-size: 32px;
      margin-bottom: 5px;
    }

    .description {
      color: #6b7280;
      margin-bottom: 25px;
    }

    .card {
      background: white;
      border-radius: 24px;
      padding: 20px;
      margin: 15px 0;
      box-shadow: 0 5px 20px rgba(0,0,0,0.08);
    }

    .clock {
      text-align: center;
      font-size: 64px;
      font-weight: bold;
      margin: 10px 0;
    }

    .status {
      text-align: center;
      font-weight: bold;
    }

    .row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 10px;
    }

    input {
      font-size: 22px;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 12px;
    }

    button {
      width: 100%;
      border: none;
      border-radius: 14px;
      padding: 15px;
      margin-top: 10px;
      font-size: 16px;
      font-weight: bold;
      background: #111827;
      color: white;
    }

    button.secondary {
      background: #e5e7eb;
      color: #111827;
    }

    #alarm {
      background: #111827;
      color: white;
    }

    .command {
      background: #f3f4f6;
      color: #111827;
      padding: 12px;
      border-radius: 12px;
      margin: 8px 0;
    }
  </style>
</head>

<body>

<div class="app">

  <h1>Clap'n'Wake ⏰</h1>

  <p class="description">
    Le réveil pour les jours où on a la flemme de se lever.
  </p>

  <div class="card">

    <div class="clock" id="clock">
      --:--
    </div>

    <div class="status" id="status">
      Réveil désactivé
    </div>

  </div>


  <div class="card">

    <div class="row">

      <b>Heure du réveil</b>

      <input
        id="alarmTime"
        type="time"
      >

    </div>

    <button id="activateButton">
      Activer le réveil
    </button>

  </div>


  <div class="card">

    <h2>🎤 Microphone</h2>

    <button id="microButton">
      Activer le microphone
    </button>

    <p id="microStatus">
      Microphone désactivé
    </p>

  </div>


  <div class="card">

    <h2>🗣️ Reconnaissance vocale</h2>

    <button id="voiceButton">
      Activer la reconnaissance vocale
    </button>

    <p id="voiceStatus">
      Reconnaissance vocale désactivée
    </p>

  </div>


  <div class="card" id="alarm" style="display:none">

    <h2>⏰ DEBOUT !</h2>

    <p>Le réveil sonne...</p>

    <div class="command">
      👏 1 claquement → +5 minutes
    </div>

    <div class="command">
      👏👏 2 claquements → arrêt
    </div>

    <div class="command">
      🗣️ « Encore 5 minutes » → +5 minutes
    </div>

    <div class="command">
      🗣️ « Stop » → arrêt
    </div>

    <button
      class="secondary"
      onclick="stopAlarm()"
    >
      Arrêter
    </button>

  </div>

</div>


<script>

let alarmActive = false;
let alarmRinging = false;

let microphoneStream = null;
let audioContext = null;
let analyser = null;

let lastClap = 0;
let clapCount = 0;
let clapTimer = null;

let recognition = null;


/* =========================
   HORLOGE
========================= */

function updateClock() {

  const now = new Date();

  const hours =
    String(now.getHours()).padStart(2, "0");

  const minutes =
    String(now.getMinutes()).padStart(2, "0");

  document.getElementById("clock").textContent =
    hours + ":" + minutes;


  if (
    alarmActive &&
    !alarmRinging &&
    document.getElementById("alarmTime").value ===
    hours + ":" + minutes
  ) {

    startAlarm();

  }

}

setInterval(updateClock, 1000);

updateClock();


/* =========================
   ACTIVER LE RÉVEIL
========================= */

document
  .getElementById("activateButton")
  .onclick = function() {

    alarmActive = !alarmActive;

    if (alarmActive) {

      this.textContent =
        "Désactiver le réveil";

      document.getElementById("status").textContent =
        "Réveil activé ✅";

    } else {

      this.textContent =
        "Activer le réveil";

      document.getElementById("status").textContent =
        "Réveil désactivé";

      stopAlarm();

    }

  };


/* =========================
   ALARME
========================= */

function startAlarm() {

  alarmRinging = true;

  document.getElementById("alarm").style.display =
    "block";

  document.getElementById("status").textContent =
    "Alarme en cours 🔔";


  if (navigator.vibrate) {

    navigator.vibrate([
      400,
      150,
      400,
      150,
      700
    ]);

  }

}


function snooze() {

  alarmRinging = false;

  document.getElementById("alarm").style.display =
    "none";


  const newTime =
    new Date(Date.now() + 5 * 60 * 1000);


  const hours =
    String(newTime.getHours()).padStart(2, "0");

  const minutes =
    String(newTime.getMinutes()).padStart(2, "0");


  document.getElementById("alarmTime").value =
    hours + ":" + minutes;


  document.getElementById("status").textContent =
    "Encore 5 minutes 😴";

}


function stopAlarm() {

  alarmRinging = false;

  clapCount = 0;

  document.getElementById("alarm").style.display =
    "none";


  document.getElementById("status").textContent =
    alarmActive
      ? "Réveil activé ✅"
      : "Réveil désactivé";

}


/* =========================
   MICROPHONE
========================= */

document
  .getElementById("microButton")
  .onclick = async function() {

    try {

      microphoneStream =
        await navigator.mediaDevices
          .getUserMedia({
            audio: true
          });


      audioContext =
        new (
          window.AudioContext ||
          window.webkitAudioContext
        )();


      const source =
        audioContext
          .createMediaStreamSource(
            microphoneStream
          );


      analyser =
        audioContext.createAnalyser();

      analyser.fftSize = 512;

      source.connect(analyser);


      document.getElementById("microStatus")
        .textContent =
        "Microphone activé ✅";


      detectClaps();


    } catch (error) {

      document.getElementById("microStatus")
        .textContent =
        "Impossible d'accéder au microphone.";

    }

  };


/* =========================
   DÉTECTION DES CLAQUEMENTS
========================= */

function detectClaps() {

  if (!analyser) return;


  const data =
    new Uint8Array(
      analyser.fftSize
    );


  analyser.getByteTimeDomainData(data);


  let sum = 0;


  for (let value of data) {

    const normalized =
      (value - 128) / 128;

    sum +=
      normalized * normalized;

  }


  const volume =
    Math.sqrt(
      sum / data.length
    );


  const currentTime =
    performance.now();


  /*
    Seuil du claquement.
    Il faudra probablement l'ajuster
    avec votre vrai micro.
  */

  if (
    alarmRinging &&
    volume > 0.16 &&
    currentTime - lastClap > 400
  ) {

    lastClap = currentTime;

    clapCount++;


    if (clapCount === 1) {

      clearTimeout(clapTimer);


      clapTimer =
        setTimeout(function() {

          if (clapCount === 1) {

            snooze();

          }

          clapCount = 0;

        }, 900);

    }


    else if (clapCount === 2) {

      clearTimeout(clapTimer);

      clapCount = 0;

      stopAlarm();

    }

  }


  requestAnimationFrame(
    detectClaps
  );

}


/* =========================
   RECONNAISSANCE VOCALE
========================= */

document
  .getElementById("voiceButton")
  .onclick = function() {

    const SpeechRecognition =
      window.SpeechRecognition ||
      window.webkitSpeechRecognition;


    if (!SpeechRecognition) {

      document.getElementById("voiceStatus")
        .textContent =
        "La reconnaissance vocale n'est pas disponible dans ce navigateur.";

      return;

    }


    if (!recognition) {

      recognition =
        new SpeechRecognition();


      recognition.lang =
        "fr-FR";


      recognition.continuous =
        true;


      recognition.interimResults =
        false;


      recognition.onresult =
        function(event) {

          const result =
            event.results[
              event.results.length - 1
            ][0].transcript
            .toLowerCase();


          document.getElementById(
            "voiceStatus"
          ).textContent =
            "Entendu : « " +
            result +
            " »";


          /*
            REPORT
          */

          if (
            alarmRinging &&
            (
              result.includes("encore") ||
              result.includes("cinq minutes") ||
              result.includes("5 minutes")
            )
          ) {

            snooze();

          }


          /*
            ARRÊT
          */

          if (
            alarmRinging &&
            (
              result.includes("stop") ||
              result.includes("arrête") ||
              result.includes("arrêt")
            )
          ) {

            stopAlarm();

          }

        };


      recognition.onerror =
        function() {

          document.getElementById(
            "voiceStatus"
          ).textContent =
            "Erreur de reconnaissance vocale.";

        };

    }


    try {

      recognition.start();

      document.getElementById(
        "voiceStatus"
      ).textContent =
        "Reconnaissance vocale activée ✅";

    }

    catch (error) {

      console.log(error);

    }

  };

</script>

</body>
</html>