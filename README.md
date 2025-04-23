-<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Booklet - Accessible Sign In / Sign Up</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Segoe+UI&display=swap');

    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      margin: 0;
      padding: 2rem 1.5rem;
      background: #e9eff6;
      color: #333;
      display: flex;
      flex-direction: column;
      align-items: center;
      min-height: 100vh;
    }
    h1, h2 {
      color: navy;
      text-align: center;
      margin: 0 0 1rem 0;
    }
    .card {
      background: #fff;
      padding: 2rem;
      margin: 1em auto;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.12);
      max-width: 420px;
      width: 100%;
      box-sizing: border-box;
    }
    label {
      display: block;
      margin-bottom: 0.4rem;
      font-weight: 600;
    }
    input[type="text"],
    input[type="password"] {
      width: 100%;
      padding: 0.7em 1em;
      font-size: 1rem;
      border: 1.5px solid #ccc;
      border-radius: 6px;
      margin-bottom: 1.25rem;
      transition: border-color 0.3s ease;
    }
    input[type="text"]:focus,
    input[type="password"]:focus {
      outline: none;
      border-color: #004080;
      box-shadow: 0 0 6px #6699ff80;
    }
    .toggle-password {
      font-size: 0.9rem;
      color: #0073e6;
      cursor: pointer;
      margin-top: -1rem;
      margin-bottom: 1.25rem;
      user-select: none;
      text-align: right;
      display: block;
    }
    button {
      width: 100%;
      background: navy;
      color: white;
      border: none;
      padding: 0.85em;
      font-size: 1rem;
      font-weight: 700;
      border-radius: 6px;
      cursor: pointer;
      transition: background-color 0.25s ease, transform 0.15s ease;
    }
    button:hover,
    button:focus {
      background-color: #003366;
      transform: scale(1.05);
      outline: none;
    }
    .feedback {
      color: #d9534f;
      font-size: 0.9rem;
      min-height: 1.4em;
      margin-bottom: 1em;
    }
    .info {
      margin-top: 0.8em;
      font-size: 0.9rem;
      color: #004080;
      text-align: center;
      cursor: pointer;
      user-select: none;
    }
    .sr-only {
      position: absolute !important;
      width: 1px !important;
      height: 1px !important;
      padding: 0 !important;
      margin: -1px !important;
      overflow: hidden !important;
      clip: rect(0,0,0,0) !important;
      white-space: nowrap !important;
      border: 0 !important;
    }
  </style>
</head>
<body>

  <h1>Booklet</h1>

  <section class="card" aria-labelledby="signin-label">
    <h2 id="signin-label">Sign In / Sign Up</h2>

    <form id="authForm" novalidate aria-describedby="authFeedback" aria-live="polite">
      <label for="usernameInput">Username</label>
      <input type="text" id="usernameInput" name="username" autocomplete="username" aria-required="true" required aria-describedby="usernameHint" />
      <div id="usernameHint" class="sr-only">Enter your username. Letters and numbers only.</div>

      <label for="passwordInput">Password</label>
      <input type="password" id="passwordInput" name="password" autocomplete="current-password" aria-required="true" required />
      <span class="toggle-password" role="button" tabindex="0" aria-pressed="false" id="togglePassword" aria-label="Show or hide password">Show Password</span>

      <div class="feedback" id="authFeedback" role="alert" aria-live="assertive"></div>

      <button type="submit" id="loginBtn">Enter</button>
    </form>

    <p class="info" id="toggleMode" tabindex="0">Don't have an account? Sign up here</p>
  </section>

  <script>
    (() => {
      const usersKey = 'users';
      const users = JSON.parse(localStorage.getItem(usersKey) || '{}');

      const form = document.getElementById('authForm');
      const usernameInput = form.username;
      const passwordInput = form.password;
      const feedback = document.getElementById('authFeedback');
      const togglePasswordBtn = document.getElementById('togglePassword');
      const toggleModeText = document.getElementById('toggleMode');
      let isSignUp = false;

      function saveUsers() {
        localStorage.setItem(usersKey, JSON.stringify(users));
      }

      // Basic username validation: letters, numbers, underscore, min 3 chars
      function isValidUsername(str) {
        return /^[a-zA-Z0-9_]{3,}$/.test(str);
      }

      // Basic password validation: min 6 chars
      function isValidPassword(str) {
        return str.length >= 6;
      }

      function clearFeedback() {
        feedback.textContent = '';
      }

      // Toggle password visibility
      togglePasswordBtn.onclick = () => {
        const type = passwordInput.type === 'password' ? 'text' : 'password';
        passwordInput.type = type;
        togglePasswordBtn.textContent = type === 'password' ? 'Show Password' : 'Hide Password';
        togglePasswordBtn.setAttribute('aria-pressed', type === 'text');
      };
      togglePasswordBtn.onkeydown = e => {
        if (e.key === 'Enter' || e.key === ' ') {
          e.preventDefault();
          togglePasswordBtn.click();
        }
      };

      // Toggle between signin and signup modes
      function toggleMode() {
        isSignUp = !isSignUp;
        clearFeedback();
        form.reset();
        if (isSignUp) {
          toggleModeText.textContent = 'Already have an account? Sign in here';
          form.querySelector('button').textContent = 'Sign Up';
          passwordInput.setAttribute('autocomplete', 'new-password');
        } else {
          toggleModeText.textContent = "Don't have an account? Sign up here";
          form.querySelector('button').textContent = 'Enter';
          passwordInput.setAttribute('autocomplete', 'current-password');
        }
        usernameInput.focus();
      }
      toggleModeText.onclick = toggleMode;
      toggleModeText.onkeydown = e => {
        if (e.key === 'Enter' || e.key === ' ') {
          e.preventDefault();
          toggleMode();
        }
      };

      form.onsubmit = e => {
        e.preventDefault();
        clearFeedback();

        const username = usernameInput.value.trim();
        const password = passwordInput.value;

        // Validate inputs
        if (!username) {
          feedback.textContent = 'Username is required.';
          usernameInput.focus();
          return;
        }
        if (!isValidUsername(username)) {
          feedback.textContent = 'Username must be 3+ characters and contain only letters, numbers, or underscores.';
          usernameInput.focus();
          return;
        }
        if (!password) {
          feedback.textContent = 'Password is required.';
          passwordInput.focus();
          return;
        }
        if (!isValidPassword(password)) {
          feedback.textContent = 'Password must be at least 6 characters long.';
          passwordInput.focus();
          return;
        }

        if (isSignUp) {
          // Sign-up flow
          if (users[username]) {
            feedback.textContent = 'Username already exists. Please choose another.';
            usernameInput.focus();
            return;
          }
          users[username] = { password, businesses: {} };
          saveUsers();
          feedback.style.color = 'green';
          feedback.textContent = 'Account successfully created! You can now sign in.';
          toggleMode(); // Switch to sign-in mode after signup
        } else {
          // Sign-in flow
          if (!users[username]) {
            feedback.textContent = 'No account found with that username.';
            usernameInput.focus();
            return;
          }
          if (users[username].password !== password) {
            feedback.textContent = 'Incorrect password.';
            passwordInput.focus();
            return;
          }
          // Successful sign-in - redirect or init app here
          feedback.style.color = 'green';
          feedback.textContent = `Welcome back, ${username}! Proceeding to your dashboard...`;
          // Example: you can trigger initialization of your app
          setTimeout(() => {
            alert(`Successfully signed in as ${username}. Please implement dashboard transition.`);
            feedback.textContent = '';
            form.reset();
          }, 1200);
        }
      };
    })();
  </script>

</body>
</html>
