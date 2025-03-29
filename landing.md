<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Email Verification</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f4f4;
      margin: 0;
      padding: 20px;
    }
    .container {
      max-width: 600px;
      margin: 50px auto;
      background: #fff;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    h1 {
      text-align: center;
    }
    form {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    input[type="email"] {
      padding: 10px;
      width: 80%;
      margin: 10px 0;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    .checkbox-group {
      margin: 10px 0;
    }
    button {
      padding: 10px 20px;
      background: #007BFF;
      color: #fff;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    button:hover {
      background: #0056b3;
    }
    .error {
      color: red;
      margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Email Verification</h1>
    <p>Please enter your email to receive your download.</p>
    <div id="error" class="error"></div>
    <form id="verificationForm">
      <input type="email" id="email" placeholder="Enter your email" required>
      <div class="checkbox-group">
        <label>
          <input type="checkbox" id="consent" required>
          I authorize failclosed to use my email address for marketing, sales, and social media communications.
        </label>
      </div>
      <button type="submit">Submit</button>
    </form>
  </div>

  <script>
    // On form submission, validate inputs and then redirect to confirmation.html with the chosen file key.
    document.getElementById("verificationForm").addEventListener("submit", function(e) {
      e.preventDefault();
      var emailField = document.getElementById("email");
      var consent = document.getElementById("consent").checked;
      var errorDiv = document.getElementById("error");
      errorDiv.textContent = "";
      
      if (!emailField.checkValidity()) {
         errorDiv.textContent = "Please enter a valid email address.";
         return;
      }
      if (!consent) {
         errorDiv.textContent = "You must agree to the terms.";
         return;
      }
      
      // Retrieve the 'file' parameter from the URL query string
      var urlParams = new URLSearchParams(window.location.search);
      var fileParam = urlParams.get("file") || "download1"; // default to download1 if missing
      
      // Optionally, store the email in localStorage (if needed for future reference)
      localStorage.setItem("user_email", emailField.value);
      
      // Redirect to confirmation.html with the file parameter preserved
      window.location.href = "confirmation.html?file=" + encodeURIComponent(fileParam);
    });
  </script>
</body>
</html>