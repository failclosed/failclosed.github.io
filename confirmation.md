<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Thank You for Downloading</title>
  <style>
    /* Basic styling for the thank you page */
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 0;
    }
    .container {
      max-width: 600px;
      margin: 50px auto;
      background-color: #fff;
      padding: 30px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      border-radius: 8px;
      text-align: center;
    }
    h1 {
      color: #333;
    }
    p {
      color: #666;
    }
    a.download-link {
      display: inline-block;
      margin-top: 20px;
      padding: 10px 20px;
      background-color: #28a745;
      color: #fff;
      text-decoration: none;
      border-radius: 4px;
      font-size: 16px;
    }
    a.download-link:hover {
      background-color: #218838;
    }
  </style>
</head>
<body>
  <div class="container" id="thank-you-container" style="display: none;">
    <h1>Thank You!</h1>
    <p>Your download is ready. Click the button below to access your digital product.</p>
    <!-- Congratulations, you have completed level 1. If you enjoy my content, please consider supporting me at ko-fi.com/failclosed. Your flag is http://bit.ly/3Y8joi8 -->
    <a class="download-link" href="https://drive.google.com/uc?export=download&id=1LMKrL0OsDyP0AQVHqsikjcp7KikgLKL3" download>Download Now</a>
  </div>
  
  <script>
    // Check if the user has validated their email using sessionStorage
    if (sessionStorage.getItem('validated') === 'true') {
      // Show the thank you content if validated
      document.getElementById('thank-you-container').style.display = "block";
    } else {
      // Redirect to the landing page if not validated
      window.location.href = "https://failclosed.com";
    }
  </script>
</body>
</html>
