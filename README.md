# reminder
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
  <title>💧 Cute Water Reminder</title>

  <!-- PWA -->
  <meta name="theme-color" content="#74d6ff">
  <link rel="manifest" href="manifest.json">

  <style>
    * { box-sizing: border-box; }
    body {
      font-family: 'Trebuchet MS', Arial, sans-serif;
      background: linear-gradient(135deg, #74d6ff, #a0c4ff, #c3f0ff, #ffffff);
      background-size: 400% 400%;
      animation: bgMove 10s ease infinite;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      -webkit-tap-highlight-color: transparent;
    }
    @keyframes bgMove {
      0% { background-position: 0% 50%; }
      50% { background-position: 100% 50%; }
      100% { background-position: 0% 50%; }
    }

    .screen {
      width: 320px;
      max-height: 640px;
      border-radius: 20px;
      overflow: hidden;
      box-shadow: 0 15px 30px rgba(0,0,0,0.15);
      background: white;
    }

    .page {
      padding: 20px 18px 24px;
      height: 100%;
    }

    .home, .settings, .app {
      display: none;
    }

    .home.active, .settings.active, .app.active {
      display: block;
    }

    h1 {
      margin: 0;
      font-size: 26px;
      text-align: center;
    }

    p {
      margin: 10px 0 20px;
      color: #666;
      text-align: center;
    }

    button {
      padding: 12px 20px;
      border-radius: 16px;
      border: none;
      cursor: pointer;
      background: linear-gradient(135deg, #84fab0, #8fd3f4);
      font-weight: bold;
      font-size: 16px;
      margin: 8px 0;
      width: 100%;
    }

    .row {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      justify-content: center;
    }

    .box {
      padding: 12px;
      border-radius: 16px;
      border: 2px solid #f1f1f1;
      text-align: center;
      width: 140px;
      cursor: pointer;
    }

    .box:hover {
      transform: scale(1.03);
    }

    .waterCup {
      text-align: center;
      margin-top: 10px;
      font-size: 60px;
    }

    .progress {
      text-align: center;
      margin-top: 8px;
      font-size: 18px;
      font-weight: bold;
    }

    .footer {
      text-align: center;
      margin-top: 14px;
      font-size: 12px;
      opacity: 0.7;
    }
  </style>
</head>

<body>
  <div class="screen">
    <div class="page">

      <!-- Home Page -->
      <div id="home" class="home active">
        <h1>💧 Cute Water Reminder</h1>
        <p>Stay hydrated with cute reminders!</p>

        <button onclick="openApp()">Start Reminder</button>
        <button onclick="openSettings()">Settings ⚙️</button>
      </div>

      <!-- Settings -->
      <div id="settings" class="settings">
        <h1>⚙️ Settings</h1>
        <p>Choose reminder time and cup size.</p>

        <div class="row">
          <div class="box" onclick="setCup(200)">🥤 200ml</div>
          <div class="box" onclick="setCup(300)">🥤 300ml</div>
          <div class="box" onclick="setCup(400)">🥤 400ml</div>
        </div>

        <p>Reminder every:</p>
        <div class="row">
          <div class="box" onclick="setIntervalMin(30)">⏱️ 30 min</div>
          <div class="box" onclick="setIntervalMin(45)">⏱️ 45 min</div>
          <div class="box" onclick="setIntervalMin(60)">⏱️ 60 min</div>
        </div>

        <button onclick="backHome()">Back</button>
      </div>

      <!-- App Page -->
      <div id="app" class="app">
        <h1>💧 Hydration Tracker</h1>
        <div class="waterCup" id="cupIcon">🥤</div>
        <div class="progress" id="progressText">0 / 0 ml</div>

        <div class="row">
          <button onclick="drink()">Drink 💦</button>
          <button onclick="reset()">Reset 🔄</button>
        </div>

        <div class="footer">You will get notifications every set time.</div>
      </div>

    </div>
  </div>

  <script>
    let cupSize = 300;
    let intervalMin = 60;
    let totalGoal = 2000; // daily goal

    let drank = 0;
    let reminderTimer;

    // load saved data
    const saved = JSON.parse(localStorage.getItem('waterData'));
    if (saved) {
      cupSize = saved.cupSize;
      intervalMin = saved.intervalMin;
      drank = saved.drank;
    }

    function saveData() {
      localStorage.setItem('waterData', JSON.stringify({
        cupSize, intervalMin, drank
      }));
    }

    function openApp() {
      document.getElementById('home').classList.remove('active');
      document.getElementById('settings').classList.remove('active');
      document.getElementById('app').classList.add('active');

      updateProgress();
      startReminder();
    }

    function openSettings() {
      document.getElementById('home').classList.remove('active');
      document.getElementById('settings').classList.add('active');
      document.getElementById('app').classList.remove('active');
    }

    function backHome() {
      document.getElementById('home').classList.add('active');
      document.getElementById('settings').classList.remove('active');
      document.getElementById('app').classList.remove('active');
      stopReminder();
    }

    function setCup(size) {
      cupSize = size;
      saveData();
      alert("Cup size set to " + size + " ml");
    }

    function setIntervalMin(min) {
      intervalMin = min;
      saveData();
      alert("Reminder set to every " + min + " minutes");
      if (document.getElementById('app').classList.contains('active')) {
        startReminder();
      }
    }

    function updateProgress() {
      const text = `${drank} / ${totalGoal} ml`;
      document.getElementById('progressText').innerText = text;
    }

    function drink() {
      drank += cupSize;
      if (drank > totalGoal) drank = totalGoal;
      updateProgress();
      saveData();
    }

    function reset() {
      drank = 0;
      updateProgress();
      saveData();
    }

    // notifications
    function startReminder() {
      stopReminder();
      requestNotificationPermission();
      reminderTimer = setInterval(() => {
        showNotification();
      }, intervalMin * 60 * 1000);
    }

    function stopReminder() {
      if (reminderTimer) clearInterval(reminderTimer);
    }

    function requestNotificationPermission() {
      if (Notification.permission === "default") {
        Notification.requestPermission();
      }
    }

    function showNotification() {
      if (Notification.permission === "granted") {
        new Notification("💧 Time to drink water!", {
          body: "Drink " + cupSize + "ml now 💦",
          icon: "https://i.imgur.com/7H1nWmV.png"
        });
      }
    }
  </script>
</body>
</html>
