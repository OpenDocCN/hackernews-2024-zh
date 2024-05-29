<!--yml
category: 未分类
date: 2024-05-27 14:34:27
-->

# perl version cheat sheet

> 来源：[https://sheet.shiar.nl/perl/](https://sheet.shiar.nl/perl/)

# Perl release summary

The most significant features introduced for recent versions of the Perl scripting language.

Core security support is provided for 3 years, so typical users should run at least [5.36](#v5036). Stable distributions such as Debian 10 maintain [5.28](#v5028)+. Enterprise platforms retain versions up to [5.10](#v5010).

## 5.38 2023-07-02: latest stable release

`use feature "module_true"`

default in use 5.37 and up, also `no feature "bareword_filehandles"`

`sub ($var ||=` default`)`

assign values when false (or undefined on `//=`) instead of omitted

`/(*{ … })/`

optimistic eval: `(?{ … })` with regex optimisations enabled

`class`

define object classes: packages with `field` variables and `method` subroutines *(feature, experimental)*

`${^LAST_<wbr>SUCCESSFUL_<wbr>PATTERN}`

explicit variable to access the previous match as in `s//…/`

`%{^HOOK}`

perform tasks `require__before` and `require__after` when calling `require`

Unicode

v15.0

## 5.36 2022-05-28: active core support

`use v5.36`

use `warnings`; use feature qw'`signatures isa`'; no feature qw'`indirect multidimensional switch`'

`use builtin`

namespace for interpreter functions, such as `weaken` and `blessed` from `Scalar::Util`, `ceil`/`floor` from `POSIX`, and `trim` like `String::Util` *(experimental)*

`is_bool(!0)`

distinguish scalar variable types (by `builtin` functions) for data interoperability

`for my ($k, $v) (%hash)`

iterate over multiple values at a time (including `builtin::indexed` for arrays) *(feature, experimental)*

`defer {}`

queue code to be executed when going out of scope *(feature, experimental)*

`try {} finally {}`

run code at the end of a `[try](#try)` construct regardless of failure *(feature, experimental)*

unicode delimiters for quoting operators *(experimental)*

`sub ($var) {!~~pop~~}`

[signature](#signatures)d subs are stable, but mixing with the arguments array `@_` remains experimental *(feature, experimental)*

`$SIG{FPE}`

floating-point exceptions no longer deferred but delivered immediately like other signals

perl `-g`

disable input record separator (slurp mode), alias for `-0777`

Unicode

v14.0

## 5.34 2021-05-20

`try {} catch`

exception handling similar to eval blocks *(feature, experimental)*

`/{,*n*}/`

empty lower bound quantifier is accepted as shorthand for 0

`\x{ … }`

insignificant space within curly braces, also for `\b{}`, `\g{}`, `\k{}`, `\N{}`, `\o{}` as well as `/{m,n}/` quantifiers

`0o0`

octal prefix `0o` alternative to `0…` and `oct`

`re::optimization(qr//)`

debug regular expression optimization information discovered at compile time

`no feature …`

disable discouraged practices of `bareword_filehandles` and `multidimensional` array emulation

## 5.32 2020-06-20

`isa`

infix operator to check class instance *(feature, experimental until 5.36)*

`$min < $_ <= $max`

chained comparison repeats inner part as `$min < $_ and $_ <= $max`

`/\p{Name=$var}/`

match Unicode Name property like `\N{}` but with interpolation and subpatterns

`open F, '+>>', undef`

respect append mode on temporary files with mixed access

`no feature 'indirect'`

disable indirect object notation such as `new Class` instead of `Class->new`

streamzip

program distributed with core IO::Compress::Base to compress stdin into a zip container

Unicode

v13.0

## 5.30 2019-05-22

`/(?<=var+)`

variable length lookbehind assertions *(experimental until 5.36)*

`m(\p{nv=/.*/})`

match unicode properties by regular expressions *(experimental)*

`~~my $state if 0~~`

workaround for `[state](#state)` (deprecated since v5.10!) is now prohibited

`qr'\N'`

Delimiters must be graphemes; unescaped `{` illegal; `\N` in single quotes

Unicode

v12.1

## 5.28 2018-06-22: still maintained by common vendors

`delete %hash{…}`

hash slices can be deleted with key+value pairs

`/(*…)/`

alphabetic synonyms for assertions, e.g. `(*atomic:…)` for `(?>…)` and `(*nlb:…)` for `(?<!…)` *(experimental until 5.31.6)*

`/(*script_run:)/`

enforces all characters to be from the same script *(experimental until 5.31.6)*

`state @a`

persistent lexical array or hash variables (in addition to [scalars](#state))

perl `-i -pe die`

safe in-place editing: files are replaced only after successful completion

`${^SAFE_LOCALES}`

locales are thread-safe on supported systems, indicated by this variable

Unicode

v10.0

## 5.26 2017-05-30

`<<~EOT`

indented here-docs, strips same whitespace before delimiter in each line

`@{^CAPTURE}`

array of last match's captures, so `${^CAPTURE}[0]` is `$1`

`//xx`

extended modifier to also ignore whitespace in bracketed character classes

`use Test2::V0`

generic testing framework to replace `Test::*` and `TAP::*`

Unicode

v9.0

## 5.24 2016-05-09

`printf '%.*2$x'`

reordered precision arguments

`/\b{lb}/`

line break boundary type (position suitable for hyphenation)

`/faster/`

various significant speedups, notably matching fixed substrings, `/i` on caseless languages, 64-bit arithmetic, scope overhead

Unicode

v8.0

## 5.22 2015-06-01

`\$alias =`

aliasing via reference (scoped as of v5.25.3) *(experimental)*

`<<>>`

safe `readline` ignoring open flags in arguments

`/()/n`

flag to disable numbered capturing, turning `()` into `(?:)`

`/\b{}/`

boundary types: *gcb* (grapheme cluster), *sb* (sentence), *wb* (word)

`&.`

`& | ^ ~` consistently numeric, dotted operators for strings *(feature, experimental until 5.28)*

`use re 'strict'`

apply stricter syntax rules to regular expression patterns *(experimental)*

`0x.beep+0`

hexadecimal floating point notation with binary power; `printf '%a'` to display

`~~??~~`

single match shorthand (deprecated since v5.14) requires the operator `*m*?PATTERN?`

`~~use CGI~~`

deprecated interface for serving http requests removed from core, see [CGI::Alternatives](https://metacpan.org/pod/CGI::Alternatives)

Unicode

v7.0

## 5.20 2014-05-27: extended vendor support 202X

`sub ($var)`

subroutine signatures *(feature, experimental until 5.36)*

`%hash{…}`

hash slices return key+value pairs

`[]->@*`

postfix dereferencing (also e.g. `$scalar->$*` for `$$scalar`) *(feature, experimental until 5.23.1)*

`use warnings 'once'; $a`

variables $a and $b are exempt from *used once* warnings

Unicode

v6.3

## 5.18 2013-05-18

`${^LAST_FH}`

last read filehandle (used by `$.`)

`/(?[ a + b ])/`

regex set operations (character subtraction `-`, union `+`, intersection `&`, xor `^`) *(experimental until 5.36)*

`my sub`

lexical subroutines (also `state`, `our`); buggy before v5.22 *(experimental until 5.26)*

`next $expression`

loop controls allow runtime expressions

`no warnings 'experimental::…'`

mechanism for experimental features, as of now required for *smartmatch*

Unicode

v6.2

## 5.16 2012-05-20

`__SUB__`

current subroutine reference *(feature)*

`fc, "\F"`

unicode foldcase to compare case-insensitively *(feature)*

`"\N{}"`

automatic `use charnames qw( :full :short )`

Unicode

v6.1

## 5.14 2011-05-14

`s///r`

non-destructive substitution

`/(?{ m() })/`

regular expressions can be nested in `/(?{})/` and `/(??{})/` *(experimental until 5.20)*

`/dalu`

regexp modifiers to restrict character classes: either **d**efault, **a**scii, **l**ocale, or **u**nicode semantics.

`use re '/flags'`

customise default modifiers

`/(?^)/`

construct to reset to default modifiers

`FH->method`

filehandle method calls load IO::File on demand (eg. `STDOUT->flush`)

`\o{}`

escape sequence for octal values beyond \777

`use JSON`

interface with data in JavaScript Object Notation {`decode_json <>`}

Unicode

v6.0+#8

## 5.12 2010-04-12

`package` version

`package` NAME VERSION shorthand for `our $VERSION`

`...`

yada-yada operator: code placeholder

`use 5.012`

implicit `strict` if use VERSION >= v5.12

`… when`

`when` is now allowed to be used as a statement modifier

`use overload 'qr'`

customisable conversion to regular expressions

`/\N/`

inverse \n to match any character except newline regardless of `/s` *(experimental until 5.18)*

Unicode

v5.2

## 5.10 2007-12-18: supported commercially until 2024

`//`

defined-or operator

`~~`

smart-match operator to compare different data types (updated in v5.10.1) *(experimental)*

`say`

print with newline, equivalent to `print @_, "\n"` *(feature)*

`given`

switch statement to smart-match with `when`/`default` *(feature, experimental)*

`/(?<name>)/`

named capture buffers into `%+`

`/(?1)/`

recursive regular expression patterns

`/(?|)/`

resets capture numbering for each contained branch

`/.++/`

possessive quantifiers `?+`, `*+`, `++` to match greedily

`s/keep\K//`

floating positive lookbehind, efficient alternative for `s/(keep)/$1/`

`/p`

optionally preserve `${^MATCH}` variables (avoiding `$&` penalty until COW in v5.20)

`/\v/, /\h/`

vertical and horizontal whitespace escapes (`\V` `\H` to invert); also `/\R/` for newlines

`state`

persistent `my` variables (scalars only until [5.28](#state_ext)) *(feature)*

`use autodie`

replace builtin functions to throw exceptions instead of returning failure {`eval {open ...} or $@->matches("open") || die`}

`use IO::Compress::Zip`

various file compression standards {`zip IO::Uncompress::Gunzip->new("test.gz")`<wbr> `=> "recompressed.zip"`}

`use Time::Piece`

timestamps as objects {`localtime->year > 1900`}

`use File::Fetch`

generic data retrieval/download {`File::Fetch->new(uri => "http://localhost/")`<wbr>`->fetch(to => \$slurp)`}

Unicode

v5.0.0

## 5.8 2002-07-18: stable minimum during 20[01]\d

`no utf8`

full unicode support, `utf8` pragma only for script encoding

`use open`

file handle behaviour altered by PerlIO layers {`binmode $fh, ":bytes"`}

`open $fh, '-|', @cmd`

open list to fork a command without spawning a shell

`open $fh, '>', \$var`

perl scalars as virtual files

`printf '%1$s', @args`

syntax to use parameters out of order

`1_2_3 == 123`

underscores between digits allowed in numeric constants

`use bignum`

transparent big number support {`length 1e100 == 101`}

`use if`

conditional module inclusion {`no if $] >= 5.022, "warnings", "redundant"`}

`use Digest`

calculate various message digests (data hashes) {`$hash = sha256_hex($data)`}

`use Encode`

character set conversion {`encode("utf8", decode("iso-8859-1", $octets))`}

`use File::Temp`

create a temporary file or directory safely {`$fh = tempfile();`}

`use List::Util`

general-utility list subroutines {`@cards = shuffle 0..51`}

`use Locale::Maketext`

various localization and internationalization in `Locale::*` and `L18N::*`

`use Memoize`

remember function results, trading space for time {`memoize "stat"`}

`use MIME::Base64`

base64 encoded strings as in email attachments

`use Test::More`

modern framework for unit testing {`is $got, $expected`}

`use Time::HiRes`

high resolution timers {`$μs = [gettimeofday]; sleep .1;`<wbr> `$elapsed = tv_interval $μs`}

Unicode

v3.2.0

## 5.6 2000-03-23: start of modern compatibility

`use warnings`

pragma to enable warnings in lexical scope

`use utf8`

experimental unicode semantics (completed in [v5.8](#utf8_data)) *(experimental until 5.8)*

`use charnames`

string escape `\N{}` to insert named character

`our`

declare global variables

`v1.2.3`

represent strings as vector of ordinals, useful in version numbers (`printf '%vd'` to display)

`0b0`

binary numbers in literals, `printf '%b'`, and `oct`

`sub :lvalue`

subroutine attribute to return a modifiable value *(experimental until 5.20)*

`open my $fh, $mode, $expr`

file handles in scoped scalars, third argument for unambiguous file name

`pack 'q'`

64-bit integer support (also large files >2GiB) *(experimental until 5.8.1)*

`sort $coderef ()`

comparison function can be a subroutine reference; prototype `($$)` to pass elements as normal `@_`

`CHECK {}`

special block called at end of compilation

`/[[:…:]]/`

POSIX character class syntax such as `/[[:alpha:]]/`

Unicode

v3.0.1