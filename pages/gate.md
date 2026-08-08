---
layout: page
title: Get Your Free Download
permalink: /gate
subtitle:
show-avatar: false
---

<!--
  GATE — PASSTHROUGH MODE

  No email/subscribe prompt is shown to visitors. This page just forwards
  straight to /confirmation, preserving ?file=, so the visitor sees no
  extra step. /software links here (/gate?file=download1) instead of
  straight to the Drive URL, so the redirect actually gets used.

  To turn this into a real email-gated download later, backed by an
  actual Substack subscription:

    1. Delete (or comment out) the passthrough <script> block below.
    2. Uncomment the gate-container block further down.
    3. Replace YOURPUB.substack.com with your real Substack domain — get
       the exact iframe src from substack.com: Settings -> Embed ->
       Subscribe widget.

  Known limitation if re-enabled: Substack's embeddable widget doesn't
  reliably tell the parent page when a signup succeeds (no dependable
  postMessage/callback event to hook), so the "Continue to your download"
  link would always be visible — an honor-system gate, not an enforced
  one. It gets real subscribers in exchange for the download, it just
  can't hard-block the file behind a verified signup the way a
  server-side check could.
-->

<script>
(function() {
  var urlParams = new URLSearchParams(window.location.search);
  var fileParam = urlParams.get("file") || "download1";
  window.location.replace("/confirmation?file=" + encodeURIComponent(fileParam));
})();
</script>

<!--
<div class="gate-container">
  <p>Subscribe to the FailClosed newsletter to get your download.</p>

  <iframe src="https://YOURPUB.substack.com/embed" width="480" height="320"
    style="border: 1px solid #EEE; background: white;" frameborder="0"
    scrolling="no"></iframe>

  <p><a id="gate-continue" href="#">Continue to your download</a></p>
</div>

<style>
.gate-container {
  max-width: 480px;
  margin: 20px auto;
  text-align: center;
}
.gate-container iframe {
  max-width: 100%;
}
</style>

<script>
(function() {
  // Carry the ?file= param through to /confirmation so it still knows
  // which download to serve, same as the old gate did.
  var urlParams = new URLSearchParams(window.location.search);
  var fileParam = urlParams.get("file") || "download1";
  document.getElementById("gate-continue").href =
    "/confirmation?file=" + encodeURIComponent(fileParam);
})();
</script>
-->
