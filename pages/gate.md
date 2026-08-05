---
layout: page
title: Get Your Free Download
permalink: /gate
subtitle:
show-avatar: false
---

<div class="gate-container">
  <p>Enter your email to receive your download.</p>
  <div id="gate-error" class="gate-error"></div>
  <form id="gateForm">
    <input type="email" id="gate-email" placeholder="Enter your email" required>
    <div class="gate-checkbox-group">
      <label>
        <input type="checkbox" id="gate-consent" required>
        I agree to receive communications.
      </label>
    </div>
    <button type="submit">Submit</button>
  </form>
</div>

<style>
.gate-container {
  max-width: 480px;
  margin: 20px auto;
  text-align: center;
}
.gate-container form {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.gate-container input[type="email"] {
  padding: 10px;
  width: 100%;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.gate-checkbox-group {
  margin: 10px 0;
  font-size: 0.9em;
}
.gate-container button {
  padding: 10px 24px;
  background: #008AFF;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1em;
}
.gate-container button:hover {
  background: #0085A1;
}
.gate-error {
  color: #c0392b;
  margin-bottom: 10px;
  min-height: 1.2em;
}
</style>

<script>
(function() {
  var storedEmail = localStorage.getItem("user_email");
  var emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (storedEmail && emailRegex.test(storedEmail)) {
    var urlParams = new URLSearchParams(window.location.search);
    var fileParam = urlParams.get("file") || "download1";
    window.location.href = "/confirmation?file=" + encodeURIComponent(fileParam);
  }
})();

document.getElementById("gateForm").addEventListener("submit", function(e) {
  e.preventDefault();
  var emailField = document.getElementById("gate-email");
  var consent = document.getElementById("gate-consent").checked;
  var errorDiv = document.getElementById("gate-error");
  errorDiv.textContent = "";

  if (!emailField.checkValidity()) {
    errorDiv.textContent = "Please enter a valid email address.";
    return;
  }
  if (!consent) {
    errorDiv.textContent = "You must agree to the terms.";
    return;
  }

  var urlParams = new URLSearchParams(window.location.search);
  var fileParam = urlParams.get("file") || "download1";

  localStorage.setItem("user_email", emailField.value);
  window.location.href = "/confirmation?file=" + encodeURIComponent(fileParam);
});
</script>
