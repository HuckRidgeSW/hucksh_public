# hucksh changelog

# 2024-03-01 - 2nd release

## Before you run

Backup your database. The new version has a minor database update, and while
it seems safe to me, I'd hate to be wrong.

## New Features

### GUI client works on Windows

The GUI client now works on Windows. Point it at a macOS or Linux server, via
ssh or "hucksh forward" (about which see below).

It works with a Windows-native Unix socket (as created by "hucksh forward"),
or a Cygwin emulated Unix socket (as created by Cygwin ssh). (The native
Windows ssh clients don't do Unix socket forwarding at all.)

(I'm working on a server for Windows. It's surprisingly (or maybe not so
surprisingly) tricky.)

### "forward" subcommand

Forward a local Unix socket to a remote system via an internal ssh client.
Mostly for Windows; other platforms don't need it. See "hucksh forward --help"
for more.

### Public forums

Created github.com/huckridgesw/hucksh_public to host public discussions and
user-entered bug reports.

### Migration to huckridge.com

I copied all my Notion pages to huckridge.com. See https://huckridge.com/hucksh/.

## Smaller features

### GUI

* Ctrl-Shift-Alt-N (open a new window) now works on Linux & Windows.
* If you pass an explicit empty string ("") as the --address, open the "choose
  a connection" dialog.
* In the "choose a connection" window, allow Cmd/Ctrl-Shift-W to close the
  window, as well as the Cmd/Ctrl-W that already worked. This makes that
  window more like the regular GUI shell window, which already supports
  Cmd/Ctrl-Shift-W to close the window.

### Server

* The server now deletes the Unix socket it was listening on when it exits.
* Listens for ^C/SIGINT and does a graceful shutdown. (If you tee the server
  output into a file, be sure to use "tee -i".)
* Duplicate the "-s" and "-d" options into the "root" hucksh command, so that
  you can "hucksh -s session_name" and it'll work like "hucksh gui -s
  session_name", or (for the server-only hucksh.server linux executable) you
  can run "hucksh -d database_name" and it'll work like "hucksh serve -d
  database_name".

## Bug fixes

* In the database, don't add the same command to a single tab multiple times
  (e.g. via "hucksh tab --add"). Clean it up via a database migration if it
  has happened already.
* Several data races fixed.
* Fixed a problem where if an internal channel could not be written to for too
  long, the app panicked. Now the write happens asynchronously, and forks to a
  goroutine if it takes too long. (And prints a full stack trace. Please send
  it to me if you encounter this problem. In my own usage, I've never yet had
  :t actually take "too long".)
* GUI: Fixed a deadlock during command output processing.
* "hucksh subscribe" prints blank lines correctly now.
* If you used the "running" function in huckshrc, and there was nothing
  actually running, fixed the server thinking it had hit a database error.
* Fix "remote" checking in huckshrc. If you have the client running in two
  places, then (from the point of view of the server) being "remote" can
  change from one command to the next. So check it each time, instead of
  setting it once when you run .huckshrc.
* Allow selecting command text after "zooming" a command.
* Fixed some display bugs.
* Fixed a panic when "removing" a tall command (one larger than the display
  window).

## Other changes

* Changed window title text from "HuckSh" to "Hucksh".
* Introduced a database migration tool internally.
* Tweaked the "choose a connection" window
  * added a "choose a connection" label, an inset around the window edge, and
    a small space above the "Rescan" button.
  * Removed the "$HOME/.hucksh.socket" prefix from the displayed list of
    connections, leaving just the hostnames.
  * Once you've chosen a connection and the GUI is reading history, show
    current tab being loaded, total number of tabs, and the current count of
    commands loaded.
* Add "refresh-tab" to huckshrc
* Use a single DB connection for writing, and multiple connections for
  reading.
* Set the DB timeout to 5s.
* Change to using SQLite "WAL" (write ahead log) mode.
* At GUI startup, when reading history from the server, read it in larger
  chunks (instead of line by line). Makes loading history much faster.
* Linux GUI: Add a grey box around the edge of the window, so it doesn't blend
  in so well with other white windows. Windows and macOS do window shadows, so
  that's less of a problem on those platforms.
* On GUI startup, track completion of old commands better, so we don't start a
  goroutine per command to read "ongoing" output from commands that've already
  finished.
* GUI: In the command widget, only look for Cmd-Up/Down on macOS (that is,
  don't look for Ctrl-Up/Down on Windows & Linux). It turns out the editor
  widget sometimes (but not always!) uses Ctrl-Up/Down itself. Use Ctrl-P/N
  (Previous / Next) instead.
* Print errors returned from deeper in the app at the command line.
* Added a couple more functions to huckshrc. Added a lot more comments.
* Updated the help text on many subcommands.
* Removed several TLS-related command-line arguments that weren't supported.
* When adding a command to a tab (e.g. via "hucksh tab --add"), or replacing a
  command in a tab, if there's a database error, panic, so as not to leave the
  server, client, and user in an inconsistent state, where the user thinks a
  command has run and been added to the tab history, and it looks like it has
  been, but it really hasn't. See the comment on reset-tab-counts in huckshrc
  for more details. This will be fixed in a future release. 
* Use "fyne" (https://github.com/fyne-io/fyne/tree/master/cmd/fyne) to package
  Windows version (hucksh.exe), which embeds the hucksh
  icon in the .exe, which is nice. Uses a locally-modified version of fyne
  that does not use "-H windowsgui".

# 2023-12-07 - Initial release
