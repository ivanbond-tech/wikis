<h1>Connecting to a WPA-ENTERPRISE Network using nmcli (Arch Linux)</h1>
<ol>
  <li>
    <h2>Determining wifi-card</h2>
    <code>ip link show</code> -> find <code>wifi-card-name</code> (like wlan0, wlp3s0, etc.)
  </li>
  <li>
    <h2>Creating a new WPA-Enterprise Network Configuration</h2>
    <p>Depending on your organization's network options, the below settings may be slightly different. Generic configuration settings for a university using "eduroam" may look like...</p>
    <ul>
      <li>wifi-sec.key-mgmt wpa-eap</li>
      <li>802-1x.eap peap</li>
      <li>802-1x.phase2-auth mschapv2</li>
      <li>802-1x.identity &ltnetwork-login-username&gt</li>
      <li>802-1x.password &ltnetwork-login-password&gt</li>
      <li>802-1x.system-ca-certs no</li>
    </ul>
    <p>If you already have a previous configuration established, you can edit these settings using the <code>nmcli connection edit &ltconfig-name&gt</code> command and edit the settings with <code>set &ltsetting&gt &ltvalue&gt</code>. Otherwise, you can create a brand new connection configuration with these settings using the following command...</p>
    <code>nmcli connection add type wifi con-name "&ltconfig-name&gt" ifname &ltwifi-card-name&gt ssid "eduroam" wifi-sec.key-mgmt wpa-eap 802-1x.identity "&ltnetwork-login-username&gt" 802-1x.password "&ltnetwork-login-password&gt" 802-1x.system-ca-certs no 802-1x.eap "peap" 802-1x.phase2-auth mschapv2</code>
  </li>
  <li>
    <h2>Activate connection</h2>
    <p>Activate the connection with <code>ncmli device wifi connect &ltconfig-name&gt</code></p>
  </li>
</ol>
