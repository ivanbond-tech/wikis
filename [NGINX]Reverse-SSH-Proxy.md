<h1>How-To Setup a SSH Reverse Proxy using NGINX</h1>
<p>If you are like me and have many computers/laptops/servers that you have on your local network that you wish to have SSH access to you can follow this guide to setup a home lab with a reverse proxy server that allows SSH connections using subdomains.</p>
<p>A great guide for this, including configuring other services such as HTTP, FTP, etc. within the reverse proxy is available <a href='https://www.howtoforge.com/reverse-proxy-for-https-ssh-and-mysql-mariadb-using-nginx/#configuring-ssh-mysqlmariadb-reverse-proxies-stream'>here.</a></p>

<h3>Example Network Configuration</h3>
<table>
  <tr>
    <th>Device</th>
    <th>Network Service</th>
    <th>Local IP Address</th>
    <th>DNS (A) Record</th>
    <th>SSH Port #</th>
    <th>Other Ports</th>
  </tr>
  <tr>
    <td>Raspberry Pi Zero W</td>
    <td>Reverse Proxy Server</td>
    <td>192.168.0.10</td>
    <td>example.com</td>
    <td>22</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td>Arch Linux Server</td>
    <td>N/A</td>
    <td>192.168.0.11</td>
    <td>arch.example.com</td>
    <td>2200</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td>Gentoo Linux Server</td>
    <td>N/A</td>
    <td>192.168.0.12</td>
    <td>gentoo.example.com</td>
    <td>2201</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td>Ubuntu Linux Server</td>
    <td>FTP Server</td>
    <td>192.168.0.13</td>
    <td>ubuntu.example.com</td>
    <td>2202</td>
    <td>21</td>
  </tr>
  <tr>
    <td>OpenBSD Server</td>
    <td>HTTP Webserver</td>
    <td>192.168.0.14</td>
    <td>openbsd.example.com</td>
    <td>2203</td>
    <td>80,443</td>
  </tr>
</table>

<h3>Reverse Proxy Host Setup</h3>
<p>In the example network configuration our reverse proxy server would be running raspbian-OS/debian so the corresponding commands would be applicable using the host's package manager - apt.</p>
<ul>
  <h4>Update System/Install Required Packages</h4>
  <li><code>sudo apt update ; sudo apt upgrade -y</code></li>
  <li><code>sudo apt install nginx ufw libnginx-mod-stream</code></li>
  <h4>Firewall (UFW) Setup</h4>
  <li><code>sudo ufw allow ssh</code></li>
  <li><code>sudo ufw allow 2200:2204/tcp</code> (corresponding to SSH Port # column from network config above)</li>
  <li><code>sudo ufw allow 2200:2204/udp</code></li>
  <li>additional firewall configurations here...</li>
  <li><code>sudo ufw enable</code></li>
  <h4>Edit SSHD Configuration</h4>
  <li><code>sudo vim /etc/ssh/sshd_config</code></li>
  <ul>
    <li><code>Port 22</code></li>
    <li><code>ListenAddress 0.0.0.0</code></li>
    <li><code>LoginGraceTime 2m</code></li>
    <li><code>PermitRootLogin no</code></li>
    <li><code>StrictModes yes</code></li>
    <li><code>MaxAuthTries 6</code></li>
    <li><code>MaxSessions 10</code></li>
    <li>If using password authentication...</li>
    <li><code>PasswordAuthentication yes</code></li>
    <li><code>PermitEmptyPasswords no</code></li>
    <li>Additional optional settings...</li>
    <li><code>PermitTTY yes</code></li>
    <li><code>PrintMotd yes</code></li>
    <li><code>PrintLastLog yes</code></li>
  </ul>
  <h4>Edit Host Discovery File /etc/hosts</h4>
  <li><code>sudo vim /etc/hosts</code></li>
  <ul>
    <li><code>192.168.0.11  arch.example.com</code></li>
    <li><code>192.168.0.12  gentoo.example.com</code></li>
    <li><code>192.168.0.13  ubuntu.example.com</code></li>
    <li><code>192.168.0.14  openbsd.example.com</code></li>
  </ul>
  <h4>NGINX Configuration</h4>
  <p>Depending on the complexity of your reverse proxy, these commands can be adapted for the specific service needed, but this example will only show the SSH configurations (see guide linked above).</p>
  <ul>
    <li><code>cd /etc/nginx && mkdir rproxy</code></li>
    <li><code>mkdir rproxy/stream && mkdir rproxy/stream/available && mkdir rproxy/stream/enabled</code></li>
    <li><code>sudo vim rproxy/stream/available/ssh.conf</code></li>
    <ul>
      <br>
      <li># For each of the server hosts in the table above create a corresponding 'upstream' and 'server' block...</li>
      <li><code>upstream arch-ssh { server 192.168.0.11:2200; }</code></li>
      <li><code>server { listen 2200; proxy_pass arch-ssh; }</code></li>
      <li>...</li>
    </ul>
    <br>
    <li><code>sudo vim /etc/nginx/nginx.conf</code></li>
    <ul>
      <br>
      <li>comment out sites-enabled include in 'http' block...</li>
      <li><code># include /etc/nginx/sites-enabled/*;</code></li>
      <li>Create a new 'stream' block after the 'http' block...</li>
      <li><code>stream { include /etc/nginx/rproxy/stream/enabled/*.conf; }</code></li>
    </ul>
    <br>
    <li><code>sudo ln -s /etc/nginx/rproxy/stream/available/*.conf /etc/nginx/rproxy/stream/enabled; }</code></li>
    <li><code>sudo nginx -t</code></li>
    <li><code>sudo systemctl restart sshd</code></li>
  </ul>
</ul>

<h3>Server Host Setup</h3>
<p>There isn't much needed on the host side to setup, apart from copying the SSHD Configuration steps from the Reverse Proxy Host Setup, but changing the <code>Port</code> to the SSH Port # from the Example Network table. If the server host has a firewall enabled, ensure that it is configured properly to work with the reverse proxy.</p>
<p>Additionally, if the <code>sshd.service</code> is not enabled on startup, it can be set using the following commands...</p>
<ul>
  <li>systemd: <code>sudo systemctl enable sshd && sudo systemctl start sshd</code></li>
  <li>openrc: <code>sudo rc-update add sshd default && sudo rc-service sshd start</code></li>
</ul>

<h3>Final Configurations (Port-Forwarding, DNS)</h3>
<p>Ensure that your domain (example.com) and all desired subdomains are pointing to your local network's public IP address. This is usually done by creating an 'A RECORD' on your DNS provider's web portal. Lastly, ensure that all required ports are correctly forwarded in your router settings, like port 22 (SSH) for TCP/UDP on your reverse proxy host.</p>

<h3>Testing SSH</h3>
<p>Assuming all went well, you can now access your server hosts using the following syntax...</p>
<ul>
  <li><code>ssh -p &ltserver-host-ssh-port&gt &ltuser&gt@&ltsubdomain&gt</code></li>
  <li>example: <code>ssh -p 2200 user@arch.example.com</code></li>
</ul>
