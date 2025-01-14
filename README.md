# Compactor

## A friendly user interface to Windows 10 filesystem compression.

With modern lightweight compression algorithms running at gigabytes per second per core, it's practically a no-brainer to apply them to filesystems to make better use of storage and IO.

Half-recognising this, Windows 10 ships with a reworked compression system that, while fast and effective, is only exposed to users via a command-line tool &mdash; [`compact.exe`].

Compactor is here to plug that gap, with a simple GUI utility anyone can use.

![](https://i.imgur.com/A9si8Zh.png)

The results are quite nice:

| Program | Size | Compacted | Ratio |
|-|-:|-:|-|
| AI War 2 | 2.43 GiB | 1.42 GiB  | 0.59x |
| Big Pharma | 1.1 GiB | 711 MiB | 0.37x |
| Crusader Kings 2 | 2.19 GiB | 1.29 GiB | 0.59x |
| Deus Ex MD | 41.31 GiB | 28.06 GiB | 0.68x |
| Infinifactory | 1.71 GiB | 742 MiB | 0.58x |
| Satisfactory | 15.82 GiB | 10.45 GiB | 0.66x |
| Space Engineers | 16.28 GiB | 9.4 GiB | 0.58x |
| Stellaris | 7.76 GiB | 5.21 GiB | 0.67x |
| Subnautica BZ | 10.62 GiB | 6.40 GiB | 0.60x |
| The Long Dark | 7.42 GiB | 5.64 GiB | 0.76x |
| Microsoft SDKs | 5.91 GiB | 2.45 GiB | 0.41x |
| Visual Studio 2017 | 9.63 GiB | 4.77 GiB | 0.50x |
| Windows Kits | 5.38 GiB | 2.03 GiB | 0.38x |

## Features

### Real-time Progress Updates

Compactor's directory analysis updates as it goes.  You too can experience the satisfaction of watching the disk-space used counter tick down with each file compressed.

### Pause, Resume, Stop

All operations can be paused and interrupted safely at any time.  Compactor will finish off what it's doing and stop, or restart where it left off.

### Compresstimation

Compactor performs a statistical compressibility check on larger files before passing them off to Windows for compaction.  A large incompressible file can be skipped in less than a second instead of tying up your disk for minutes for zero benefit.

### Machine Learning

Using advanced condition-based AI logic, Compactor can skip over files that have been previously found to be incompressible, making re-running Compactor on a previously compressed folder much quicker.

(Yes, it's an if statement and a trivial hash database, hush)

### Scalable and Fast

Written in [Rust], a modern compiled systems programming language from Mozilla, Compactor can cope easily with large folders containing millions of files.

![](https://i.imgur.com/VxyJmgR.png)

## Caveats

### Beta Software

Compactor has received only limited testing.  I've used it extensively on my desktop and laptop without a problem, it's saved me *hundreds of gigabytes* of disk space.  Hopefully it will do the same for you.

**Make backups**.  Report bugs.  Be nice.  You are reminded:

```
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### Permissions

Compactor currently has no mechanism to elevate its privileges using UAC for protected files.  If you're using a limited account you might need to run it as an Administrator, or a user with similar privileges.

Be careful what you compress.  System files should be skipped automatically, and the Windows folder should be in the list of default exclusions (if you want to compact Windows, check out its [CompactOS] feature), but you almost certainly don't want to blindly run this across your entire `C:\` drive.

### Modifiable Files

Compaction is designed for **files that rarely change** &mdash; any modifications result in the file being uncompressed in its entirety.  In fact, simply opening a file in write mode will *hang* until the file is uncompressed, even if no changes are made.

This generally doesn't matter much for application folders, but it's not great for databases, logs, virtual machine images, and various other things that *hopefully* mostly live elsewhere.

If a game uses large files and in-place binary patching for updates, it might be worth adding to the exclusions list.


## Future

These may or may not happen, but have been on my mind.

* Double-check the default exclusions list.  Should be able to do something with the compresstimation code to verify them.
* Examine [overlapped IO], see if we can get more information and control out of the compression process (per-file progress and cancellation).
* Recompression, for changing compression modes without manually decompressing/recompressing.
* Scheduled task or a background service to periodically recompress selected directories.
* Write bindings to Microsoft's [Compression API], add benchmarks for the various compression modes to help users decide which is most appropriate for their system.
* Less rubbish installer.  Why does this involve so much XML oh god.
* Sign the binaries/installer.  If I give away my code I get a free one, right?


## Alternatives

* [`compact.exe`] is a command-line tool that ships with Windows 10.  If you're familiar with the command line and batch files, maybe you'd prefer that. Weirdo.
* [CompactGUI] is a popular Visual Basic program that shells out to `compact.exe` to do its work, instead of using the Windows API directly as Compactor does.  It has some... performance issues, particularly with larger folders.

Are you aware of any others?  Do let me know.


## Nerdy Technical Stuff

Compactor is primarily written in [Rust].  The front-end is basically an embedded website driven by the [web-view] crate and my own fork of the underlying [webview] library.  It does *not* depend on any remote resources or open any ports.

Under the hood it uses [`DeviceIoControl`] with [`FSCTL_SET_EXTERNAL_BACKING`] and [`FSCTL_DELETE_EXTERNAL_BACKING`], and a few functions from [WofApi] (Windows Overlay Filesystem).  This is, of course, in part thanks to the [winapi] crate.  Eventually I hope to get around to finishing off some of my bindings and contributing them back.

Compresstimation uses a simple linear sampling algorithm, passing blocks through LZ4 level 1 as a compressibility check and averaging across the entire file.  The code is [available on Github][compresstimator].

The incompressible-files database is simply an append-only list of SipHash128 path hashes.  It should be safe to share between multiple instances if you want to compress different drives at the same time.  It lives in your `%APPDIR%` Roaming folder under `Freaky\Compactor`.


## Author

Compactor is written by [Thomas Hurst], a nerdy, aloof weirdo from the north-east of England, and a programmer for about 25 years.

He mostly works with FreeBSD and focuses on Unix platforms, but uses Windows because he plays games instead of having a social life.

You can find him on Twitter at [@blaagh], or bug him on IRC as `Freaky` on FreeNode.


[`compact.exe`]: https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/compact
[Rust]: https://www.rust-lang.org/
[CompactGUI]: https://github.com/ImminentFate/CompactGUI
[web-view]: https://github.com/Boscop/web-view
[webview]: https://github.com/Freaky/webview/tree/various-fixes
[`DeviceIoControl`]: https://docs.microsoft.com/en-us/windows/desktop/api/ioapiset/nf-ioapiset-deviceiocontrol
[`FSCTL_SET_EXTERNAL_BACKING`]: https://docs.microsoft.com/en-us/windows-hardware/drivers/ifs/fsctl-set-external-backing
[`FSCTL_DELETE_EXTERNAL_BACKING`]: https://docs.microsoft.com/en-us/windows-hardware/drivers/ifs/fsctl-delete-external-backing
[WofApi]: https://docs.microsoft.com/en-us/windows/desktop/api/wofapi/
[Compression API]: https://docs.microsoft.com/en-gb/windows/desktop/cmpapi/using-the-compression-api
[winapi]: https://github.com/retep998/winapi-rs
[CompactOS]: https://technet.microsoft.com/en-us/windows/dn940129(v=vs.60)
[Thomas Hurst]: https://hur.st/
[@blaagh]: https://twitter.com/blaagh
[overlapped IO]: https://docs.microsoft.com/en-us/windows/desktop/sync/synchronization-and-overlapped-input-and-output
[compresstimator]: https://github.com/Freaky/compresstimator
