# Reference: <https://postmarketos.org/vendorkernel>
# Kernel config based on: arch/arm64/configs/(CHANGEME!)

pkgname=linux-nokia-wasp
pkgver=4.19.127
pkgrel=0
pkgdesc="Nokia 2.2 kernel fork"
arch="aarch64"
_carch="arm64"
_flavor="nokia-wasp"
url="https://kernel.org"
license="GPL-2.0-only"
options="!strip !check !tracedeps pmb:cross-native"
makedepends="
	bash
	bc
	bison
	cpio
	devicepkg-dev
	findutils
	flex
	openssl-dev
	perl
	python3
	xz
"

# Source
_repository="android_kernel_nokia_mt6761"
_commit="8cfc500010d0daa96032ddd86e0c3ebe9aacb8d7"
_config="config-$_flavor.$arch"
source="
	$pkgname-$_commit.tar.gz::https://github.com/nokia-mt6761-devs/$_repository/archive/$_commit.tar.gz
	$_config
	edit-selinux-generated-headers.patch
	stop-inlining-uclamp_se_set-blk_crypto_flock-and-ksm.patch
	use_system_cpio.patch
"
builddir="$srcdir/$_repository-$_commit"
_outdir="out"

prepare() {
	default_prepare
	REPLACE_GCCH=0 . downstreamkernel_prepare
}

build() {
	unset LDFLAGS
	make O="$_outdir" ARCH="$_carch" CC="${CC:-gcc}" \
		KBUILD_BUILD_VERSION="$((pkgrel + 1 ))-postmarketOS"
}

package() {
	# Gzipping the kernel, not as an unpacker program, but as a real gzip archive, is required for it to boot.
	# And because of that, we need to remove the uncompressed kernel and replace it with the compressed kernel.
	# If you do not do that, you will get a bootloader error like this while trying to boot it:
	# [5349] panic (caller 0x48037667): [5349] decompress kernel image fail!!!

	if [ -f "$_outdir"/arch/arm64/boot/Image ]
	then
		rm "$_outdir"/arch/arm64/boot/Image
	fi

	mv "$_outdir"/arch/arm64/boot/Image.gz "$_outdir"/arch/arm64/boot/Image

	downstreamkernel_package "$builddir" "$pkgdir" "$_carch" \
		"$_flavor" "$_outdir"

	make dtbs_install O="$_outdir" ARCH="$_carch" \
		INSTALL_DTBS_PATH="$pkgdir"/boot/dtbs

	# We also need to convert the kernel DTB into a proper android dtb image with the 64-byte header

	mv "$pkgdir"/boot/dtbs/mediatek/mt6761.dtb "$pkgdir"/boot/dtbs/mediatek/mt6761.dtb.bak
	"$_outdir"/../scripts/mkdtboimg.py create "$pkgdir"/boot/dtbs/mediatek/mt6761.dtb "$pkgdir"/boot/dtbs/mediatek/mt6761.dtb.bak
	rm "$pkgdir"/boot/dtbs/mediatek/mt6761.dtb.bak
}

sha512sums="
277f21461f9b9047975e04bf4db5b56f251c1069affbfcae3a23ff68c4fcca70ce016aacb33969fb32f275fceea962eaf7a5c626318b8079ab50687fd2531bf2  linux-nokia-wasp-8cfc500010d0daa96032ddd86e0c3ebe9aacb8d7.tar.gz
e97c2067343736ad417830233167a5ba252c379b655d27b38c3d9b614f3659a458adcf3fa08e78c5d3686bf46a4faef5e7455acf41ca7cdedc7066c96dc50d6b  config-nokia-wasp.aarch64
fc8f36d3106ad6306d4f07db83d26f0d57e92c29440a6d25271295a894017b82f0f1da2568b399d3bdc13677bb894ba635e013c39f4a9466a28851e7c0e21c48  edit-selinux-generated-headers.patch
e8c4302e671ad187348bd54eedb81cf3a0c5d368066ab872295936c102ac1d6024ed5fbbea5386b04270c75bb0948cfc51d3b2febb4a4daa5e051643db67b909  stop-inlining-uclamp_se_set-blk_crypto_flock-and-ksm.patch
48c0728018a58229e2af7756ce5f1a8b98d38d36857539529a1d0d834a96a0edb95e46390d8a947a61e957811259a11cdc21b3b0d117da254683f4a393080048  use_system_cpio.patch
"
