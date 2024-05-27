<!--yml
category: 未分类
date: 2024-05-27 12:46:52
-->

# 'CVS: cvs.openbsd.org: src' - MARC

> 来源：[https://marc.info/?l=openbsd-cvs&m=171200100510963&w=2](https://marc.info/?l=openbsd-cvs&m=171200100510963&w=2)

```
[[prev in list](?l=openbsd-cvs&m=171199962910212&w=2)] [[next in list](?l=openbsd-cvs&m=171200380712333&w=2)] [[prev in thread](?l=openbsd-cvs&m=171199962910212&w=2)] [[next in thread](?l=openbsd-cvs&m=171200380712333&w=2)] 
 List:       [openbsd-cvs](?l=openbsd-cvs&r=1&w=2)
Subject:    [CVS: cvs.openbsd.org: src](?t=97834182900002&r=1&w=2)
From:       [Scott Soule Cheloha <cheloha () cvs ! openbsd ! org>](?a=161055562800001&r=1&w=2)
Date:       [2024-04-01 19:52:07](?l=openbsd-cvs&r=1&w=2&b=202404)
Message-ID: [3d329ff2b7df9749 () cvs ! openbsd ! org](?i=3d329ff2b7df9749%20()%20cvs%20!%20openbsd%20!%20org)
[Download RAW [message](?l=openbsd-cvs&m=171200100510963&q=mbox) or [body](?l=openbsd-cvs&m=171200100510963&q=raw)]

CVSROOT:	/cvs
Module name:	src
Changes by:	cheloha@cvs.openbsd.org	2024/04/01 14:48:02

Modified files:
	gnu/usr.bin    : Makefile
Added files:
	gnu/usr.bin/xz : AUTHORS CMakeLists.txt COPYING COPYING.0BSD
	                 COPYING.CC-BY-SA-4.0 COPYING.GPLv2 COPYING.GPLv3
	                 COPYING.LGPLv2.1 ChangeLog INSTALL INSTALL.generic
	                 Makefile.am NEWS PACKAGERS README THANKS TODO
	                 autogen.sh configure.ac
	gnu/usr.bin/xz/build-aux: ci_build.sh manconv.sh version.sh
	gnu/usr.bin/xz/cmake: remove-ordinals.cmake tuklib_common.cmake
	                      tuklib_cpucores.cmake tuklib_integer.cmake
	                      tuklib_large_file_support.cmake tuklib_mbstr.cmake
	                      tuklib_physmem.cmake tuklib_progname.cmake
	gnu/usr.bin/xz/debug: Makefile.am README crc32.c full_flush.c hex2bin.c
	                      known_sizes.c memusage.c repeat.c sync_flush.c
	                      translation.bash
	gnu/usr.bin/xz/doc: faq.txt history.txt lzma-file-format.txt
	                    xz-file-format.txt xz-logo.png
	gnu/usr.bin/xz/doc/examples: 00_README.txt 01_compress_easy.c
	                             02_decompress.c 03_compress_custom.c
	                             04_compress_easy_mt.c 11_file_info.c
	                             Makefile
	gnu/usr.bin/xz/dos: INSTALL.txt Makefile README.txt config.h
	gnu/usr.bin/xz/doxygen: Doxyfile footer.html update-doxygen
	gnu/usr.bin/xz/extra/7z2lzma: 7z2lzma.bash
	gnu/usr.bin/xz/extra/scanlzma: scanlzma.c
	gnu/usr.bin/xz/lib: Makefile.am getopt-cdefs.h getopt-core.h
	                    getopt-ext.h getopt-pfx-core.h getopt-pfx-ext.h
	                    getopt.c getopt.in.h getopt1.c getopt_int.h
	gnu/usr.bin/xz/m4: ax_pthread.m4 getopt.m4 posix-shell.m4
	                   tuklib_common.m4 tuklib_cpucores.m4
	                   tuklib_integer.m4 tuklib_mbstr.m4 tuklib_physmem.m4
	                   tuklib_progname.m4 visibility.m4
	gnu/usr.bin/xz/po: LINGUAS Makevars POTFILES.in ca.po cs.po da.po de.po
	                   eo.po es.po fi.po fr.po hr.po hu.po it.po ko.po pl.po
	                   pt.po pt_BR.po ro.po sr.po sv.po tr.po uk.po vi.po
	                   xz.pot-header zh_CN.po zh_TW.po
	gnu/usr.bin/xz/po4a: de.po fr.po ko.po po4a.conf pt_BR.po ro.po uk.po
	                     update-po
	gnu/usr.bin/xz/src: Makefile.am
	gnu/usr.bin/xz/src/common: common_w32res.rc mythread.h sysdefs.h
	                           tuklib_common.h tuklib_config.h
	                           tuklib_cpucores.c tuklib_cpucores.h
	                           tuklib_exit.c tuklib_exit.h tuklib_gettext.h
	                           tuklib_integer.h tuklib_mbstr.h
	                           tuklib_mbstr_fw.c tuklib_mbstr_width.c
	                           tuklib_open_stdxxx.c tuklib_open_stdxxx.h
	                           tuklib_physmem.c tuklib_physmem.h
	                           tuklib_progname.c tuklib_progname.h
	gnu/usr.bin/xz/src/liblzma: Makefile.am liblzma.pc.in
	                            liblzma_generic.map liblzma_linux.map
	                            liblzma_w32res.rc validate_map.sh
	gnu/usr.bin/xz/src/liblzma/api: Makefile.am lzma.h
	gnu/usr.bin/xz/src/liblzma/api/lzma: base.h bcj.h block.h check.h
	                                     container.h delta.h filter.h
	                                     hardware.h index.h index_hash.h
	                                     lzma12.h stream_flags.h version.h
	                                     vli.h
	gnu/usr.bin/xz/src/liblzma/check: Makefile.inc check.c check.h
	                                  crc32_arm64.h crc32_fast.c
	                                  crc32_small.c crc32_table.c
	                                  crc32_table_be.h crc32_table_le.h
	                                  crc32_tablegen.c crc32_x86.S
	                                  crc64_fast.c crc64_small.c
	                                  crc64_table.c crc64_table_be.h
	                                  crc64_table_le.h crc64_tablegen.c
	                                  crc64_x86.S crc_common.h
	                                  crc_x86_clmul.h sha256.c
	gnu/usr.bin/xz/src/liblzma/common: Makefile.inc alone_decoder.c
	                                   alone_decoder.h alone_encoder.c
	                                   auto_decoder.c block_buffer_decoder.c
	                                   block_buffer_encoder.c
	                                   block_buffer_encoder.h
	                                   block_decoder.c block_decoder.h
	                                   block_encoder.c block_encoder.h
	                                   block_header_decoder.c
	                                   block_header_encoder.c block_util.c
	                                   common.c common.h
	                                   easy_buffer_encoder.c index.c
	                                   easy_decoder_memusage.c
	                                   easy_encoder.c
	                                   easy_encoder_memusage.c easy_preset.c
	                                   easy_preset.h file_info.c
	                                   filter_buffer_decoder.c
	                                   filter_buffer_encoder.c
	                                   filter_common.c filter_common.h
	                                   filter_decoder.c filter_decoder.h
	                                   filter_encoder.c filter_encoder.h
	                                   filter_flags_decoder.c
	                                   filter_flags_encoder.c
	                                   hardware_cputhreads.c
	                                   hardware_physmem.c index.h
	                                   index_decoder.c index_decoder.h
	                                   index_encoder.c index_encoder.h
	                                   index_hash.c lzip_decoder.c
	                                   lzip_decoder.h memcmplen.h
	                                   microlzma_decoder.c
	                                   microlzma_encoder.c outqueue.c
	                                   outqueue.h stream_buffer_decoder.c
	                                   stream_buffer_encoder.c
	                                   stream_decoder.c stream_decoder.h
	                                   stream_decoder_mt.c stream_encoder.c
	                                   stream_encoder_mt.c
	                                   stream_flags_common.c
	                                   stream_flags_common.h
	                                   stream_flags_decoder.c
	                                   stream_flags_encoder.c
	                                   string_conversion.c vli_decoder.c
	                                   vli_encoder.c vli_size.c
	gnu/usr.bin/xz/src/liblzma/delta: Makefile.inc delta_common.c
	                                  delta_common.h delta_decoder.c
	                                  delta_decoder.h delta_encoder.c
	                                  delta_encoder.h delta_private.h
	gnu/usr.bin/xz/src/liblzma/lz: Makefile.inc lz_decoder.c lz_decoder.h
	                               lz_encoder.c lz_encoder.h
	                               lz_encoder_hash.h lz_encoder_hash_table.h
	                               lz_encoder_mf.c
	gnu/usr.bin/xz/src/liblzma/lzma: Makefile.inc fastpos.h fastpos_table.c
	                                 fastpos_tablegen.c lzma2_decoder.c
	                                 lzma2_decoder.h lzma2_encoder.c
	                                 lzma2_encoder.h lzma_common.h
	                                 lzma_decoder.c lzma_decoder.h
	                                 lzma_encoder.c lzma_encoder.h
	                                 lzma_encoder_optimum_fast.c
	                                 lzma_encoder_optimum_normal.c
	                                 lzma_encoder_presets.c
	                                 lzma_encoder_private.h
	gnu/usr.bin/xz/src/liblzma/rangecoder: Makefile.inc price.h
	                                       price_table.c price_tablegen.c
	                                       range_common.h range_decoder.h
	                                       range_encoder.h
	gnu/usr.bin/xz/src/liblzma/simple: Makefile.inc arm.c arm64.c armthumb.c
	                                   ia64.c powerpc.c riscv.c
	                                   simple_coder.c simple_coder.h
	                                   simple_decoder.c simple_decoder.h
	                                   simple_encoder.c simple_encoder.h
	                                   simple_private.h sparc.c x86.c
	gnu/usr.bin/xz/src/lzmainfo: Makefile.am lzmainfo.1 lzmainfo.c
	                             lzmainfo_w32res.rc
	gnu/usr.bin/xz/src/scripts: Makefile.am xzdiff.1 xzdiff.in xzgrep.1
	                            xzgrep.in xzless.1 xzless.in xzmore.1
	                            xzmore.in
	gnu/usr.bin/xz/src/xz: Makefile.am args.c args.h coder.c coder.h
	                       file_io.c file_io.h hardware.c hardware.h list.c
	                       list.h main.c main.h message.c message.h mytime.c
	                       mytime.h options.c options.h private.h sandbox.c
	                       sandbox.h signals.c signals.h suffix.c suffix.h
	                       util.c util.h xz.1 xz_w32res.rc
	gnu/usr.bin/xz/src/xzdec: Makefile.am lzmadec_w32res.rc xzdec.1 xzdec.c
	                          xzdec_w32res.rc
	gnu/usr.bin/xz/tests: Makefile.am bcj_test.c code_coverage.sh
	                      compress_prepared_bcj_sparc
	                      compress_prepared_bcj_x86 create_compress_files.c
	                      test_bcj_exact_size.c test_block_header.c
	                      test_check.c test_compress.sh
	                      test_compress_generated_abc
	                      test_compress_generated_random
	                      test_compress_generated_text
	                      test_compress_prepared_bcj_sparc
	                      test_compress_prepared_bcj_x86 test_files.sh
	                      test_filter_flags.c test_filter_str.c
	                      test_hardware.c test_index.c test_index_hash.c
	                      test_lzip_decoder.c test_memlimit.c
	                      test_microlzma.c test_scripts.sh
	                      test_stream_flags.c test_suffix.sh test_vli.c
	                      tests.h tuktest.h xzgrep_expected_output
	gnu/usr.bin/xz/tests/files: README bad-0-backward_size.xz
	                            bad-0-empty-truncated.xz
	                            bad-0-footer_magic.xz bad-0-header_magic.xz
	                            bad-0-nonempty_index.xz bad-0cat-alone.xz
	                            bad-0cat-header_magic.xz
	                            bad-0catpad-empty.xz bad-0pad-empty.xz
	                            bad-1-block_header-1.xz
	                            bad-1-block_header-2.xz
	                            bad-1-block_header-3.xz
	                            bad-1-block_header-4.xz
	                            bad-1-block_header-5.xz bad-1-vli-1.xz
	                            bad-1-block_header-6.xz
	                            bad-1-check-crc32-2.xz bad-1-check-crc32.xz
	                            bad-1-check-crc64.xz bad-1-check-sha256.xz
	                            bad-1-lzma2-1.xz bad-1-lzma2-10.xz
	                            bad-1-lzma2-11.xz bad-1-lzma2-2.xz
	                            bad-1-lzma2-3.xz bad-1-lzma2-4.xz
	                            bad-1-lzma2-5.xz bad-1-lzma2-6.xz
	                            bad-1-lzma2-7.xz bad-1-lzma2-8.xz
	                            bad-1-lzma2-9.xz bad-1-stream_flags-1.xz
	                            bad-1-stream_flags-2.xz
	                            bad-1-stream_flags-3.xz
	                            bad-1-v0-uncomp-size.lz bad-1-v1-crc32.lz
	                            bad-1-v1-dict-1.lz bad-1-v1-dict-2.lz
	                            bad-1-v1-magic-1.lz bad-1-v1-magic-2.lz
	                            bad-1-v1-member-size.lz
	                            bad-1-v1-trailing-magic.lz
	                            bad-1-v1-uncomp-size.lz bad-1-vli-2.xz
	                            bad-2-compressed_data_padding.xz
	                            bad-2-index-1.xz bad-2-index-2.xz
	                            bad-2-index-3.xz good-0-empty.xz
	                            bad-2-index-4.xz bad-2-index-5.xz
	                            bad-3-corrupt_lzma2.xz
	                            bad-3-index-uncomp-overflow.xz
	                            bad-dict_size.lzma
	                            bad-too_big_size-with_eopm.lzma
	                            bad-too_small_size-without_eopm-1.lzma
	                            bad-too_small_size-without_eopm-2.lzma
	                            bad-too_small_size-without_eopm-3.lzma
	                            bad-unknown_size-without_eopm.lzma
	                            good-0cat-empty.xz good-0catpad-empty.xz
	                            good-0pad-empty.xz good-1-3delta-lzma2.xz
	                            good-1-arm64-lzma2-1.xz
	                            good-1-arm64-lzma2-2.xz
	                            good-1-block_header-1.xz
	                            good-1-block_header-2.xz
	                            good-1-block_header-3.xz
	                            good-1-check-crc32.xz good-1-check-crc64.xz
	                            good-1-check-none.xz good-1-check-sha256.xz
	                            good-1-delta-lzma2.tiff.xz
	                            good-1-empty-bcj-lzma2.xz good-1-lzma2-1.xz
	                            good-1-lzma2-2.xz good-1-lzma2-3.xz
	                            good-1-lzma2-4.xz good-1-lzma2-5.xz
	                            good-1-riscv-lzma2-1.xz
	                            good-1-riscv-lzma2-2.xz
	                            good-1-sparc-lzma2.xz
	                            good-1-v0-trailing-1.lz good-1-v0.lz
	                            good-1-v1-trailing-1.lz
	                            good-1-v1-trailing-2.lz good-1-v1.lz
	                            good-1-x86-lzma2.xz good-2-lzma2.xz
	                            good-2-v0-v1.lz good-2-v1-v0.lz
	                            good-2-v1-v1.lz good-2cat.xz
	                            good-known_size-with_eopm.lzma
	                            good-known_size-without_eopm.lzma
	                            good-large_compressed.lzma
	                            good-small_compressed.lzma
	                            good-unknown_size-with_eopm.lzma
	                            unsupported-1-v234.lz
	                            unsupported-block_header.xz
	                            unsupported-check.xz
	                            unsupported-filter_flags-1.xz
	                            unsupported-filter_flags-2.xz
	                            unsupported-filter_flags-3.xz
	gnu/usr.bin/xz/tests/ossfuzz: Makefile fuzz_common.h fuzz_decode_alone.c
	                              fuzz_decode_stream.c fuzz_encode_stream.c
	gnu/usr.bin/xz/tests/ossfuzz/config: fuzz_decode_alone.options
	                                     fuzz_decode_stream.options
	                                     fuzz_encode_stream.options
	                                     fuzz_lzma.dict fuzz_xz.dict
	gnu/usr.bin/xz/windows: INSTALL-MSVC.txt
	                        INSTALL-MinGW-w64_with_Autotools.txt
	                        INSTALL-MinGW-w64_with_CMake.txt
	                        README-Windows.txt build-with-cmake.bat
	                        build.bash liblzma-crt-mixing.txt

Log message:
gnu/usr.bin: import xz-utils 5.6.1

If we want to continue distributing ISO 9660 images sized for CD-ROM
we need to take a more aggressive approach to build artifact
compression.  650MB just ain't what it used to be.  gzip(1) is fine
and all, but the DEFLATE compression algorithm pales in comparison to
the state of the art.

Enter xz, a modern compression utility for modern binaries.  If
gzip(1) is a Subaru Forester that "gets you where you need to go",
xz is a lifted Ford F-250 that "just ran a stop and killed a cyclist".

We're in good company.  Several notable projects have adopted xz for
distribution, including kernel.org, FreeBSD, and Debian.  I think it's
pretty safe to say that xz is thoroughly battle-tested in production.

This commit imports the v5.6.1 release of xz-utils, replete with
binary inputs in tests/files/ for regress testing.  /usr/bin will
expand to include xz(1) and a smattering of wrapper scripts for
searching and paging through LZMA2 containers.

It will take a little bit of elbow grease to wire this thing up to our
build, but not too much.  I have full confidence that OpenBSD 7.6 will
be compressed -- and decompressed! -- with xz(1).

Joint effort with millert@.

ok tedu@

[[prev in list](?l=openbsd-cvs&m=171199962910212&w=2)] [[next in list](?l=openbsd-cvs&m=171200380712333&w=2)] [[prev in thread](?l=openbsd-cvs&m=171199962910212&w=2)] [[next in thread](?l=openbsd-cvs&m=171200380712333&w=2)] 

```

[Configure](?q=configure) | [About](?q=about) | [News](?q=news) | [Add a list](mailto:webguy@marc.info?subject=Add%20a%20list%20to%20MARC) | Sponsored by [KoreLogic](http://www.korelogic.com/)