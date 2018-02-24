# OpenSSL 升级文档

## 作者

**@author: `anxu@centrin.com.cn` || `axu.home@gmail.com`**

## 安装步骤

```bash
# 查看现有版本
>  openssl version
OpenSSL 1.0.1e-fips 11 Feb 2013

# 安装依赖组件
> yum -y install gcc gcc-c++ autoconf automake zlib zlib-devel pcre-devel perl-Params-Validate perl-Module-Load-Conditional telnet-server perl-core
[...]

# 下载最新稳定版本的 OpenSSL
# OpenSSL官网（https://www.openssl.org/source/）
> cd /opt
> wget https://www.openssl.org/source/openssl-1.1.0g.tar.gz
[...]

# 检查
> ll /opt/openssl-1.1.0g.tar.gz
-rw-r--r-- 1 root root 5404748 Nov  2 22:51 /opt/openssl-1.1.0g.tar.gz

# 解压
> tar -zxvf openssl-1.1.0g.tar.gz
[...]

# 检查
> ll openssl-1.1.0g
total 1640
-rw-r--r--  1 root root     87 Nov  2 22:29 ACKNOWLEDGEMENTS
drwxr-xr-x  4 root root   4096 Nov  2 22:29 apps
-rw-r--r--  1 root root    955 Nov  2 22:29 appveyor.yml
-rw-r--r--  1 root root    362 Nov  2 22:29 AUTHORS
-rw-r--r--  1 root root   2282 Nov  2 22:29 build.info
-rw-r--r--  1 root root 544528 Nov  2 22:29 CHANGES
-rwxr-xr-x  1 root root  28008 Nov  2 22:29 config
-rw-r--r--  1 root root   2507 Nov  2 22:29 config.com
-rw-r--r--  1 root root 258513 Nov  2 22:29 configdata.pm
drwxr-xr-x  2 root root   4096 Nov  2 22:29 Configurations
-rwxr-xr-x  1 root root  92389 Nov  2 22:29 Configure
-rw-r--r--  1 root root   2600 Nov  2 22:29 CONTRIBUTING
drwxr-xr-x 59 root root   4096 Nov  2 22:29 crypto
drwxr-xr-x  8 root root   4096 Nov  2 22:29 demos
drwxr-xr-x  6 root root   4096 Nov  2 22:29 doc
drwxr-xr-x  5 root root   4096 Nov  2 22:29 engines
-rw-r--r--  1 root root  15344 Nov  2 22:29 e_os.h
drwxr-xr-x  3 root root   4096 Nov  2 22:29 external
-rw-r--r--  1 root root     84 Nov  2 22:29 FAQ
drwxr-xr-x  2 root root   4096 Nov  2 22:29 fuzz
drwxr-xr-x  4 root root   4096 Nov  2 22:29 include
-rw-r--r--  1 root root  40715 Nov  2 22:29 INSTALL
-rw-r--r--  1 root root   6128 Nov  2 22:29 LICENSE
-rw-r--r--  1 root root 469211 Nov  2 22:29 Makefile
-rw-r--r--  1 root root  18400 Nov  2 22:29 Makefile.shared
drwxr-xr-x  2 root root   4096 Nov  2 22:29 ms
-rw-r--r--  1 root root  37562 Nov  2 22:29 NEWS
-rw-r--r--  1 root root   2095 Nov  2 22:29 NOTES.DJGPP
-rw-r--r--  1 root root   4580 Nov  2 22:29 NOTES.PERL
-rw-r--r--  1 root root   1276 Nov  2 22:29 NOTES.UNIX
-rw-r--r--  1 root root   2738 Nov  2 22:29 NOTES.VMS
-rw-r--r--  1 root root   5360 Nov  2 22:29 NOTES.WIN
drwxr-xr-x  2 root root   4096 Nov  2 22:29 os-dep
-rw-r--r--  1 root root   3209 Nov  2 22:29 README
-rw-r--r--  1 root root   3552 Nov  2 22:29 README.ECC
-rw-r--r--  1 root root  16087 Nov  2 22:29 README.ENGINE
-rw-r--r--  1 root root     61 Nov  2 22:29 README.FIPS
drwxr-xr-x  4 root root   4096 Nov  2 22:29 ssl
drwxr-xr-x 10 root root   4096 Nov  2 22:29 test
drwxr-xr-x  2 root root   4096 Nov  2 22:29 tools
drwxr-xr-x  4 root root   4096 Nov  2 22:29 util
drwxr-xr-x  2 root root   4096 Nov  2 22:29 VMS

# 切换目录
> cd /opt/openssl-1.1.0g
> pwd
/opt/openssl-1.1.0g

# 编译安装
> ./config
Operating system: x86_64-whatever-linux2
Configuring for linux-x86_64
Configuring OpenSSL version 1.1.0g (0x1010007fL)
    no-asan         [default]  OPENSSL_NO_ASAN
    no-crypto-mdebug [default]  OPENSSL_NO_CRYPTO_MDEBUG
    no-crypto-mdebug-backtrace [default]  OPENSSL_NO_CRYPTO_MDEBUG_BACKTRACE
    no-ec_nistp_64_gcc_128 [default]  OPENSSL_NO_EC_NISTP_64_GCC_128
    no-egd          [default]  OPENSSL_NO_EGD
    no-fuzz-afl     [default]  OPENSSL_NO_FUZZ_AFL
    no-fuzz-libfuzzer [default]  OPENSSL_NO_FUZZ_LIBFUZZER
    no-heartbeats   [default]  OPENSSL_NO_HEARTBEATS
    no-md2          [default]  OPENSSL_NO_MD2 (skip dir)
    no-msan         [default]  OPENSSL_NO_MSAN
    no-rc5          [default]  OPENSSL_NO_RC5 (skip dir)
    no-sctp         [default]  OPENSSL_NO_SCTP
    no-ssl-trace    [default]  OPENSSL_NO_SSL_TRACE
    no-ssl3         [default]  OPENSSL_NO_SSL3
    no-ssl3-method  [default]  OPENSSL_NO_SSL3_METHOD
    no-ubsan        [default]  OPENSSL_NO_UBSAN
    no-unit-test    [default]  OPENSSL_NO_UNIT_TEST
    no-weak-ssl-ciphers [default]  OPENSSL_NO_WEAK_SSL_CIPHERS
    no-zlib         [default]
    no-zlib-dynamic [default]
Configuring for linux-x86_64
CC            =gcc
CFLAG         =-Wall -O3 -pthread -m64 -DL_ENDIAN  -Wa,--noexecstack
SHARED_CFLAG  =-fPIC -DOPENSSL_USE_NODELETE
DEFINES       =DSO_DLFCN HAVE_DLFCN_H NDEBUG OPENSSL_THREADS OPENSSL_NO_STATIC_ENGINE OPENSSL_PIC OPENSSL_IA32_SSE2 OPENSSL_BN_ASM_MONT OPENSSL_BN_ASM_MONT5 OPENSSL_BN_ASM_GF2m SHA1_ASM SHA256_ASM SHA512_ASM RC4_ASM MD5_ASM AES_ASM VPAES_ASM BSAES_ASM GHASH_ASM ECP_NISTZ256_ASM PADLOCK_ASM POLY1305_ASM
LFLAG         =
PLIB_LFLAG    =
EX_LIBS       =-ldl
APPS_OBJ      =
CPUID_OBJ     =x86_64cpuid.o
UPLINK_OBJ    =
BN_ASM        =asm/x86_64-gcc.o x86_64-mont.o x86_64-mont5.o x86_64-gf2m.o rsaz_exp.o rsaz-x86_64.o rsaz-avx2.o
EC_ASM        =ecp_nistz256.o ecp_nistz256-x86_64.o
DES_ENC       =des_enc.o fcrypt_b.o
AES_ENC       =aes-x86_64.o vpaes-x86_64.o bsaes-x86_64.o aesni-x86_64.o aesni-sha1-x86_64.o aesni-sha256-x86_64.o aesni-mb-x86_64.o
BF_ENC        =bf_enc.o
CAST_ENC      =c_enc.o
RC4_ENC       =rc4-x86_64.o rc4-md5-x86_64.o
RC5_ENC       =rc5_enc.o
MD5_OBJ_ASM   =md5-x86_64.o
SHA1_OBJ_ASM  =sha1-x86_64.o sha256-x86_64.o sha512-x86_64.o sha1-mb-x86_64.o sha256-mb-x86_64.o
RMD160_OBJ_ASM=
CMLL_ENC      =cmll-x86_64.o cmll_misc.o
MODES_OBJ     =ghash-x86_64.o aesni-gcm-x86_64.o
PADLOCK_OBJ   =e_padlock-x86_64.o
CHACHA_ENC    =chacha-x86_64.o
POLY1305_OBJ  =poly1305-x86_64.o
BLAKE2_OBJ    =
PROCESSOR     =
RANLIB        =ranlib
ARFLAGS       =
PERL          =/usr/bin/perl

SIXTY_FOUR_BIT_LONG mode

Configured for linux-x86_64.

#!# 编译时间可能有点久
> make && make install
[...]
```
