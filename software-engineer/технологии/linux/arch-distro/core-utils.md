[wiki.archlinux.org](https://wiki.archlinux.org/title/Core_utilities#Essentials)

# Core utilities - ArchWiki

14–18 minutes

---

_Core utilities_ are the basic, fundamental tools of a [GNU/Linux](https://en.wikipedia.org/wiki/GNU/Linux_naming_controversy "wikipedia:GNU/Linux naming controversy") system. This article provides an incomplete overview of them, links their documentation and describes useful alternatives. The scope of this article includes, but is not limited to, the [GNU Core Utilities](https://en.wikipedia.org/wiki/GNU_Core_Utilities "wikipedia:GNU Core Utilities"). Most core utilities are traditional [Unix](https://en.wikipedia.org/wiki/Unix "wikipedia:Unix") tools and many were standardized by [POSIX](https://en.wikipedia.org/wiki/POSIX "wikipedia:POSIX") but have been developed further to provide more features.

Most command-line interfaces are documented in [man pages](https://wiki.archlinux.org/title/Man_page "Man page"), utilities by the [GNU Project](https://wiki.archlinux.org/title/GNU_Project "GNU Project") are documented primarily in [Info manuals](https://wiki.archlinux.org/title/Info_manual "Info manual"), some [shells](https://wiki.archlinux.org/title/Shell "Shell") provide a _help_ command for shell builtin commands. Additionally most utilities print their usage when run with the `--help` flag.

## Essentials

The following table lists some important utilities which [Arch Linux](https://wiki.archlinux.org/title/Arch_Linux "Arch Linux") users should be familiar with. See also [intro(1)](https://man.archlinux.org/man/intro.1).

| Package                                                         | Utility                                                        | Description                                                                                                                              | Documentation                                                                                                                                                                                                   | Alternatives                                                                                                                                                                   |
| --------------------------------------------------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| shell built-ins                                                 | cd                                                             | change directory                                                                                                                         | [cd(1p)](https://man.archlinux.org/man/cd.1p)                                                                                                                                                                   | [#cd alternatives](about:reader?url=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FCore_utilities%23Essentials#cd_alternatives)                                                    |
| GNU [coreutils](https://archlinux.org/packages/?name=coreutils) | ls                                                             | list directory                                                                                                                           | [ls(1)](https://man.archlinux.org/man/ls.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/ls-invocation.html)                                                                                 | [tree](https://archlinux.org/packages/?name=tree), [#ls alternatives](about:reader?url=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FCore_utilities%23Essentials#ls_alternatives) |
| cat                                                             | concatenate files to stdout                                    | [cat(1)](https://man.archlinux.org/man/cat.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/cat-invocation.html)       | [tac(1)](https://man.archlinux.org/man/tac.1), [#cat alternatives](about:reader?url=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FCore_utilities%23Essentials#cat_alternatives)                                    |                                                                                                                                                                                |
| mkdir                                                           | make directory                                                 | [mkdir(1)](https://man.archlinux.org/man/mkdir.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/mkdir-invocation.html) |                                                                                                                                                                                                                 |                                                                                                                                                                                |
| rmdir                                                           | remove empty directory                                         | [rmdir(1)](https://man.archlinux.org/man/rmdir.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/rmdir-invocation.html) |                                                                                                                                                                                                                 |                                                                                                                                                                                |
| rm                                                              | remove files or directories                                    | [rm(1)](https://man.archlinux.org/man/rm.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/rm-invocation.html)          | [shred](https://wiki.archlinux.org/title/Shred "Shred") [unlink(1)](https://man.archlinux.org/man/unlink.1)                                                                                                     |                                                                                                                                                                                |
| cp                                                              | copy files or directories                                      | [cp(1)](https://man.archlinux.org/man/cp.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/cp-invocation.html)          | [#cp alternatives](about:reader?url=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FCore_utilities%23Essentials#cp_alternatives)                                                                                     |                                                                                                                                                                                |
| mv                                                              | move files or directories                                      | [mv(1)](https://man.archlinux.org/man/mv.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/mv-invocation.html)          |                                                                                                                                                                                                                 |                                                                                                                                                                                |
| ln                                                              | make hard or symbolic links                                    | [ln(1)](https://man.archlinux.org/man/ln.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/ln-invocation.html)          | [sln(8)](https://man.archlinux.org/man/sln.8) (soname recovery)                                                                                                                                                 |                                                                                                                                                                                |
| [chown](https://wiki.archlinux.org/title/Chown "Chown")         | change file owner and group                                    | [chown(1)](https://man.archlinux.org/man/chown.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/chown-invocation.html) | [chgrp(1)](https://man.archlinux.org/man/chgrp.1)                                                                                                                                                               |                                                                                                                                                                                |
| [chmod](https://wiki.archlinux.org/title/Chmod "Chmod")         | change file permissions                                        | [chmod(1)](https://man.archlinux.org/man/chmod.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/chmod-invocation.html) |                                                                                                                                                                                                                 |                                                                                                                                                                                |
| [dd](https://wiki.archlinux.org/title/Dd "Dd")                  | convert and copy a file                                        | [dd(1)](https://man.archlinux.org/man/dd.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/dd-invocation.html)          | [#dd alternatives](about:reader?url=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FCore_utilities%23Essentials#dd_alternatives)                                                                                     |                                                                                                                                                                                |
| df                                                              | report file system disk space usage                            | [df(1)](https://man.archlinux.org/man/df.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/df-invocation.html)          | [#df alternatives](about:reader?url=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FCore_utilities%23Essentials#df_alternatives)                                                                                     |                                                                                                                                                                                |
| du                                                              | estimate disk space used by files and directories              | [du(1)](https://man.archlinux.org/man/du.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/du-invocation.html)          | [#du alternatives](about:reader?url=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FCore_utilities%23Essentials#du_alternatives)                                                                                     |                                                                                                                                                                                |
| GNU [tar](https://archlinux.org/packages/?name=tar)             | tar                                                            | tar archiver                                                                                                                             | [tar(1)](https://man.archlinux.org/man/tar.1), [info](https://www.gnu.org/software/tar/manual/html_chapter/index.html)                                                                                          | [archivers](https://wiki.archlinux.org/title/Archiver "Archiver")                                                                                                              |
| GNU [less](https://archlinux.org/packages/?name=less)           | less                                                           | terminal pager                                                                                                                           | [less(1)](https://man.archlinux.org/man/less.1)                                                                                                                                                                 | [terminal pagers](https://wiki.archlinux.org/title/Terminal_pager "Terminal pager")                                                                                            |
| GNU [findutils](https://archlinux.org/packages/?name=findutils) | find                                                           | search files or directories                                                                                                              | [find(1)](https://man.archlinux.org/man/find.1), [info](https://www.gnu.org/software/findutils/manual/html_node/find_html/index.html), [GregsWiki](https://mywiki.wooledge.org/UsingFind "gregswiki:UsingFind") | [#find alternatives](about:reader?url=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FCore_utilities%23Essentials#find_alternatives)                                                |
| GNU [diffutils](https://archlinux.org/packages/?name=diffutils) | diff                                                           | compare files line by line                                                                                                               | [diff(1)](https://man.archlinux.org/man/diff.1), [info](https://www.gnu.org/software/diffutils/manual/html_node/Invoking-diff.html)                                                                             | [#diff alternatives](about:reader?url=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FCore_utilities%23Essentials#diff_alternatives)                                                |
| GNU [grep](https://archlinux.org/packages/?name=grep)           | grep                                                           | print lines matching a pattern                                                                                                           | [grep(1)](https://man.archlinux.org/man/grep.1), [info](https://www.gnu.org/software/grep/manual/html_node/index.html)                                                                                          | [#grep alternatives](about:reader?url=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FCore_utilities%23Essentials#grep_alternatives)                                                |
| GNU [sed](https://archlinux.org/packages/?name=sed)             | sed                                                            | stream editor                                                                                                                            | [sed(1)](https://man.archlinux.org/man/sed.1), [info](https://www.gnu.org/software/sed/manual/html_node/index.html), [one-liners](https://www.pement.org/sed/sed1line.txt)                                      | [sad](https://archlinux.org/packages/?name=sad), [sd](https://archlinux.org/packages/?name=sd)                                                                                 |
| GNU AWK ([gawk](https://archlinux.org/packages/?name=gawk))     | [AWK](https://wiki.archlinux.org/title/AWK "AWK")              | pattern scanning and processing language                                                                                                 | [gawk(1)](https://man.archlinux.org/man/gawk.1), [info](https://www.gnu.org/software/gawk/manual/html_node/index.html), [one-liners](https://www.pement.org/awk/awk1line.txt)                                   | [AWK#Alternative implementations](https://wiki.archlinux.org/title/AWK#Alternative_implementations "AWK")                                                                      |
| [util-linux](https://archlinux.org/packages/?name=util-linux)   | [dmesg](https://en.wikipedia.org/wiki/dmesg "wikipedia:dmesg") | print or control the kernel ring buffer                                                                                                  | [dmesg(1)](https://man.archlinux.org/man/dmesg.1)                                                                                                                                                               | [systemd journal](https://wiki.archlinux.org/title/Systemd_journal#Printing_kernel_messages_only "Systemd journal")                                                            |
| [lsblk](https://wiki.archlinux.org/title/Lsblk "Lsblk")         | list block devices                                             | [lsblk(8)](https://man.archlinux.org/man/lsblk.8)                                                                                        |                                                                                                                                                                                                                 |                                                                                                                                                                                |
| [mount](https://wiki.archlinux.org/title/Mount "Mount")         | mount a filesystem                                             | [mount(8)](https://man.archlinux.org/man/mount.8)                                                                                        |                                                                                                                                                                                                                 |                                                                                                                                                                                |
| [umount](https://wiki.archlinux.org/title/Umount "Umount")      | unmount a filesystem                                           | [umount(8)](https://man.archlinux.org/man/umount.8)                                                                                      |                                                                                                                                                                                                                 |                                                                                                                                                                                |
| [su](https://wiki.archlinux.org/title/Su "Su")                  | substitute user                                                | [su(1)](https://man.archlinux.org/man/su.1)                                                                                              | [sudo](https://wiki.archlinux.org/title/Sudo "Sudo"), [doas](https://wiki.archlinux.org/title/Doas "Doas")                                                                                                      |                                                                                                                                                                                |
| kill                                                            | terminate a process                                            | [kill(1)](https://man.archlinux.org/man/kill.1)                                                                                          | [pkill(1)](https://man.archlinux.org/man/pgrep.1), [killall(1)](https://man.archlinux.org/man/killall.1)                                                                                                        |                                                                                                                                                                                |
| [procps-ng](https://archlinux.org/packages/?name=procps-ng)     | pgrep                                                          | look up processes by name or attributes                                                                                                  | [pgrep(1)](https://man.archlinux.org/man/pgrep.1)                                                                                                                                                               | [pidof(1)](https://man.archlinux.org/man/pidof.1)                                                                                                                              |
| ps                                                              | show information about processes                               | [ps(1)](https://man.archlinux.org/man/ps.1)                                                                                              | [top(1)](https://man.archlinux.org/man/top.1), [system monitors](https://wiki.archlinux.org/title/List_of_applications/Utilities#System_monitors "List of applications/Utilities")                              |                                                                                                                                                                                |
| free                                                            | display amount of free and used memory                         | [free(1)](https://man.archlinux.org/man/free.1)                                                                                          |                                                                                                                                                                                                                 |                                                                                                                                                                                |

### Preventing data loss

`rm`, `mv`, `cp` and shell redirections happily delete or overwrite files without asking. `rm`, `mv`, and `cp` all support the `-i` flag to prompt the user before every removal / overwrite. Some users like to enable the `-i` flag by default using [aliases](https://wiki.archlinux.org/title/Alias "Alias"). Relying upon these shell options can be dangerous, because you get used to them, resulting in potential data loss when you use another system or user that does not have them. The best way to prevent data loss is to create [backups](https://wiki.archlinux.org/title/Backup "Backup").

## Nonessentials

This table lists core utilities that often come in handy.

|Package|Utility|Description|Documentation|Alternatives|
|---|---|---|---|---|
|shell built-ins|[alias](https://wiki.archlinux.org/title/Alias "Alias")|define or display aliases|[alias(1p)](https://man.archlinux.org/man/alias.1p)|
|type|print the type of a command|[type(1p)](https://man.archlinux.org/man/type.1p)|[command(1p)](https://man.archlinux.org/man/command.1p), [whereis(1)](https://man.archlinux.org/man/whereis.1), [which(1)](https://man.archlinux.org/man/which.1)|
|time|time a command|[time(1p)](https://man.archlinux.org/man/time.1p)|
|GNU [coreutils](https://archlinux.org/packages/?name=coreutils)|[tee](https://wiki.archlinux.org/title/Tee "Tee")|read stdin and write to stdout and files|[tee(1)](https://man.archlinux.org/man/tee.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/tee-invocation.html)|[pee(1)](https://man.archlinux.org/man/pee.1)|
|mktemp|make a temporary file or directory|[mktemp(1)](https://man.archlinux.org/man/mktemp.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/mktemp-invocation.html)|
|mknod|create named pipe or device node|[mknod(1)](https://man.archlinux.org/man/mknod.1), [mkfifo(1)](https://man.archlinux.org/man/mkfifo.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/mkfifo-invocation.html#mkfifo-invocation)|
|truncate|shrink or extend the size of a file|[truncate(1)](https://man.archlinux.org/man/truncate.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/truncate-invocation.html#truncate-invocation)|[fallocate(1)](https://man.archlinux.org/man/fallocate.1)|
|basenc|encoding input and output it|[basenc(1)](https://man.archlinux.org/man/basenc.1), [base64(1)](https://man.archlinux.org/man/base64.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/basenc-invocation.html)|
|cut|print selected parts of lines|[cut(1)](https://man.archlinux.org/man/cut.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/cut-invocation.html)|[colrm(1)](https://man.archlinux.org/man/colrm.1), [hck](https://archlinux.org/packages/?name=hck), [choose](https://archlinux.org/packages/?name=choose)|
|tr|translate or delete characters|[tr(1)](https://man.archlinux.org/man/tr.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/tr-invocation.html)|[uconv(1)](https://man.archlinux.org/man/uconv.1)|
|od|dump files in octal and other formats|[od(1)](https://man.archlinux.org/man/od.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/od-invocation.html)|[hexdump(1)](https://man.archlinux.org/man/hexdump.1), [vim](https://wiki.archlinux.org/title/Vim "Vim")'s [xxd(1)](https://man.archlinux.org/man/xxd.1)|
|sort|sort lines|[sort(1)](https://man.archlinux.org/man/sort.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/sort-invocation.html)|
|uniq|report or omit repeated lines|[uniq(1)](https://man.archlinux.org/man/uniq.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/uniq-invocation.html)|[anewer](https://archlinux.org/packages/?name=anewer), [runiq](https://aur.archlinux.org/packages/runiq/)AUR, [huniq-git](https://aur.archlinux.org/packages/huniq-git/)AUR|
|comm|compare two sorted files line by line|[comm(1)](https://man.archlinux.org/man/comm.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/comm-invocation.html)|[zet](https://aur.archlinux.org/packages/zet/)AUR|
|head|output the first part of files|[head(1)](https://man.archlinux.org/man/head.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/head-invocation.html)|
|join|join lines of two inputs on a common field|[join(1)](https://man.archlinux.org/man/join.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/join-invocation.html)|[combine(1)](https://man.archlinux.org/man/combine.1) [zet](https://aur.archlinux.org/packages/zet/)AUR|
|md5sum|calculate cryptography hash functions of inputs and output|[sha256sum(1)](https://man.archlinux.org/man/sha256sum.1), [sha512sum(1)](https://man.archlinux.org/man/sha512sum.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/md5sum-invocation.html)|[shasum(1)](https://man.archlinux.org/man/shasum.1), [rhash(1)](https://man.archlinux.org/man/rhash.1)|
|tail|output the last part of files, or follow files|[tail(1)](https://man.archlinux.org/man/tail.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/tail-invocation.html)|
|wc|print newline, word and byte count|[wc(1)](https://man.archlinux.org/man/wc.1), [info](https://www.gnu.org/software/coreutils/manual/html_node/wc-invocation.html)|
|GNU [binutils](https://archlinux.org/packages/?name=binutils)|strings|print printable characters in binary files|[strings(1)](https://man.archlinux.org/man/strings.1), [info](https://sourceware.org/binutils/docs/binutils/strings.html)|[stringsext](https://aur.archlinux.org/packages/stringsext/)AUR|
|[util-linux](https://archlinux.org/packages/?name=util-linux)|column|columnate file, optionally pretty-printing in table with grid|[column(1)](https://man.archlinux.org/man/column.1)|[paste(1)](https://man.archlinux.org/man/paste.1), [csview](https://aur.archlinux.org/packages/csview/)AUR|
|GNU [findutils](https://archlinux.org/packages/?name=findutils)|xargs|combine or template arguments from stdin to invoke external command|[xargs(1)](https://man.archlinux.org/man/xargs.1)|[parallel(1)](https://man.archlinux.org/man/parallel.1) ([parallel_alternatives(7)](https://man.archlinux.org/man/parallel_alternatives.7))|
|GNU [glibc](https://archlinux.org/packages/?name=glibc)|iconv|convert character encodings|[iconv(1)](https://man.archlinux.org/man/iconv.1)|[recode](https://archlinux.org/packages/?name=recode), [uconv(1)](https://man.archlinux.org/man/uconv.1)|
|GNU [sharutils](https://archlinux.org/packages/?name=sharutils)|uudecode|encode file into email friendly text|[uuencode(1)](https://man.archlinux.org/man/uuencode.1), [uudecode(1)](https://man.archlinux.org/man/uudecode.1), [info](https://www.gnu.org/software/sharutils/manual/html_node/uuencode-Invocation.html)|[uudeview(1)](https://man.archlinux.org/man/uudeview.1)|
|[file](https://archlinux.org/packages/?name=file)|file|guess file type|[file(1)](https://man.archlinux.org/man/file.1)|

The [moreutils](https://archlinux.org/packages/?name=moreutils) package provides useful tools like [sponge(1)](https://man.archlinux.org/man/sponge.1) that are missing from the GNU coreutils.

## Alternatives

Alternative core utilities are provided by the following packages:

- **9base** — A port of various original Plan9 tools to unix.

[https://tools.suckless.org/9base](https://tools.suckless.org/9base) || [9base](https://archlinux.org/packages/?name=9base)

- **[BusyBox](https://wiki.archlinux.org/title/BusyBox "BusyBox")** — Utilities for rescue and embedded systems.

[https://busybox.net](https://busybox.net/) || [busybox](https://archlinux.org/packages/?name=busybox)

- **Heirloom Toolchest** — Traditional implementations of standard Unix utilities.

[https://heirloom.sourceforge.net](https://heirloom.sourceforge.net/) || [heirloom-sh](https://aur.archlinux.org/packages/heirloom-sh/)AUR, [heirloom-doctools](https://aur.archlinux.org/packages/heirloom-doctools/)AUR

- **[sbase](https://en.wikipedia.org/wiki/sbase "wikipedia:sbase")** — A suckless variant of the *nix core utilities.

[https://core.suckless.org/sbase](https://core.suckless.org/sbase) || [sbase-git](https://aur.archlinux.org/packages/sbase-git/)AUR

- **[Toybox](https://en.wikipedia.org/wiki/Toybox "wikipedia:Toybox")** — An all-in-one Linux command line.

[https://landley.net/toybox](https://landley.net/toybox) || [toybox](https://aur.archlinux.org/packages/toybox/)AUR

- **ubase** — An extension of the sbase utilities.

[https://core.suckless.org/ubase](https://core.suckless.org/ubase) || [ubase-git](https://aur.archlinux.org/packages/ubase-git/)AUR

- **uutils** — Cross-platform Rust rewrite of the GNU coreutils.

[https://github.com/uutils/coreutils](https://github.com/uutils/coreutils) || [uutils-coreutils](https://archlinux.org/packages/?name=uutils-coreutils)

### cat alternatives

- **bat** — A cat clone with syntax highlighting and Git integration.

[https://github.com/sharkdp/bat](https://github.com/sharkdp/bat) || [bat](https://archlinux.org/packages/?name=bat)

### cd alternatives

- **autojump** — A faster way to navigate your filesystem from the command line.

[https://github.com/wting/autojump](https://github.com/wting/autojump) || [autojump](https://aur.archlinux.org/packages/autojump/)AUR

- **zoxide** — A smart cd command that learns your habits, allowing you to navigate anywhere in just a few keystrokes.

[https://github.com/ajeetdsouza/zoxide](https://github.com/ajeetdsouza/zoxide) || [zoxide](https://archlinux.org/packages/?name=zoxide)

See also [Bash#Auto "cd" when entering just a path](https://wiki.archlinux.org/title/Bash#Auto_%22cd%22_when_entering_just_a_path "Bash") and [Zsh#Remembering recent directories](https://wiki.archlinux.org/title/Zsh#Remembering_recent_directories "Zsh").

### date alternatives

- **dateutils** — Nifty command line date and time utilities; fast date calculations and conversion in the shell.

[https://www.fresse.org/dateutils/](https://www.fresse.org/dateutils/) || [dateutils](https://archlinux.org/packages/?name=dateutils)

- **pdd** — Tiny datetime diff calculator.

[https://github.com/jarun/pdd](https://github.com/jarun/pdd) || [pdd](https://aur.archlinux.org/packages/pdd/)AUR

### cp alternatives

Using [rsync#As cp/mv alternative](https://wiki.archlinux.org/title/Rsync#As_cp/mv_alternative "Rsync") allows you to resume a failed transfer, to show the transfer status, to skip already existing files and to make sure of the destination files integrity using checksums.

### ls alternatives

- **broot** — A new way to see and navigate directory trees.

[https://github.com/Canop/broot](https://github.com/Canop/broot) || [broot](https://archlinux.org/packages/?name=broot)

- **clifm** — A file manager that can list files like ls(1) would (plus icons and RGB colors support).

[https://github.com/leo-arch/clifm/wiki/Advanced#files-lister-ls-mode](https://github.com/leo-arch/clifm/wiki/Advanced#files-lister-ls-mode) || [clifm](https://aur.archlinux.org/packages/clifm/)AUR

- **eza** — Another ls replacement with color support, tree view, git integration and other features. Based on exa, which is no longer supported.

[https://github.com/eza-community/eza](https://github.com/eza-community/eza) || [eza](https://archlinux.org/packages/?name=eza)

- **lsd** — Modern ls with a lot of pretty colors and awesome icons.

[https://github.com/Peltoche/lsd](https://github.com/Peltoche/lsd) || [lsd](https://archlinux.org/packages/?name=lsd)

### find alternatives

- **fd** — Simple, fast and user-friendly alternative to find. Ignores hidden and `.gitignore`'d files by default.

[https://github.com/sharkdp/fd](https://github.com/sharkdp/fd) || [fd](https://archlinux.org/packages/?name=fd)

- **fuzzy-find** — Fuzzy completion for finding files.

[https://github.com/silentbicycle/ff](https://github.com/silentbicycle/ff) || [ff-git](https://aur.archlinux.org/packages/ff-git/)AUR

- **[plocate](https://wiki.archlinux.org/title/Locate "Locate")** — A much faster locate.

[https://plocate.sesse.net/](https://plocate.sesse.net/) || [plocate](https://archlinux.org/packages/?name=plocate)

- **rawhide** — find files using pretty C expressions.

[https://raf.org/rawhide/](https://raf.org/rawhide/) || [rawhide](https://aur.archlinux.org/packages/rawhide/)AUR

- **uutils-findutils** — Rust rewrite of findutils

[https://github.com/uutils/findutils](https://github.com/uutils/findutils) || [uutils-findutils](https://aur.archlinux.org/packages/uutils-findutils/)AUR

For graphical file searchers, see [List of applications/Utilities#File searching](https://wiki.archlinux.org/title/List_of_applications/Utilities#File_searching "List of applications/Utilities").

### diff alternatives

- **uutils-diffutils** — Rust rewrite of diffutils.

[https://github.com/uutils/diffutils](https://github.com/uutils/diffutils) || [uutils-diffutils](https://aur.archlinux.org/packages/uutils-diffutils/)AUR

While [diffutils](https://archlinux.org/packages/?name=diffutils) does not provide a word-wise diff, several other programs do:

- **cwdiff** — A GNU wdiff wrapper that colorizes the output.

[https://github.com/junghans/cwdiff](https://github.com/junghans/cwdiff) || [cwdiff](https://aur.archlinux.org/packages/cwdiff/)AUR

- **dwdiff** — A word diff front-end for the diff program; supports colors.

[https://os.ghalkes.nl/dwdiff.html](https://os.ghalkes.nl/dwdiff.html) || [dwdiff](https://aur.archlinux.org/packages/dwdiff/)AUR

- [git](https://wiki.archlinux.org/title/Git "Git") diff can do a word diff with `--color-words`, using `--no-index` it can also be used for files outside of Git working trees.
- **git-delta** — A syntax-highlighting pager for git, diff, and grep output.

[https://dandavison.github.io/delta/](https://dandavison.github.io/delta/) || [git-delta](https://archlinux.org/packages/?name=git-delta)

- **icdiff** — A colorized diff tool written in Python. "Improved color diff" is meant to supplement normal diff use.

[https://github.com/jeffkaufman/icdiff](https://github.com/jeffkaufman/icdiff) || [icdiff](https://aur.archlinux.org/packages/icdiff/)AUR

- **wdiff** — A wordwise implementation of GNU diff; does not support colors.

[https://www.gnu.org/software/wdiff/](https://www.gnu.org/software/wdiff/) || [wdiff](https://archlinux.org/packages/?name=wdiff)

See also [List of applications/Utilities#Comparison, diff, merge](https://wiki.archlinux.org/title/List_of_applications/Utilities#Comparison,_diff,_merge "List of applications/Utilities").

### grep alternatives

- **mgrep** — A multiline grep.

[https://sourceforge.net/projects/multiline-grep/](https://sourceforge.net/projects/multiline-grep/) || [mgrep](https://aur.archlinux.org/packages/mgrep/)AUR

- **pdfgrep** — A tool to search text in PDF files.

[https://pdfgrep.org/](https://pdfgrep.org/) || [pdfgrep](https://archlinux.org/packages/?name=pdfgrep)

- **ripgrep-all** — Search in plain text and also in PDFs, E-Books, Office documents, zip, tar.gz.

[https://github.com/phiresky/ripgrep-all](https://github.com/phiresky/ripgrep-all) || [ripgrep-all](https://archlinux.org/packages/?name=ripgrep-all)

#### Code searchers

These tools aim to replace grep for code search. They do recursive search by default, skip binary files and respect `.gitignore`.

- **ack** — A Perl-based grep replacement, aimed at programmers with large trees of heterogeneous source code.

[https://beyondgrep.com/](https://beyondgrep.com/) || [ack](https://aur.archlinux.org/packages/ack/)AUR

- **pcre2grep** — Perl-compatible grep, but it uses the PCRE2 regular expression library.

[https://github.com/PCRE2Project/pcre2](https://github.com/PCRE2Project/pcre2) || [pcre2](https://archlinux.org/packages/?name=pcre2)

- **ripgrep (rg)** — A search tool that combines the usability of ag with the raw speed of grep.

[https://github.com/BurntSushi/ripgrep](https://github.com/BurntSushi/ripgrep) || [ripgrep](https://archlinux.org/packages/?name=ripgrep)

- **The Silver Searcher (ag)** — Code searching tool similar to ack, but faster.

[https://github.com/ggreer/the_silver_searcher](https://github.com/ggreer/the_silver_searcher) || [the_silver_searcher](https://archlinux.org/packages/?name=the_silver_searcher)

- **ugrep (ug)** — Ultrafast grep with interactive user interface, fuzzy search, boolean queries, hexdumps and more.

[https://github.com/Genivia/ugrep](https://github.com/Genivia/ugrep) || [ugrep](https://archlinux.org/packages/?name=ugrep)

See also: [cscope](https://archlinux.org/packages/?name=cscope).

#### Interactive filters

- **fnf** — An interactive fuzzy finder for the terminal.

[https://github.com/leo-arch/fnf](https://github.com/leo-arch/fnf) || [fnf](https://aur.archlinux.org/packages/fnf/)AUR

- **[fzf](https://wiki.archlinux.org/title/Fzf "Fzf")** — General-purpose command-line fuzzy finder, powered by find by default.

[https://github.com/junegunn/fzf](https://github.com/junegunn/fzf) || [fzf](https://archlinux.org/packages/?name=fzf)

- **fzy** — A fast, simple fuzzy text selector with an advanced scoring algorithm.

[https://github.com/jhawthorn/fzy](https://github.com/jhawthorn/fzy) || [fzy](https://archlinux.org/packages/?name=fzy)

- **peco** — Simplistic interactive filtering tool.

[https://github.com/peco/peco](https://github.com/peco/peco) || [peco](https://archlinux.org/packages/?name=peco)

- **percol** — Adds flavor of interactive filtering to the traditional pipe concept of the UNIX shell.

[https://github.com/mooz/percol](https://github.com/mooz/percol) || [percol](https://aur.archlinux.org/packages/percol/)AUR

- **skim** — Fuzzy finder written in Rust, similar to fzf.

[https://github.com/lotabout/skim](https://github.com/lotabout/skim) || [skim](https://archlinux.org/packages/?name=skim)

### dd alternatives

![](https://wiki.archlinux.org/images/1/19/Tango-view-fullscreen.svg)**This article or section needs expansion.**

**Reason:** ddrescue should be added to a list below and briefly described. Given the name similarity, it would be nice to mention the differences from dd_rescue. (Discuss in [Talk:Core utilities](https://wiki.archlinux.org/title/Talk:Core_utilities))

See also: [dd](https://wiki.archlinux.org/title/Dd "Dd") and [ddrescue](https://wiki.archlinux.org/title/Ddrescue "Ddrescue")

#### Alternative dd implementations

This subsection lists _dd_ implementations whose interface and default behaviour is mostly compliant with the POSIX specification of [dd(1p)](https://man.archlinux.org/man/dd.1p).

- **ddpt** — A portable rewrite of [sg_dd(8)](https://man.archlinux.org/man/sg_dd.8) by the SCSI subsystem maintainer of the Linux kernel, with optional but very specialised hardware I/O (SCSI command sets) support, plus many other features.

[https://sg.danny.cz/sg/ddpt.html](https://sg.danny.cz/sg/ddpt.html) || [ddpt](https://aur.archlinux.org/packages/ddpt/)AUR

- **sdd** — A dd implementation portable across UNIX environments by Joerg Schilling, that can checksum the copied data and retry reading bad blocks.

[https://schilytools.sourceforge.net/](https://schilytools.sourceforge.net/) || [schily-tools-sdd](https://aur.archlinux.org/packages/schily-tools-sdd/)AUR

##### Spin-offs of GNU dd

The GNU implementation of _dd_ found in [coreutils](https://archlinux.org/packages/?name=coreutils) also conforms to POSIX. This subsection lists its forks.

- **[dc3dd](https://en.wikipedia.org/wiki/Dd_\(Unix\)#dc3dd "wikipedia:Dd (Unix)")** — Another patched version of GNU dd from the United States Department of Defense Cyber Crime Center (DC3), with similar goals and features to dcfldd.

[https://sourceforge.net/projects/dc3dd/](https://sourceforge.net/projects/dc3dd/) || [dc3dd](https://aur.archlinux.org/packages/dc3dd/)AUR

- **[dcfldd](https://en.wikipedia.org/wiki/Dd_\(Unix\)#dcfldd "wikipedia:Dd (Unix)")** — feature-enhanced fork of GNU dd for forensics and security scenarios, includes on-the-fly hashing capability, flexible wipes, write verification, output to multiple targets at the same time, split and piped output.

[https://dcfldd.sourceforge.net](https://dcfldd.sourceforge.net/) || [dcfldd](https://aur.archlinux.org/packages/dcfldd/)AUR

#### Modernised dd analogues

This subsection lists _dd_ alternatives that do not conform to POSIX (in terms of the JCL-resembling command-line syntax and [default behaviour](https://unix.stackexchange.com/a/192114)).

- **dd_rescue** — A feature-packed, modernised dd analogue that is suitable for daily scripting, disk cloning, and data recovery.

[https://www.garloff.de/kurt/linux/ddrescue/](https://www.garloff.de/kurt/linux/ddrescue/) || [dd_rescue](https://archlinux.org/packages/?name=dd_rescue)

- **rw** — Minimal and portable _dd_ analogue with conventional command-line flags.

[https://sortix.org/rw/](https://sortix.org/rw/) || [rw](https://aur.archlinux.org/packages/rw/)AUR

#### buffer spin-offs

This subsection lists forks of [buffer](https://aur.archlinux.org/packages/buffer/)AUR, a general-purpose I/O buffering utility similar to _dd_ but has a dynamic-sized buffer. It supports blockwise I/O and can be used when dumping from/to an LTO-tape to avoid shoe shining.

- **mbuffer** — Continuation of the _buffer_ utility with threading and other features.

[https://www.maier-komor.de/mbuffer.html](https://www.maier-komor.de/mbuffer.html) || [mbuffer](https://archlinux.org/packages/?name=mbuffer)

### df alternatives

- **duf** — A disk usage/free utility.

[https://github.com/muesli/duf](https://github.com/muesli/duf) || [duf](https://archlinux.org/packages/?name=duf)

### du alternatives

- **cdu** — du wrapper with colors and a pretty histogram.

[http://arsunik.free.fr/prog/cdu.html](http://arsunik.free.fr/prog/cdu.html) || [cdu](https://aur.archlinux.org/packages/cdu/)AUR

- **dua** — Fast **d**isk **u**sage **a**nalyzer, supports deleting files, written in Rust.

[https://github.com/Byron/dua-cli](https://github.com/Byron/dua-cli) || [dua-cli](https://archlinux.org/packages/?name=dua-cli)

- **dust** — A more intuitive version of du, in Rust.

[https://github.com/bootandy/dust](https://github.com/bootandy/dust) || [dust](https://archlinux.org/packages/?name=dust)

- **gdu** — Disk usage analyzer with console interface, written in Go.

[https://github.com/Dundee/gdu](https://github.com/Dundee/gdu) || [gdu](https://archlinux.org/packages/?name=gdu)

- **ncdu** — An extremely lightweight and simple ncurses based disk usage analyzer, written in Zig.

[https://dev.yorhel.nl/ncdu](https://dev.yorhel.nl/ncdu) || [ncdu](https://archlinux.org/packages/?name=ncdu)

See also [List of applications/Utilities#Disk usage display](https://wiki.archlinux.org/title/List_of_applications/Utilities#Disk_usage_display "List of applications/Utilities").

## POSIX shell utilities

Many common packages already install most popular [POSIX utilities](https://pubs.opengroup.org/onlinepubs/9799919799/idx/utilities.html) as dependencies, but the [posix](https://archlinux.org/packages/?name=posix) metapackage can be installed to ensure all of them being always present.

Beside mandatory utilities, there are also metapackages for some of the optional categories:

- [posix-c-development](https://archlinux.org/packages/?name=posix-c-development)
- [posix-software-development](https://archlinux.org/packages/?name=posix-software-development)
- [posix-user-portability](https://archlinux.org/packages/?name=posix-user-portability)
- [posix-xsi](https://archlinux.org/packages/?name=posix-xsi)

**Note** Not all optional utilities from given category are necessarily present in corresponding metapackage.

## Tips and tricks

### Override or add missing coreutils

Some commands (`arch`, `kill`, etc.) are missing from [coreutils](https://archlinux.org/packages/?name=coreutils) or taken from other packages. To complete them for compatibility, install [uutils-coreutils](https://archlinux.org/packages/?name=uutils-coreutils) and do:

```bash
# ln -sf /usr/bin/uu-coreutils /usr/local/bin/arch
# echo -e "#compdef arch=uu-arch\n_uu-arch" > /usr/local/share/zsh/site-functions/_arch
# echo "complete -c arch -w uu-arch" > /usr/local/share/fish/vendor_completions.d/arch.fish
```

## See also

- [GNU Coreutils documentation](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [GNU Coreutils FAQ](https://www.gnu.org/software/coreutils/faq/coreutils-faq.html)
- [Coreutils Gotchas](https://www.pixelbeat.org/docs/coreutils-gotchas.html): GNU coreutils maintainer's notes about some confusing behaviour in coreutils components
- [POSIX utilities](https://pubs.opengroup.org/onlinepubs/9799919799/idx/utilities.html)