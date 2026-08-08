---
layout: page
title: Thank You for Your Download
permalink: /confirmation
subtitle:
show-avatar: false
---

<div class="confirmation-container" id="thank-you-container">
  <p>Your download will start automatically in a few seconds. If it does not, please click the "Download Now" button below.</p>
  <!-- Congratulations, you have completed level 1. If you enjoy my content, please consider supporting me at https://ko-fi.com/failclosed. Your flag is https://bit.ly/4bDVNga -->
  <a class="confirmation-download-link" id="downloadBtn" href="" download>Download Now</a>
</div>

<style>
.confirmation-container {
  max-width: 480px;
  margin: 20px auto;
  text-align: center;
}
.confirmation-download-link {
  display: inline-block;
  margin-top: 20px;
  padding: 10px 24px;
  background: #28a745;
  color: #fff;
  text-decoration: none;
  border-radius: 4px;
  font-size: 1em;
}
.confirmation-download-link:hover {
  background: #218838;
  color: #fff;
  text-decoration: none;
}
</style>

<script>
window.onload = function() {
  var downloadLinks = {
    "download1": "https://bit.ly/4wdQICw"
  };

  var urlParams = new URLSearchParams(window.location.search);
  var fileKey = urlParams.get('file');

  var downloadUrl = downloadLinks[fileKey] || downloadLinks["download1"];

  document.getElementById('downloadBtn').href = downloadUrl;
  document.getElementById('downloadBtn').click();

  setTimeout(function(){
    window.location.href = "/";
  }, 3000);
};
</script>
