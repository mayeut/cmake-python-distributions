ARG BASE_IMAGE

FROM --platform=${BUILDPLATFORM} ghcr.io/mayeut/static-clang:edge AS static_clang_bin

FROM ${BASE_IMAGE}
COPY --from=static_clang_bin /opt/clang /opt/clang
RUN <<EOFD
#!/bin/bash
set -euxo pipefail

case "${AUDITWHEEL_PLAT}" in
	manylinux*_armv7l) TARGET_TRIPLET=arm-unknown-linux-gnueabihf;;
	musllinux*_armv7l) TARGET_TRIPLET=armv7-alpine-linux-musleabihf;;
	manylinux*_ppc64le) TARGET_TRIPLET=powerpc64le-none-linux-gnu;;
	musllinux*_ppc64le) TARGET_TRIPLET=powerpc64le-alpine-linux-musl;;
	musllinux*_i686) TARGET_TRIPLET=i586-alpine-linux-musl;;
	manylinux*_riscv64) TARGET_TRIPLET=riscv64-linux-gnu;;
	manylinux*) TARGET_TRIPLET=${AUDITWHEEL_ARCH}-none-linux-gnu;;
	musllinux*) TARGET_TRIPLET=${AUDITWHEEL_ARCH}-alpine-linux-musl;;
esac
case "${AUDITWHEEL_ARCH}" in
	riscv64) M_ARCH="-march=rv64gc";;
	x86_64) M_ARCH="-march=x86-64";;
	armv7l) M_ARCH="-march=armv7a";;
	i686) M_ARCH="-march=i686";;
esac

TOOLCHAIN_PATH=/opt/clang
# hack until we can use CMake 3.29+
ln -s lld "${TOOLCHAIN_PATH}/bin/ld"

cat<<EOF >"${TOOLCHAIN_PATH}/bin/${AUDITWHEEL_PLAT}.cfg"
	-target ${TARGET_TRIPLET}
	${M_ARCH:-}
	--gcc-toolchain=${DEVTOOLSET_ROOTPATH:-}/usr
EOF

cat<<EOF >"${TOOLCHAIN_PATH}/bin/clang.cfg"
	@${AUDITWHEEL_PLAT}.cfg
	-fuse-ld=lld
EOF
cat<<EOF >"${TOOLCHAIN_PATH}/bin/clang++.cfg"
	@${AUDITWHEEL_PLAT}.cfg
	-fuse-ld=lld
EOF
cat<<EOF >"${TOOLCHAIN_PATH}/bin/clang-cpp.cfg"
	@${AUDITWHEEL_PLAT}.cfg
EOF

mv /usr/local/bin/manylinux-entrypoint /usr/local/bin/manylinux-entrypoint-org
cat<<EOF >/usr/local/bin/manylinux-entrypoint
#!/bin/bash

set -eu

export PATH="${TOOLCHAIN_PATH}/bin:\${PATH}"
exec /usr/local/bin/manylinux-entrypoint-org "\$@"
EOF

chmod +x /usr/local/bin/manylinux-entrypoint
EOFD
