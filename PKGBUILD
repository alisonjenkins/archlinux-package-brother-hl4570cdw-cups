# Contributor: Joris Claassen <info at ebait org>
# Original contributor: Jonas E. Huber <info at hubjo org>
#
# The Brother printer drivers for CUPS are essentially only wrappers around
# the LPR drivers. This package integrates both RPMs and thus facilitates the
# whole process of installing both, LPR drivers and CUPS wrapper.
#
# While this PKGBUILD is specific for the drivers for the HL5040 laser printer,
# it should be easily adjustable for other Brother drivers. When I find the
# time, I'll write a little howto in the wiki or extend the existing one.
#
# Don't hesitate to contact me if you have further questions or suggestions.

pkgname=brother-hl4570cdw-cups
_cups=hl4570cdwcupswrapper
_lpr=hl4570cdwlpr
pkgver=1.1.1
pkgrel=5
pkgdesc="CUPS driver for HL-4570CDW laser printer, including underlying LPR driver."
url="http://solutions.brother.com/linux/en_us/"
arch=('i686' 'x86_64')
license=('GPL2' 'custom:brother')
depends=('cups' 'ghostscript' 'lib32-glibc' 'cpio')
makedepends=('rpmextract')
source=('http://download.brother.com/welcome/dlf005945/hl4570cdwcupswrapper-1.1.1-5.i386.rpm'
		'http://download.brother.com/welcome/dlf005943/hl4570cdwlpr-1.1.1-5.i386.rpm'
		'license.txt'
		'cupswrapper.patch'
		)
md5sums=('c6401a582ee037bc52b5343f6cf7fd7e'
        '07df0c342cc1ed64a9b7cf6e602a89af'
        '73cf49a126378f8dc7e6f3b444a1ff57'
        '7da192c0dddee77937d7ac7ba1b9b695')

build() {

	# Clean up
	rm -rf usr
	rm -rf var

	# Install LPD driver
	# =====================
	# Note that some paths are changed in order to comply with the Arch packaging
	# standard. You have to make these changes also in the patch for the CUPS wrapper!

	# Extracting RPM
	cd $srcdir
	rpmextract.sh "$_lpr-$pkgver-$pkgrel.i386.rpm" || return 1
	rm "$_lpr-$pkgver-$pkgrel.i386.rpm"

	# Move into new directory structure
	mkdir -p usr/share/ || return 1
	mv usr/local/Brother usr/share/brother || return 1
	rm -rf usr/local || return 1

	# Install the *custom* license for the LPR driver.
	install -D -m644 license.txt usr/share/licenses/$pkgname/license.txt || return 1


	# Install CUPS wrapper
	# ====================

	# Extracting RPM
	rpmextract.sh "$_cups-$pkgver-$pkgrel.i386.rpm" || return 1
	rm "$_cups-$pkgver-$pkgrel.i386.rpm"

	# Move brcupsconfig to correct location
	install -D usr/local/Brother/Printer/hl4570cdw/cupswrapper/brcupsconfpt1 usr/share/brother/cupswrapper/brcupsconfpt1

	# Move script to main directory
	mv "usr/local/Brother/Printer/hl4570cdw/cupswrapper/cupswrapperhl4570cdw" . || return 1

    # Move PPD to main directory
    mv "usr/local/Brother/Printer/hl4570cdw/cupswrapper/hl4570cdw.ppd" . || return 1

	rm -rf usr/local || return 1

	# The patch disables everything except the generation of the PPD and filter
	# which are written into separate files in the CWD by executing the script.
	patch < cupswrapper.patch
	./cupswrapperhl4570cdw

	# Install PPD and filter
	install -D -m644 hl4570cdw.ppd usr/share/cups/model/hl4570cdw.ppd || return 1
	install -D -m755 cupswrapperhl4570cdw usr/lib/cups/filter/brlpdwrapperhl4570cdw || return 1

	# Adapt paths where necessary
	sed -i 's|/usr/local/Brother|/usr/share/brother|g' `grep -lr '/usr/local/Brother' ./` || return 1

	# Create and install the file 'brPrintList'. This file must exist and contain the name
	# of the printer in order to make CUPS settings work. Else, settings done in CUPS are
	# not reflected in the file /usr/share/brother/inf/brHL5040rc and thus are not considered
	# by the LPR driver that's doing the actual printing.
	echo "HL4570CDW" > brPrintList
	install -D -m644 brPrintList usr/share/brother/inf/brPrintList

	# Clean up
	rm hl4570cdw.ppd
	rm cupswrapperhl4570cdw
	rm brPrintList

	# Move everything into the pkg directory
	find . | cpio -p -dum $pkgdir || return 1
}
