# AddressSanitizer support in IX

> Prerequisites:<br>
> [IX.md](IX.md)<br>

**IX** can build C/C++ binaries with [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) to detect and debug memory
safety errors. This feature is considered experimental because statically linked binaries are [not](https://clang.llvm.org/docs/AddressSanitizer.html#limitations)
officially supported by AddressSanitizer.

Note: AddressSanitizer integration in **IX** only works for `lib/musl`. Alternative C libraries are currently not supported.

## Basic usage

1. Specify the `--sanitize=address` flag when building the binary package.
2. (optional, but highly recommended) Set the `ASAN_SYMBOLIZER_PATH` environment variable to the path to the `llvm-symbolizer` binary that is built along with `clang` as part of **IX** bootstrapping: `export ASAN_SYMBOLIZER_PATH=$(find /ix/store -name llvm-symbolizer | head -n1)`.

For example, let's reproduce [this issue](https://github.com/gavinhoward/bc/commit/8303d0bcc6a78d0e55bf365366761040ed0a17b8)
in `bin/bc/gavin`. At the time of writing, this issue hasn't yet been patched in the latest publicly available release.

Without AddressSanitizer, the sequence of commands described in the issue leads to an unhelpful segfault:
```
> ix run bin/bc/gavin -- dc
TOUCH /ix/store/mgSvhXAoiKDgnCi5lw89X1-rlm-ephemeral/touch
*

Runtime error: stack has too few elements
    0: (main)

f
zsh: segmentation fault (core dumped)  ./ix run bin/bc/gavin -- dc
```

With AddressSanitizer, we can immediately see where the crash comes from:
```
> export ASAN_SYMBOLIZER_PATH=$(find /ix/store -name llvm-symbolizer | head -n1)
> ix run bin/bc/gavin --sanitize=address -- dc
TOUCH /ix/store/LEoZmBPsRU2IvVMcaDn555-rlm-ephemeral/touch
*

Runtime error: stack has too few elements
    0: (main)

f
AddressSanitizer:DEADLYSIGNAL
=================================================================
==2217784==ERROR: AddressSanitizer: SEGV on unknown address (pc 0x00000037741c bp 0x7ffc4fd38af0 sp 0x7ffc4fd38aa0 T0)
==2217784==The signal is caused by a READ memory access.
==2217784==Hint: this fault was caused by a dereference of a high value address (see register values below).  Disassemble the provided pc to learn which register was used.
    #0 0x00000037741c in bc_program_print (/ix/store/kChw3BvAVuPhuPlCGacRh2-bin-bc-gavin/bin/bc+0x37741c)
    #1 0x00000031b350 in __sanitizer::BufferedStackTrace::UnwindImpl(unsigned long, unsigned long, void*, bool, unsigned int) (/ix/store/kChw3BvAVuPhuPlCGacRh2-bin-bc-gavin/bin/bc+0x31b350)
    #2 0x0000003379b6 in __sanitizer::ReportDeadlySignal(__sanitizer::SignalContext const&, unsigned int, void (*)(__sanitizer::SignalContext const&, void const*, __sanitizer::BufferedStackTrace*), void const*) (/ix/store/kChw3BvAVuPhuPlCGacRh2-bin-bc-gavin/bin/bc+0x3379b6)
    #3 0x000000316c06 in __asan::ScopedInErrorReport::~ScopedInErrorReport() (/ix/store/kChw3BvAVuPhuPlCGacRh2-bin-bc-gavin/bin/bc+0x316c06)
    #4 0x000000316a08 in __asan::ReportDeadlySignal(__sanitizer::SignalContext const&) (/ix/store/kChw3BvAVuPhuPlCGacRh2-bin-bc-gavin/bin/bc+0x316a08)
    #5 0x000000316011 in __asan::AsanOnDeadlySignal(int, void*, void*) (/ix/store/kChw3BvAVuPhuPlCGacRh2-bin-bc-gavin/bin/bc+0x316011)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV (/ix/store/kChw3BvAVuPhuPlCGacRh2-bin-bc-gavin/bin/bc+0x37741c) in bc_program_print
==2217784==ABORTING
```

## More examples

_Warning: these examples are for illustrative purposes only. Do not use AddressSanitizer as a hardening tool. Quoting the official documentation:_
> While it may be useful for testing, AddressSanitizer’s runtime was not developed with security-sensitive constraints in mind and may compromise the security of the resulting executable.

Let's look at some of the known memory-safety-related vulnerabilities in programs packaged in **IX**, and try to trigger them with AddressSanitizer enabled.
All of these examples were ran with setting `ASAN_SYMBOLIZER_PATH` as described above.

### CVE-2025-26519

`lib/musl` has an unpatched (at the time of writing) buffer overflow in `iconv()` when converting from `EUC-KR` to `UTF-8`. We will create a dummy package with a simple program
that intentionally triggers the overflow by crafting a malicious input of specified size:

<!-- {% raw %} -->
```cpp
> cat pkgs/bin/CVE-2025-26519/m.cpp
#include <iconv.h>

#include <iostream>
#include <vector>
#include <string>

int main(int argc, char** argv)
{
        if (argc != 2) {
                std::cerr << "Usage: " << argv[0] << " <N>" << std::endl;
                return 1;
        }

        size_t n = std::stoll(argv[1]);
        iconv_t cd = iconv_open("UTF-8", "EUC-KR");
        std::cout << "Processing input of size " << n << std::endl;

        std::vector<char> in(n);
        size_t ii = 0;
        for (ii = 0; ii + 1 < n; ++ii) {
                in[ii] = ii % 2 == 0 ? 0xc8 : 0x41;
        }
        in[ii] = 0x42;
        size_t inb = in.size();
        char* pin = in.data();

        std::vector<char> out(4 * in.size());
        size_t outb = out.size();
        char* pout = out.data();

        size_t res = iconv(cd, &pin, &inb, &pout, &outb);
        std::cout << "Converted " << res << " characters" << std::endl;

        iconv_close(cd);
}
```
```
> cat pkgs/bin/CVE-2025-26519/ix.sh
{% extends '//die/inline/program.sh' %}

{% block bld_libs %}
lib/c
lib/c++
{% endblock %}

{% block name %}
CVE-2025-26519
{% endblock %}

{% block sources %}
m.cpp
{% endblock %}
```

Without AddressSanitizer, the overflow either goes completely undetected, or leads to a segfault:
```
> ix run bin/CVE-2025-26519 -- CVE-2025-26519 69
TOUCH /ix/store/YD7dWQtGDJz6UcAelY2V50-rlm-ephemeral/touch
Processing input of size 69
Converted 0 characters
> ix run bin/CVE-2025-26519 -- CVE-2025-26519 313371
TOUCH /ix/store/YD7dWQtGDJz6UcAelY2V50-rlm-ephemeral/touch
Processing input of size 313371
zsh: segmentation fault (core dumped)  ./ix run bin/CVE-2025-26519 -- CVE-2025-26519 313371
```

With AddressSanitizer, we get a report about overflowing the heap, including the location of the problematic memory access:
```
> ix run bin/CVE-2025-26519 --sanitize=address -- CVE-2025-26519 69
TOUCH /ix/store/2zNsusYlpM3TiiGfBjmjZ2-rlm-ephemeral/touch
Processing input of size 69
=================================================================
==2228214==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x7cd91848001e at pc 0x0000003f7292 bp 0x7ffcd6918db0 sp 0x7ffcd6918da8
WRITE of size 1 at 0x7cd91848001e thread T0
    #0 0x0000003f7291 in __real_wcrtomb (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x3f7291)
    #1 0x0000002f9990 in __sanitizer::BufferedStackTrace::UnwindImpl(unsigned long, unsigned long, void*, bool, unsigned int) (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x2f9990)
    #2 0x000000295bcb in __asan::ErrorGeneric::Print() (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x295bcb)
    #3 0x0000002f5036 in __asan::ScopedInErrorReport::~ScopedInErrorReport() (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x2f5036)
    #4 0x0000002f7c72 in __asan::ReportGenericError(unsigned long, unsigned long, unsigned long, unsigned long, bool, unsigned long, unsigned int, bool) (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x2f7c72)
    #5 0x0000002f8548 in __asan_report_store1 (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x2f8548)

0x7cd91848001e is located 34 bytes before 276-byte region [0x7cd918480040,0x7cd918480154)
allocated by thread T0 here:
    #0 0x0000002f1bd4 in malloc (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x2f1bd4)
    #1 0x0000004602c7 in operator new(unsigned long) (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x4602c7)
    #2 0x00000031cfd9 in std::__1::vector<char, std::__1::allocator<char>>::vector[abi:ne190107](unsigned long) (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x31cfd9)
    #3 0x00000031cb8e in main (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x31cb8e)
    #4 0x00000046023c in libc_start_main_stage2 (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x46023c)
    #5 0x00000045ff2d in _start (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x45ff2d)

SUMMARY: AddressSanitizer: heap-buffer-overflow (/ix/store/6ygb6QxOpVZKGVXzYoZol7-bin-cve-2025-26519/bin/CVE-2025-26519+0x3f7291) in __real_wcrtomb
Shadow bytes around the buggy address:
  0x7cd91847fd80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7cd91847fe00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7cd91847fe80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7cd91847ff00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7cd91847ff80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x7cd918480000: fa fa fa[fa]fa fa fa fa 00 00 00 00 00 00 00 00
  0x7cd918480080: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7cd918480100: 00 00 00 00 00 00 00 00 00 00 04 fa fa fa fa fa
  0x7cd918480180: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x7cd918480200: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x7cd918480280: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==2228214==ABORTING
```

Note something very interesting: the report says the `WRITE` happened _inside_ `lib/musl` (namely, in [this](https://git.musl-libc.org/cgit/musl/tree/src/multibyte/wcrtomb.c#n6) function).
**IX** makes this possible by using static linking and instrumenting everything compiled into the final binary, including the C library.
When using AddressSanitizer outside of **IX**, the C library is typically linked dynamically and is not instrumented.

### CVE-2025-2574

`bin/xpdf` is a PDF renderer written in C++ that is notorious for [multiple unpatched vulnerabilities](https://github.com/NixOS/nixpkgs/blob/5f4f306bea96741f1588ea4f450b2a2e29f42b98/pkgs/applications/misc/xpdf/default.nix#L96-L105).

Let's trigger [one of them](https://www.xpdfreader.com/security-bug/CVE-2025-2574.html): a [stack overflow](https://github.com/karelbilek/xpdf-mirror/blob/2009ee360556e8260a2d8b21820bb8f36c9377e8/xpdf/Function.cc#L1229)
when executing a PostScript function.

To run `bin/xpdf` with AddressSanitizer, we currently have to apply workarounds for two known issues related to AddressSanitizer
integration in **IX**, both of which will eventually be fixed.

First, the following patch should be applied to the IX package base:
```
> git diff
diff --git a/pkgs/bin/xpdf/viewer/ix.sh b/pkgs/bin/xpdf/viewer/ix.sh
index 3cec77d0e..913284aab 100644
--- a/pkgs/bin/xpdf/viewer/ix.sh
+++ b/pkgs/bin/xpdf/viewer/ix.sh
@@ -3,7 +3,6 @@
 {% block bld_libs %}
 lib/qt/6/base
 lib/qt/6/deps
-lib/{{allocator}}/trim(delay=1,bytes=1000000)
 {{super()}}
 {% endblock %}
```
<!-- {% endraw %} -->

Second, we need to set the `ASAN_OPTIONS` environment variable to `detect_odr_violation=0`.

With that out of the way, let's craft a malicious PDF:
```
> cat xpdf_CVE-2025-2574.pdf
%PDF-1.4
1 0 obj
<< /Type /Catalog /Pages 2 0 R >>
endobj
2 0 obj
<< /Type /Pages /Kids [3 0 R] /Count 1 >>
endobj
3 0 obj
<< /Type /Page
   /MediaBox [0 0 3 3]
   /Resources <<
        /Shading << /Sh1 4 0 R >>
    >>
   /Contents 5 0 R >>
endobj
4 0 obj
<< /ShadingType 1 /Function 6 0 R >>
endobj
5 0 obj
<< /Length 7 >>
stream
/Sh1 sh
endstream
endobj
6 0 obj
<< /FunctionType 4 /Domain [0 1 0 1] /Range [0 1 0 1 0 1]
   /Length 33 >>
stream
{ 2147483647 1 roll }
endstream
endobj
trailer
<< /Size 7 /Root 1 0 R >>
%%EOF
```

And try to open it with `bin/xpdf` to get a nice crash report:
```
> ix run bin/xpdf --opengl=mesa/radv --vulkan=mesa/radv --sanitize=address -- xpdf xpdf_CVE-2025-2574.pdf
TOUCH /ix/store/rKMJ0W8qTWG2owpaOdYtM7-rlm-ephemeral/touch
Config Error: No paper information available - using defaults
Config Error: No display font for 'Courier'
Config Error: No display font for 'Courier-Bold'
Config Error: No display font for 'Courier-BoldOblique'
Config Error: No display font for 'Courier-Oblique'
Config Error: No display font for 'Helvetica'
Config Error: No display font for 'Helvetica-Bold'
Config Error: No display font for 'Helvetica-BoldOblique'
Config Error: No display font for 'Helvetica-Oblique'
Config Error: No display font for 'Symbol'
Config Error: No display font for 'Times-Bold'
Config Error: No display font for 'Times-BoldItalic'
Config Error: No display font for 'Times-Italic'
Config Error: No display font for 'Times-Roman'
Config Error: No display font for 'ZapfDingbats'
=================================================================
==2234223==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7b206f240f40 at pc 0x000002a6b27a bp 0x7b206fdd3cf0 sp 0x7b206fdd3ce8
READ of size 8 at 0x7b206f240f40 thread T4
    #0 0x000002a6b279 in PostScriptFunction::exec(double*, int) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a6b279)
    #1 0x000002a6900b in PostScriptFunction::transform(double*, double*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a6900b)
    #2 0x000002a67643 in PostScriptFunction::PostScriptFunction(Object*, Dict*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a67643)
    #3 0x000002a6035e in Function::parse(Object*, int, int, int) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a6035e)
    #4 0x000002ac22f6 in GfxFunctionShading::parse(Dict*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2ac22f6)
    #5 0x000002ac0445 in GfxShading::parse(Object*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2ac0445)
    #6 0x000002a81057 in GfxResources::lookupShading(char const*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a81057)
    #7 0x000002a7ee3c in Gfx::opShFill(Object*, int) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a7ee3c)
    #8 0x000002a840e1 in Gfx::execOp(Object*, Object*, int) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a840e1)
    #9 0x000002a8331c in Gfx::go(int) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a8331c)
    #10 0x000002a825d8 in Gfx::display(Object*, int) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a825d8)
    #11 0x000002b5a043 in Page::displaySlice(OutputDev*, double, double, int, int, int, int, int, int, int, int, int (*)(void*), void*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2b5a043)
    #12 0x000002b64aef in PDFDoc::displayPageSlice(OutputDev*, int, double, double, int, int, int, int, int, int, int, int, int (*)(void*), void*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2b64aef)
    #13 0x000002c22656 in TileCache::rasterizeTile(CachedTileDesc*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c22656)
    #14 0x000002c22251 in TileCacheThreadPool::worker() (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c22251)
    #15 0x000002c21ec8 in TileCacheThreadPool::threadFunc(void*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c21ec8)
    #16 0x0000028e8bd0 in asan_thread_start(void*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x28e8bd0)

Address 0x7b206f240f40 is located in stack of thread T4 at offset 832 in frame
    #0 0x000002a68c6f in PostScriptFunction::transform(double*, double*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a68c6f)

  This frame has 1 object(s):
    [32, 832) 'stack' <== Memory access at offset 832 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism, swapcontext or vfork
      (longjmp and C++ exceptions *are* supported)
Thread T4 created by T0 here:
    #0 0x0000028dcced in pthread_create (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x28dcced)
    #1 0x000002c21df3 in TileCacheThreadPool::TileCacheThreadPool(TileCache*, int) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c21df3)
    #2 0x000002c22c61 in TileCache::TileCache(DisplayState*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c22c61)
    #3 0x000002bb9a5c in PDFCore::PDFCore(SplashColorMode, int, int, unsigned char*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2bb9a5c)
    #4 0x000002c36762 in QtPDFCore::QtPDFCore(QWidget*, QScrollBar*, QScrollBar*, unsigned char*, unsigned char*, int) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c36762)
    #5 0x000002c931d5 in XpdfWidget::setup(QColor const&, QColor const&, bool) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c931d5)
    #6 0x000002c938e0 in XpdfWidget::XpdfWidget(QWidget*, QColor const&, QColor const&, bool) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c938e0)
    #7 0x000002c65b93 in XpdfViewer::addTab() (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c65b93)
    #8 0x000002c624a3 in XpdfViewer::createWindow() (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c624a3)
    #9 0x000002c61914 in XpdfViewer::XpdfViewer(XpdfApp*, int) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c61914)
    #10 0x000002c62e91 in XpdfViewer::create(XpdfApp*, QString, int, QString, int, QString, int) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c62e91)
    #11 0x000002c465d5 in XpdfApp::openInNewWindow(QString, int, QString, int, QString, int, char const*) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c465d5)
    #12 0x000002c43cc3 in XpdfApp::XpdfApp(int&, char**) (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c43cc3)
    #13 0x000002c9c911 in main (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2c9c911)
    #14 0x00000944698c in libc_start_main_stage2 (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x944698c)
    #15 0x000009446675 in _start (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x9446675)

SUMMARY: AddressSanitizer: stack-buffer-overflow (/ix/store/9tJIZuirWt11B1gevHh8m1-bin-xpdf-viewer/bin/xpdf+0x2a6b279) in PostScriptFunction::exec(double*, int)
Shadow bytes around the buggy address:
  0x7b206f240c80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7b206f240d00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7b206f240d80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7b206f240e00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7b206f240e80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x7b206f240f00: 00 00 00 00 00 00 00 00[f3]f3 f3 f3 f3 f3 f3 f3
  0x7b206f240f80: f3 f3 f3 f3 f3 f3 f3 f3 00 00 00 00 00 00 00 00
  0x7b206f241000: f1 f1 f1 f1 00 00 00 00 00 00 00 00 00 00 00 00
  0x7b206f241080: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7b206f241100: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x7b206f241180: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==2234223==ABORTING
```

Note that `bin/xpdf` is an application that depends on complex libraries such as FreeType, Qt, and Mesa.
We didn't have to care about any of that: everything was statically linked and instrumented, so
if the vulnerability was in any of the dependencies, AddressSanitizer would also catch it.

## Implementation details

### Interceptors

The AddressSanitizer runtime provides dozens of [interceptors](https://maskray.me/blog/2023-01-08-all-about-sanitizer-interceptors),
which override selected functions from the standard C/C++ libraries. An interceptor can either:
  1. Completely override a standard function (e.g., `malloc()`).
  2. Resolve a standard function at runtime using `dlsym()` (e.g, `strcpy()`) and call it.

Since **IX** intentionally doesn't support dynamic linking, interceptors of the second type are modified to directly call the standard
function prefixed with `__real_` (e.g., `__real_strcpy()` instead of `strcpy()`). The library containing the `strcpy()` implementation
(i.e., `lib/musl`) is postprocessed after the build so that `strcpy()` is renamed to `__real_strcpy()`.

Most of the interceptors are actually redundant with static linking (since everything is instrumented anyway), but not all of them:
for example, functions written in pure assembly still need to be intercepted. For simplicity, we keep all interceptors that were
originally present in the AddressSanitizer runtime.

### Circular dependencies at compile-time

The AddressSanitizer runtime is built uninstrumented (for obvious reasons), and should technically depend on:
  1. `lib/musl`, for standard C functions, such as `getrlimit()`
  2. `lib/c++`, for stack unwinding
  3. both `lib/musl` and `lib/c++` for intercepted functions (see the section about interceptors above)

However, `lib/musl` and `lib/c++` depend back on the AddressSanitizer runtime, since they are instrumented,
and therefore need functions such as `__asan_report_load1()`. This introduces a circular dependency.

To resolve this, the AddressSanitizer runtime is built against the uninstrumented version of `lib/musl`, which provides
the headers for standard C functions (this solves 1). `clang` also has an internal version of `unwind.h` for stack unwinding,
which allows us to avoid depending on `lib/c++` (this solves 2). The intercepted functions are _declared_ in the
AddressSanitizer runtime itself (this solves 3).

The only remaining problem is that `lib/c++` (and also `lib/compiler_rt`) are built with CMake, which will try to link
a simple binary as part of the build process to check if the provided compiler is valid. This will not be possible,
because the binary is linked with the AddressSanitizer runtime, which contains references to functions residing in
`lib/c++`, which hasn't been built yet at that point. `lib/build/sanitize/hack_cmake` takes care of that.

### Circular dependencies at runtime

Instrumenting everything in `lib/musl` also introduces a circular dependency at runtime:
  * the instrumented TLS initialization code in `lib/musl` needs shadow memory to function properly
  * however, the sanitizer runtime initialization code that sets the shadow memory up implicitly assumes that TLS has already been initialized

We solve this by instructing the compiler to avoid instrumenting all `lib/musl` initialization code that is run before `main()`.
