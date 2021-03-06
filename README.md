Debian Squeeze Kernel for Linkstation Pro Duo / LS-WXL
======================================================

For building linux-2.6.32 Debian Squeeze Kernel Deb-Package for Linkstation Orion5x/Kirkwood.
Original source was migrated from Debian SVN for easy maintain.
Currently confirmed working on LS-ProDuo & LS-WXL.

----
Make kernel package (linux-image-2.6, etc)
----

Here's the cross build script, confirmed working on Wheezy amd64 box.
Step0, build tool preparation (need squeeze version due to gcc version requirement):

	cat << EOT >> /etc/apt/sources.list
	deb http://www.emdebian.org/debian wheezy main
	deb http://www.emdebian.org/debian squeeze main
	deb http://ftp.riken.jp/Linux/debian/debian squeeze main non-free contrib
	deb http://ftp.jp.debian.org/debian squeeze main non-free contrib
	EOT
	aptitude update; aptitude install fakeroot debhelper devscripts g++-4.3-arm-linux-gnueabi xmlto
	wget http://ftp.jp.debian.org/debian/pool/main/l/linux-2.6/linux-2.6_2.6.32.orig.tar.gz

Step1, clone repo:

	mkdir linux-2.6; cd $_; git init
	git remote add origin https://github.com/rogeryan0/linux-2.6_squeeze-security
	git fetch
	git checkout -b master origin/linkstation-kernel

Step2, package building:

	export $(dpkg-architecture -aarmel)
	fakeroot make -f debian/rules clean 2>&1 | tee ../log0_clean.log
	mv .git ../linux-2.6.git; fakeroot make -f debian/rules orig 2>&1 | tee ../log1_orig.log; mv ../linux-2.6.git .git
	fakeroot make -f debian/rules source 2>&1 | tee ../log2_source.log
	fakeroot make -f debian/rules.gen setup_armel_none_orion5x 2>&1 | tee ../log3_setup.log
	fakeroot make -f debian/rules.gen binary-arch_armel_none_orion5x binary-indep DEBIAN_KERNEL_JOBS=4 2>&1 | tee ../log4_build.log

Or for Kirkwood, final two lines need to be adjusted as follows:

	fakeroot make -f debian/rules.gen setup_armel_none_kirkwood 2>&1 | tee ../log3_setup.log
	fakeroot make -f debian/rules.gen binary-arch_armel_none_kirkwood binary-indep DEBIAN_KERNEL_JOBS=4 2>&1 | tee ../log4_build.log


----
Install the kernel package on Linkstation box
----

Basiclly on a Debian system, "dpkg -i linux-image-xxx.deb" should get the kernel installed, but Marvell Orion5x board need a device ID at the head of the kernel image. And both kernel and initrd need to be repacked by mkimage. So here's the script:

	ver=2.6.32-5-orion5x
	cd /boot
	devio > /tmp/foo 'wl 0xe3a01c07,4' 'wl 0xe3811027,4'
	cat foo /boot/vmlinuz-$ver > /tmp/zImage
	mkimage -A arm -O linux -T kernel -C none -a 0x00008000 -e 0x00008000 -n $ver -d /tmp/zImage vmlinuz.uimg-$ver
	rm /tmp/foo /tmp/zImage
	mkimage -A arm -O linux -T ramdisk -C none -a 0x00008000 -e 0x00008000 -n $ver -d initrd.img-$ver initrd.uimg-$ver
	ln -sf vmlinuz.uimg-$ver uImage.buffalo
	ln -sf initrd.uimg-$ver initrd.buffalo
	cd -

For newly Marvel Kirkwood board, it gets much simpler:

	ver=2.6.32-5-kirkwood
	cd /boot
	mkimage -A arm -O linux -T kernel -C none -a 0x00008000 -e 0x00008000 -n $ver -d vmlinuz-$ver vmlinuz.uimg-$ver
	mkimage -A arm -O linux -T ramdisk -C none -a 0x00008000 -e 0x00008000 -n $ver -d initrd.img-$ver initrd.uimg-$ver
	ln -sf vmlinuz.uimg-$ver uImage.buffalo
	ln -sf initrd.uimg-$ver initrd.buffalo
	cd -


----
Status
----

Confirmed to be working on the model below:

 - LS-WXL (2.6.32-48squeeze8)


----
Memo
----

The original source is migrated from debian SVN, but only keep the history after 2.6.32-48(r19819):
	svn://svn.debian.org/kernel/dists/squeeze-security/linux-2.6/debian
and the original source is saved, untouched, on "debian-svn" branch.


----
Credit
----

- http://buffalo.nas-central.org
- http://download.prodigy7.de/files/packages/linkstation/kernel/lsproduo-vanilla/
- http://buffalo-nas-devices.googlecode.com
