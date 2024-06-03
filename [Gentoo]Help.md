### Kernel Rebuild

- Get new kernel version (gentoo-sources)
  - `emerge --ask --update --deep --with-bdeps=y --newuse @world`
- Set new kernel version (set x)
  - `eselect kernel list`
  - `cd /usr/src/linux`
- Recompile kernel
  - `make -j4`
  - `make modules_install`
  - `make install`
- Install kernel, build initramfs
  - `cp -v arch/x86/boot/bzImage /boot/vmlinuz-<release>-gentoo`
  - `genkernel --install --kernel-config=/usr/src/linux/.config initramfs`
- Update bootloader
  - `grub-install --target=x86_64-efi --efi-directory=/boot`
  - `grub-mkconfig -o /boot/grub/grub.cfg`

### Updating System Packages
- Sync portage tree
  - `emerge --sync`
- Update portage package manager before using
  - `emerge --oneshot sys-apps/portage`
- Update all packages
  - `emerge --update --deep --newuse @world`
  - or `emerge -avuDN @world` for shorthand
- Update config (dot) files
  - `sudo etc-update`
- Cleanup
  - `emerge --depclean`   
