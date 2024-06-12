<h2>Installing Gentoo</h2>
<h3>Gentoo Minimal USB Setup</h3>
<h4>Wireless Adapter Setup</h4>
<ul>
  <li>Find adapter name: <code>ip a</code></li>
  <li><code>net-setup &ltwifi-adapter&gt</code></li>
  <li>Go through ncurses menu and setup WiFi Network</li>
  <li>Bring adapter online: <code>ifconfig &ltwifi-adapter&gt up</code></li>
  <li>Test: <code>ping gentoo.org</code></li>
</ul>
<h4>Disk Partitioning</h4>
<ul>
  <li>Identify disk to partition with <code>lsblk</code></li>
  <li>Erase stored filesystem: <code>wipefs -a /dev/&ltdisk&gt</code></li>
  <li>Enter gparted: <code>parted -a optimal &ltdisk&gt</code></li>
  <ul>
    <li><code>mklabel gpt</code></li>
    <li><code>mkpart primary fat32 1MiB 3MiB</code></li>
    <li><code>name 1 grub</code></li>
    <li><code>set 1 bios_grub on</code></li>
    <li><code>mkpart primary fat32 3MiB 131MiB</code></li>
    <li><code>name 2 boot</code></li>
    <li><code>mkpart primary linux-swap 131MiB 4227MiB</code></li>
    <li><code>name 3 swap</code></li>
    <li><code>mkpart primary ext4 4227MiB -1</code></li>
    <li><code>name 4 rootfs</code></li>
    <li><code>print</code></li>
  </ul>
</ul>
<h4>Creating Filesystems</h4>
<ul>
  <li><code>mkfs.fat -F 32 /dev/&ltdisk-p1&gt</code></li>
  <li><code>mkswap /dev/&ltdisk-p2&gt</code></li>
  <li><code>swapon /dev/&ltdisk-p2&gt</code></li>
  <li><code>mkfs.ext4 /dev/&ltdisk-p3&gt</code></li>
  <li><code>mkfs.ext4 /dev/&ltdisk-p4&gt</code></li>
</ul>
<h4>Stage 3</h4>
<ul>
  <li><code>mount /dev/&ltdisk-3&gt /mnt/gentoo</code></li>
  <h5>Downloading Stage3</h5>
  <ul>
    <li>Download stage3 from <a href="https://www.gentoo.org/downloads/#other-arches">gentoo</a> using <code>wget</code> or <code>links</code></li>
    <li>For example: <code>wget https://distfiles.gentoo.org/releases/amd64/autobuilds/20240609T164903Z/stage3-amd64-openrc-20240609T164903Z.tar.xz</code></li>
    <li>Also download the .CONTENTS.gz, .DIGESTS, .sha256 files</li>
  </ul>
  <h5>Verify Download</h5>
  <ul>
    <li><code>openssl dgst -r -sha512 stage3-amd-64-&ltrelease&gt-&ltinit&gt.tar.xz</code></li>
    <li><code>openssl dgst -r -blake2b512 stage3-amd-64-&ltrelease&gt-&ltinit&gt.tar.xz</code></li>
    <li><code>sha256sum --check stage3-amd-64-&ltrelease&gt-&ltinit&gt.tar.xz.sha256</code></li>
    <li><code>gpg --import /usr/share/openpgp-keys/gentoo-release.asc</code></li>
    <li><code>gpg --verify stage3-amd64-&ltrelease&gt-&ltinit&gt.tar.xz.*</code></li>
  </ul>
  <h5>Unpack the stage file</h5>
  <ul>
    <li><code>tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner</code></li>
    <li>It is recommend to configure portage at this point...</li>
    <li><code>scp ivan@gentoo-box:/etc/portage/make.conf etc/portage/</code></li>
    <li><code>mkdir --parents /mnt/gentoo/etc/portage/repos.conf</code></li>
    <li><code>cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf</code></li>
  </ul>
</ul>
<h4>Chroot</h4>
<ul>
  <li><code>cp --dereference /etc/resolv.conf /mnt/gentoo/etc</code></li>
  <li>If using Gentoo install media: <code>arch-chroot /mnt/gentoo</code>, else: </li>
  <ul>
    <li><code>mount --types proc /proc /mnt/gentoo/proc</code></li>
    <li><code>mount --rbind /sys /mnt/gentoo/sys</code></li>
    <li><code>mount --make-rslave /mnt/gentoo/sys</code></li>
    <li><code>mount --rbind /dev /mnt/gentoo/dev</code></li>
    <li><code>mount --make-rslave /mnt/gentoo/dev</code></li>
    <li><code>mount --bind /run /mnt/gentoo/run</code></li>
    <li><code>mount --make-slave /mnt/gentoo/run</code></li>
    <li><code>chroot /mnt/gentoo /bin/bash</code></li>
  </ul>
  <li><code>source /etc/profile</code></li>
  <li><code>export PS1="(chroot) ${PS1}"</code></li>
  <li><code>mount /dev/&ltdisk-1&gt /boot</code></li>
</ul>
<h4>System Configuration</h4>
<ul>
  <li><code>emerge-webrsync</code></li>
  <li><code>eselect profile list</code>, and select profile with <code>eselect profile set &ltprofile-id&gt</code></li>
  <li><code>emerge --verbose --update --deep --newuse @world</code></li>
  <li><code>emerge --depclean</code></li>
  <li><code>echo "America/New_York" > /etc/timezone</code></li>
  <li><code>emerge --config sys-libs/timezone-data</code></li>
  <li><code>vim /etc/locale.gen</code> and uncomment both 'en_US' options</li>
  <li><code>locale-gen</code></li>
  <li><code>eselect locale set &ltlocale-id-for-en_US&gt</code></li>
  <li><code>env-update && source /etc/profile</code></li>
</ul>
<h4>Kernel Setup</h4>
<h5>Kernel Installation</h5>
<ul>
  <li><code>emerge sys-kernel/gentoo-sources genkernel</code></li>
  <li>NOTE: If you receive a collision with /bin/cpio, set `-collision-protect` in /etc/portage/make.conf</li>
  <li><code>emerge sys-apps/pciutils app-arch/lzop app-arch/lz4</code></li>
  <li><code>eselect kernel set &ltkernel-id&gt</code></li>
</ul>
<h5>Kernel Configuration</h5>
<ul>
  <li><code>cd /usr/src/linux</code></li>
  <li><code>make menuconfig</code></li>
</ul>
