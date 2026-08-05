---
layout: page
title: Get Your Free Download
permalink: /gate
subtitle:
show-avatar: false
---

<!--
  GATE — CURRENTLY DISABLED

  Nothing links to this page right now. /software links straight to the
  Google Drive file (see pages/software.md), bypassing this page and
  /confirmation entirely. The old version of this page only wrote the
  email to localStorage and never sent it anywhere, so it was pulled —
  it added a click for the visitor and captured nothing real.

  To re-enable a real email-gated download, backed by an actual Substack
  subscription instead of the old fake localStorage capture:

    1. Uncomment the block below.
    2. Replace YOURPUB.substack.com with your real Substack domain — get
       the exact iframe src from substack.com: Settings -> Embed ->
       Subscribe widget.
    3. In pages/software.md, change the Email Security link back to
       /gate?file=download1 instead of the direct Drive URL.
    4. In pages/confirmation.md, downloadLinks already has "download1"
       mapped to the Drive URL, so no change needed there unless you add
       more gated files.

  Known limitation: Substack's embeddable widget doesn't reliably tell the
  parent page when a signup succeeds (no dependable postMessage/callback
  event to hook), so the "Continue to your download" link below is always
  visible — this is an honor-system gate, not an enforced one. It gets you
  real subscribers in exchange for the download, it just can't hard-block
  the file behind a verified signup the way a server-side check could.
-->

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
