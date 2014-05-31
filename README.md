debian squeeze linux kernel (2.6.32-48squeeze6) for Linkstation Pro Duo
=======================================================================

For building linux-2.6.32 squeeze kernel for Linkstation ProDuo. Exported from Debian SVN.

----
Make kernel package (linux-image-2.6, etc)
----

Here's the cross build script, confirmed working on wheezy amd64 box.

	wget http://security.debian.org/debian-security/pool/updates/main/l/linux-2.6/linux-2.6_2.6.32.orig.tar.gz
	mkdir linux-2.6; cd $_; git init
	git remote add origin https://github.com/rogeryan0/linux-2.6_squeeze-security
	git fetch
	git checkout -b master origin/linkstation-produo_stable
	export $(dpkg-architecture -aarmel)
	fakeroot make -f debian/rules clean 2>&1 | tee ../log0_clean.log
	mv .git ../linux-2.6.git; fakeroot make -f debian/rules orig 2>&1 | tee ../log1_orig.log; mv ../linux-2.6.git .git
	fakeroot make -f debian/rules source 2>&1 | tee ../log2_source.log
	fakeroot make -f debian/rules.gen setup_armel_none_orion5x 2>&1 | tee ../log3_setup.log
	fakeroot make -f debian/rules.gen binary-arch_armel_none_orion5x binary-indep DEBIAN_KERNEL_JOBS=4 2>&1 | tee ../log4_build.log


----
Install the kernel package on Linkstation box
----

Basiclly on a Debian system, "dpkg -i linux-image-xxx.deb" should get the kernel installed, but Marvell ARM board need a device ID at the head of the kernel image. And both kernel and initrd need to be repacked by mkimage. So here's the script:

	ver=2.6.32-5-orion5x
	devio > foo 'wl 0xe3a01c07,4' 'wl 0xe3811027,4'
	cat foo /boot/vmlinuz-$ver > zImage
	mkimage -A arm -O linux -T kernel -C none -a 0x00008000 -e 0x00008000 -n $ver -d zImage /boot/vmlinuz.uimg-$ver
	rm foo zImage
	mkimage -A arm -O linux -T ramdisk -C none -a 0x00008000 -e 0x00008000 -n $ver -d /boot/initrd.img-$ver /boot/initrd.uimg-$ver
	ln -sf /boot/vmlinuz.uimg-$ver /boot/uImage.buffalo
	ln -sf /boot/initrd.uimg-$ver /boot/initrd.buffalo


----
Memo
----

The original source is migrated from debian SVNi, but only keep the history after 2.6.32-48(r19819):
	svn://svn.debian.org/kernel/dists/squeeze-security/linux-2.6/debian
and the original source is saved, untouched, on "debian-svn" branch.


----
Credit
----

- http://buffalo.nas-central.org
- http://download.prodigy7.de/files/packages/linkstation/kernel/lsproduo-vanilla/
- http://buffalo-nas-devices.googlecode.com
