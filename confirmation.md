<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Thank You for Your Download</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 0;
    }
    .container {
      max-width: 600px;
      margin: 50px auto;
      background: #fff;
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
    .download-link {
      display: inline-block;
      margin-top: 20px;
      padding: 10px 20px;
      background: #28a745;
      color: #fff;
      text-decoration: none;
      border-radius: 4px;
      font-size: 16px;
    }
    .download-link:hover {
      background: #218838;
    }
  </style>
</head>
<body>
  <div class="container" id="thank-you-container" style="display: block;">
    <h1>Thank You!</h1>
    <p>Your download will start automatically in a few seconds. If it does not, please click the "Download Now" button below.</p>
    <!-- Congratulations, you have completed level 1. If you enjoy my content, please consider supporting me at https://ko-fi.com/failclosed. Your flag is http://bit.ly/3Y8joi8 -->
    <a class="download-link" id="downloadBtn" href="" download>Download Now</a>
  </div>

  <script>
    window.onload = function() {
      // Mapping of file keys to download URLs
      var downloadLinks = {
        "download1": "https://drive.google.com/uc?export=download&id=1LMKrL0OsDyP0AQVHqsikjcp7KikgLKL3",
        "download2": "https://example.com/path/to/secondfile.pdf",
        "download3": "https://example.com/path/to/thirdfile.pdf"
      };

      // Retrieve the 'file' parameter from the URL query string
      var urlParams = new URLSearchParams(window.location.search);
      var fileKey = urlParams.get('file');

      // Choose the correct download URL (default to 'download1' if missing or invalid)
      var downloadUrl = downloadLinks[fileKey] || downloadLinks["download1"];

      // Set the href attribute of the download button dynamically
      document.getElementById('downloadBtn').href = downloadUrl;

      // Auto-trigger the download by simulating a click on the download link
      document.getElementById('downloadBtn').click();

      // Redirect to failclosed.com after 3 seconds
      setTimeout(function(){
        window.location.href = "https://failclosed.com";
      }, 3000);
    };
  </script>
</body>
</html>