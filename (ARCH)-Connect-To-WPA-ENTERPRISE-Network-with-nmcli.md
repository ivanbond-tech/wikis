<h1>Connecting to a WPA-ENTERPRISE Network using nmcli (Arch Linux)</h1>
<ol>
  <li>
    <h2>Determining wifi-card</h2>
    <code>ip link show</code> -> get wifi-card name (like wlan0, wlp3s0, etc.)
  </li>
  <li>
    <h2>Creating a new WPA-Enterprise Network Configuration</h2>
    <p>Depending on your organization's network options, the below settings may be slightly different. Generic configuration settings for a university using "eduroam" may look like...</p>
    <code>ncmli connection add type wifi connection-name</code>
  </li>
</ol>
