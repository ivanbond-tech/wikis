<h1>Virtualizing Arch Linux on QEMU (x86_64)</h1>
<h2>(For Linux Kernel Development)</h2>

<p>
  This guide was written to help aid in virtualizing Arch Linux using QEMU. Bash commands are   prefaced with $. These steps were executed on a x86_64 Macbook Pro 14,3 running macOS 13.6.4 and zsh 5.9, but should work for most x86 UNIX systems, with small adaptations.
</p>

<ol>
  <li>
    <h4>Downloading Arch .iso files</h4>
    <p>
      The official and most up-to-date .iso and signature files of Arch Linux can be downloaded from
      Pitt's official mirror site at: 
      <a href="https://mirror.cs.pitt.edu/archlinux/iso/latest">
        https://mirror.cs.pitt.edu/archlinux/iso/latest
      </a>.
      <br><br>Download the following files (recommended: using <code>wget</code>):
      <ul>
        <li><code>archlinux-&ltlatest-date&gt-x86_64.iso</code></li> 
        <li><code>archlinux-&ltlatest-date&gt-x86_64.iso.sig</code></li>
        <li><code>b2sums.txt</code></li>
        <li><code>sha256sums.txt</code></li>
      </ul>
    </p>
  </li>
  <li>
    <h4>Verifying Installation</h4>
    <p>
      Once you have all the required files, navigate to the directory containing them on your host 
      machine and verify the signatures using the code below.
    </p>
    <ul>
      <h5>Verify SHA256 Checksum</h5>
      <li>Run: <code>$ shasum -a 256 -c sha256sums.txt</code></li>
      <li>Should output: <code>archlinux-&ltlatest-date&gt-x86_64.iso: OK</code></li>
      <li>Note: the other checksums will fail since we are only using the latest .iso</li>
    </ul>
    <ul>
      <h5>Verify BLAKE2 Checksum</h5>
      <li>Install b2sum (macOS, <a href="https://brew.sh/">homebrew</a>): <code>$ brew install b2sum</code></li>
      <li>Run: <code>$ diff <(b2sum archlinux-*.iso) <(head -n 1 b2sums.txt)</code></li>
      <li>Should output: nothing, meaning no difference in checksums</li>
    </ul>
    <ul>
      <h5>Verify Signature</h5>
      <li>Install gnupg (macOS, <a href="https://brew.sh/">homebrew</a>): <code>$ brew install gnupg</code></li>
      <li>Run: <code>$ gpg --auto-key-locate clear,wkd -v --locate-external-key pierre@archlinux.org</code></li>
      <li>Run: <code>$ gpg --verify archlinux-*.iso.sig</code></li>
      <li>Verify signature in output</li>
    </ul>
  </li>
  <li>
    <h4>Setting up QEMU</h4>
    <p>
      Once you have verified the checksums and signatures of the downloaded files, you can start configuring your
      arch virtual machine with <a href="https://www.qemu.org/">QEMU</a>. Ensure you have QEMU installed on your
      machine before proceeding. It can be easily installed on macOS with homebrew: <code>$ brew install qemu</code>,
      or similarly on Linux with your respective package-manager.
    </p>
    <ul>
      <h5>Creating Virtual Disk</h5>
      <li><code>$ qemu-img create -f qcow2 arch.img 16G</code></li>
      <li>Alternatively: <code>$ qemu-img create arch.hd 16G</code></li>
      <li>
        This will create an 16GB virtual disk named arch.img (or arch.hd) that we will
        use to house our root filesystem from the .iso we downloaded earlier
      </li>
    </ul>
    <ul>
      <h5>Initial Boot</h5>
      <li>        
        This is the qemu-system-x86_64 that I use to run my VM, please change accordingly for your host machine
      </li>
      <li>Specifically: -m, -cpu, -machine, -vga</li>
      <li>
        <code>qemu-system-x86_64 -m 4G -cpu host -smp 4 -machine type=q35,accel=hvf -rtc base=localtime -hda arch.hd -cdrom archlinux-2024.02.01-x86_64.iso -net nic -net user,hostfwd=tcp::10022-:22 -vga virtio -netdev vmnet-shared,id=net0 -device virtio-net,netdev=net0</code>
      </li>
    </ul>
  </li>
  <li>
    <h4>Arch Linux Installation</h4>
    <p>
      At first boot from QEMU, we must partition our virtual disk, mount it, and install Arch from the 
      provided live image (Arch Linux install medium (x86_64, BIOS)).
    </p>
    <ol>
      <h5>First steps</h5>
      <ul>
        <li>Set password for root: <code>$ passwd</code></li>
        <li>Verify networking is properly configured: <code>$ ping pitt.edu</code></li>
      </ul>
      <h5>Partitioning Virtual Disk</h5>
      <ul>
        <li>Determine disk to partition with <code>$ fdisk -l</code>, should be /dev/sda</li>
        <li>Enter 'cfdisk' with <code>$ cfdisk /dev/sda</code> and parition in DOS format</li>
        <li>Create 256M bootable primary partition for /boot</li>
        <li>Create 16G-256M primary partition (15.7G) for /root</li>
        <li>Write your results and exit 'cfdisk'</li>
      </ul>
      <h5>Creating Filesystems</h5>
      <ul>
        <li>Now add filesystems to the partitions with: <code>$ mkfs.ext4 /dev/sda1; mkfs.ext4 /dev/sda2</code></li>
        <li>Mount the root partition: <code>$ mount /dev/sda2 /mnt</code></li>
        <li>Make boot directory and mount it: <code>$ mkdir /mnt/boot ; mount /dev/sda1 /mnt/boot</code></li>
        <li>Verify changes with <code>$ lsblk</code></li>
      </ul>
      <h5>Pac-straping Kernel, Changing into installed system</h5>
      <ul>
        <li>Install Arch: <code>$ pacstrap /mnt base base-devel linux linux-firmware vim</code></li>
        <li>Create fstab: <code>$ genfstab -U /mnt >> /mnt/etc/fstab</code></li>
        <li>Change root: <code>$ arch-chroot /mnt /bin/bash</code></li>
      </ul>
      <h5>Installing Dependencies, Enabling Networking</h5>
      <ul>
        <li>Install required packages: <code>$ pacman -S networkmanager grub openssh xmlto inetutils wget bc</code></li>
        <li>Enable networking: <code>$ systemctl enable NetworkManager</code></li>
        <li><code>$ vim /etc/ssh/sshd_config</code>, and uncomment/edit these lines:
          <ul>
            <li><code>PermitRootLogin yes</code></li>
            <li><code>PasswordAuthentication yes</code></li>
            <li><code>UsePAM no</code></li>
          </ul>
        </li>
      </ul>
      <h5>Configuring GRUB</h5>
      <ul>
        <li><code>$ grub-install /dev/sda</code></li>
        <li><code>$ grub-mkconfig -o /boot/grub/grub.cfg</code></li>
      </ul>
      <h5>Configuring Locale</h5>
      <ul>
        <li><code>$ vim /etc/locale.gen</code>, Uncomment 'en_US.UTF-8'</li>
        <li><code>$ locale-gen</code></li>
        <li><code>$ vim /etc/locale.conf</code>: write LANG=en_US.UTF-8</li>
      </ul>
      <h5>Final Configurations</h5>
      <ul>
        <li>Set hostname with: <code>$ vim /etc/hostname</code></li>
        <li>Link timezone: <code>$ ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime</code></li>
      </ul>
      <h5>Exiting, Rebooting</h5>
      <p>All done! Exit with: <code>$ exit ; umount -R /mnt</code>. You can now safely reboot</p>
    </ol>
  </li>
  <li>
    <h4>Additional Configurations for SSH</h4>
    <p>After booting the VM, ssh will not directly start because the root user must be logged
    in for our etc/ssh/sshd_config to take effect. We can mitigate this by creating a .profile
    in our $HOME (/root) directory and pasting in the following:
    </p>
    <code># include .bashrc if it exists</code><br>
    <code>if [ -f "$HOME/.bashrc" ]; then</code><br>
    <code>   . "$HOME/.bashrc"</code><br>
    <code>fi</code><br>
    <code># restart sshd to allow ssh connections to VM</code><br>
    <code>systemctl restart sshd</code><br>
    <p>Now when you start QEMU with the <code>qemu-system-x86_64</code> command from earlier and
    login with the root user, you can ssh to the VM from your terminal with: </p>
    <code>$ ssh -p 10022 root@localhost</code>
  </li>
  <li>
    <h4>Kernel Configuration and Compilation</h4>
    <ul>
      <h5>Downloading Latest Linux Kernel</h5>
      <li>Download latest build from: 
        <a href="https://mirrors.edge.kernel.org/pub/linux/kernel/">www.kernel.org</a>
         using <code>wget</code>
      </li>
      <li>Verify signatures as in step 2</li>
      <li>Decompress: <code>$ tar xJf linux-&ltlatest&gt.tar.xz</code></li>
      <h5>Making Necessary Configuration Changes</h5>
      <li>In linux-&ltlatest&gt directory, <code>$ make clean && make mrproper</code></li>
      <li>Copy Arch Default configuration: <code>$ zcat /proc/config.gz > .config</code></li>
      <li>Copy VM's local drivers to config: <code>$ make localmodconfig</code></li>
      <li>Copy KVM options to config: <code>$ make kvm_guest.config</code></li>
      <h5>Manual Configurations, <code>$ vim .config</code></h5>
      <li>Append release: set <code>CONFIG_LOCALVERSION="-arch-1980"</code></li>
      <li>Use LZ4 compression: set <code>CONFIG_KERNEL_LZ4=y</code></li>
      <h5>Compilation</h5>
      <li><code>$ make -j&ltn&gt</code>, where n is the # of cores available in your VM</li>
      <li>Compile modules: <code>$ make modules</code></li>
      <li>Install modules: <code>$ make modules_install</code></li>
      <h5>Post Compilation</h5>
      <li>Copy created Image: <code>$ cp -v arch/x86/boot/bzImage /boot/vmlinuz-linux-arch-1980</code></li>
      <li>Copy initramfs preset: <code>$ cp /etc/mkinitcpio.d/linux.preset /etc/mkinitcpio.d/linux-arch-1980.preset</code></li>
      <li><code>$ vim /etc/mkinitcpio.d/linux-arch-1980.preset</code></li>
      <ul>
        <li><code>ALL_kver="/boot/vmlinuz-linux-arch-1980</code></li>
        <li><code>default_image="/boot/initramfs-linux-arch-1980.img"</code></li>
        <li><code>fallback_image="/boot/initramfs-linux-arch-1980-fallback.img"</code></li>
        <li>save and exit</li>
      </ul>
      <li>Generate initramfs: <code>$ mkinitcpio -p linux-arch-1980</code></li>
      <li>Copy System.map: <code>$ cp System.map /boot/System.map-linux-arch-1980</code></li>
      <li>Symlink: <code>$ ln -sf /boot/System.map-linux-arch-1980 /boot/System.map</code></li>
      <h5>Configure Bootloader</h5>
      <li><code>$ grub-mkconfig -o /boot/grub/grub.cfg</code></li>
      <li>Reboot and launch built kernel image</li>
    </ul>
  </li>
</ol>
