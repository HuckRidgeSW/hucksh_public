# hucksh changelog

# 2024-07-31 - 4.0.0

## Bug fixes (existing users: please read!)

- When creating the database, set the umask to 0177, so that it's not readable
  by anyone but the owner (i.e. 0600, or -rw-------).

  Please make sure your hucksh database(s) are only readable by you / the
  people you want to be able to read them.

  You probably want to run this command:

    # Assumes default database name; please change as appropriate.
    chmod go-rwx ~/.hucksh.db*

  everywhere you run the server. You can do this inside hucksh while the
  server is running.

  I'd add code to do this for you, but it's possible that you *want* other
  people or groups to be able to read your database, and while I think that's
  unlikely, I didn't want to break something like that unexpectedly.

## New features

### Save output of running commands when signaled

The hucksh server saves command output only when the command exits. It now
also saves it when the server is signaled, e.g. via ^C or "killall". (But not
"kill -9" of course.)

When signaled, the server waits up to 5s for the save to finish, and then
exits regardless.

(Be advised that when the host machine is shutting down, it might not wait for
the server to finish.)

### EULA & Licensing reworked

- You can now agree to the EULA right in the GUI
- You can include the license text right on the "hucksh license" command line:

  hucksh license -l "{ ... license text ... }"
- When you set a license, it takes effect immediately; you no longer have to
  bounce either the server or the client.

## Other changes

- When sending a command to the server, if the connection drops, add a 5s
  timeout to the reconnection attempt. Otherwise (if the connection drops) the
  whole GUI hangs.
- If the connection drops, reset the gRPC retry timer when the GUI regains
  focus.

  So say the connection drops. You fix it, in a different window. When you
  switch back to the hucksh GUI, it will immediately attempt to reconnect.
- Allow closing the "choose a connection" and "choose a session" windows via
  Cmd-Shift-W (Ctrl-Shift-W on Windows/Linux).
- Tweak the "hucksh run" output to not include trailing spaces on non-blank
  lines.
- On Windows, if you start the GUI and the given server Unix socket address
  (given via -a/--addr) doesn't exist (whether you left the default or set a
  value), open the "choose a connection" dialog.

# 2024-07-12 - 3.1.2

## Bug fixes

Fix a panic on shutdown when installing a license.

"hucksh license" panicked on shutdown after adding the license to the
database, trying to shut down a gRPC server that it hadn't actually started.

# 2024-06-28 - 5th release; 3.1.1

## New features

Nothing hugely new in this release, except for a .pkg file signed (by me) &
notarized (by Apple), for macOS users. Not really a "feature" of the app, as
such.

## Other changes

* Changed to a semver version scheme. Reviewed previous releases and assigned
  them appropriate semantic version numbers, below in this changelog. Did not
  update the released tags on Github.
* Distribute a signed & notarized Apple pkg file in a zip file, instead of an
  unsigned app in a tarball.

  The pkg file worked on my beta-tester's machine. Please let me know if you
  run into any problems!

## Bug fixes

* Make completion work better.

  While working on the pkg file mentioned above, I noticed that e.g. `*pkg`
  wasn't generating the proper completions; it was as if I was completing on
  just `pkg`. So I fixed that and several similar bugs (e.g. with `?`, `:`,
  and other characters).

# 2024-06-19 - 4th release; 3.1.0

## New feature

### Show times

hucksh used to have tooltips that popped up if you hovered the mouse briefly
over command output. These tooltips showed the time of the output, and the
time since the previous line (if it was >= 1ms). But they were kind of wonky
and didn't always go away when you moved the mouse away.

So instead I've removed the tooltips and added a "Toggle Show Times" menu
option (on macOS) & hotkey (everywhere).

The times are displayed on the right side of the window, and include the time
since the previous line, if it was >= 1ms.

Example:
    (cmd:) echo ; date ; sleep 1 ; date ; date ; date ; sleep 2 ; date
                                                          16:44:57.705, 13ms
    Mon Jun 17 16:44:57 EDT 2024                                16:44:57.705
    Mon Jun 17 16:44:58 EDT 2024                        16:44:58.732, 1.027s
    Mon Jun 17 16:44:58 EDT 2024                          16:44:58.742, 10ms
    Mon Jun 17 16:44:58 EDT 2024                          16:44:58.754, 11ms
    Mon Jun 17 16:45:00 EDT 2024                        16:45:00.778, 2.024s

Removing the tooltips and displaying all times for all command output revealed
a bug (now fixed) in how I recorded output times; see below.

Hotkey:
macOS: Cmd-Opt-Shift-T, Linux/Windows: Ctrl-Alt-Shift-T

## Other changes

* Built with Go 1.22.1 on all platforms.
* The "wrap" setting is kept for repeated commands. That is, if you run a
  command, click "Nowrap", and then press Cmd/Ctrl-R to repeat it, the new
  command will also be unwrapped. This makes it easier to re-run commands with
  wide output.
* Fix a server panic when presented with an unknown color in vt10x/hucksh.go,
  Color.ToRealColor. For color numbers > 15, use a 6x6x6 palette similar to
  "web-safe" colors, from the Wikipedia article on `ANSI_escape_codes`.
* Updated mvdan/sh as of 6/17/24.
* Redirect stderr to the logfile, so "panic" output goes there.
* Enhance Home/End/PageUp/PageDown in the directory history ("HIST") tab and
  the command history search panel to set the selected item to the item at the
  top or bottom of the screen, as appropriate.
* Start completion by pressing TAB.
* Completions are now case-insensitive. So c<tab> will complete the same files
  as C<tab>.
* Completions now consider "-" as part of a word. So foo-<tab> invokes
  completions for foo-*.
* Updated to newer Gio. They reworked the event framework again. Please let me
  know if I got focus wrong anywhere, or if any of the hotkeys don't work as
  expected.
* Change the "focused command" box from red to blue. The "wrapped line" bars
  are still red. Commands are drawn a couple of pixels inset so that the focus
  box and the wrapped line bars don't overlap any more.
* The "output time" of the first line of output is compared to the program
  start time when calculating the "time since previous line".

  Before, the time of the first line of output was always considered to be the
  "origin". But if you "sleep 10 ; echo foo", then the "output time" for "foo"
  should show as 10s since the command started.
* In case it matters: Tweaked my build process to use
  "macosx-version-min=11.0" when compiling on macOS.

## Bug fixes

* Home, End, PgUp, PgDn, and Cmd/Ctrl-Shift-C didn't work in Zoomed or Popout
  (popped-out?) windows. Now they do.
* Command output times are now saved correctly in the database.

  Before, if you ran a command, and then hovered the mouse to get the time
  tooltip, you'd get the right time. But if you bounced hucksh and re-read the
  command from the database, every line was recorded as when the program
  ended, not when that line was output.

  This has been fixed.

  Note: Commands are (still) only saved once they finish, they are not
  streamed to the DB live. If the server crashes while a command is running,
  its output won't be saved.
* Fix left/right scrolling in unwrapped command output.
* Cursor position should be more accurate.
* Terminal emulation should be more accurate. E.g. running Vim in hucksh seems
  to work correctly in most cases.

# 2024-03-29 - 3rd release; 3.0.0

## Before you run

* As before, backup your database. This version has a minor database update,
  and while it seems safe to me, I'd hate to be wrong.
* Please replace the server and client at the same time.

## New features

### Change terminal emulator libraries

Switch from Darktile (github.com/liamg/darktile) to vt10x
(github.com/HuckRidgeSW/vt10x, forked from github.com/ActiveState/vt10x).

Related:
* Improve line-wrapping a lot (see below under "Bug fixes")
* Show a cursor in the terminal emulator. Not perfect, but way better than it
  was. I was surprised at how much this improved the perceived usability of
  interactive programs.

### Make hucksh startable from a file browser, icon, or the Mac Dock

(macOS & Linux only)

Make the hucksh GUI auto-start the server.

You used to have to start the hucksh server at the command line, and then
start the GUI from a different command line.

Now (on macOS & Linux) you can just click on the hucksh executable in a file
browser (e.g. on macOS, in the Applications tab of the Finder, or from the
Dock), and the GUI will auto-start the server. If the GUI exits, the server
will continue.

(The Windows version still requires that you start it from a shell. I still
want the GUI to be able to log to the terminal, and to do that, it can't be
what Windows considers a GUI-only executable. I considered shipping two
versions, a GUI-only and a command-line version, but at least for the moment
I'm not doing that. Let me know how you feel about it either way.)

The server runs in "verbose" (`-v`) mode and logs by default to
`$HOME/.hucksh.server.log`. The GUI also runs in "verbose" mode and logs to
`$HOME/.hucksh.gui.log`. You can change where they log using the `--slog` and
`--glog` command-line options (except then you have to run from the
command-line again).

By default, hucksh appends to logs; if you want it to truncate the file (clear
the current contents), add `,trunc` to the end of the filename. You can also
just use `,truc` by itself to use the default logfile, but truncate it first.
Use `-` (e.g. `--glog -`) to log to stdout.

#### Related config variables set up in the database

Since you can start the GUI just by clicking on it now, you need some way to
set some options you used to set from the terminal (to wit, the `-v` setting,
and `$PATH`), so the GUI now reads two settings from the `config` table:

* `client_path` should be set to the same as `$PATH` in your shell
* `client_verbose` can be true/false (or 1/0 or several other values), and
  sets whether the GUI should log in verbose mode (or not)

The huckshrc in the distribution file has a function to make these easier to
set:

```
# Only run this locally.
set-config client_path $PATH
```

## Other changes

* Updated mvdan/sh as of 2024/03/07
  * More mvdan/sh updates: cancellable reader. "read" in the shell now aborts
    if you press the INT button. Not merged into master yet, but running in
    the current version of hucksh.
* Calculate text width a little more accurately on Windows & Linux, so text
  should wrap more accurately.

## Bug fixes

* With Darktile, if you printed a very long line (something that filled the
  whole window, e.g. on macOS if I ran `ps auxww`, the command line of
  crashpad_handler is 8742 characters long), the tty code would hang and use
  100% cpu. Changing to vt10x fixed this.
* If a command printed anything at all (so much as `echo foo`), and then you
  resized the window to be smaller, the output was considered to take up the
  entire width of the window, and the trailing spaces would be line-wrapped.
  Going the other way, so to speak, if you printed a line that was wider than
  the window and it really did line-wrap, and then you enlarged the window,
  the line would not be "unwrapped".

  Both of these have been fixed.
* Display errors (e.g. network errors) correctly across all GUI windows.
  Before, if you started a second window and then closed the first window and
  then the network went down, you'd never see the error.
* When running an interactive program (e.g. Vim), if the server or network
  crashed, and you kept pressing keys, you'd get errors to the log for every
  key. Now just throw those keys away.
* When you start the GUI, if you say `--address ""`, the GUI opens the "choose
  connection" window, as if you'd pressed Cmd-Opt-N/Ctrl-Alt-N. On macOS &
  Linux, it removes the `$HOME/.hucksh.socket.` prefix of all listed sockets,
  so e.g. if you had `.hucksh.socket`, `.hucksh.socket.host`, and
  `.hucksh.socket.vm`, you'd see `local`, `host`, and `vm`. This didn't work
  in Windows, and you'd see e.g. `C:\Users\your_id\.hucksh.socket.host`.
  That's been fixed.
* Log using the right logger. Sometimes I used the default logger, which
  writes to stderr, instead of whatever the rest of the app used.
* Set the Windows drive letter correctly when opening the Hucksh database, so
  that e.g. if your database is in `C:\users\your_id\.hucksh.db`, and you were
  currently in `E:\some\path`, and you started hucksh from there, it can find
  the db correctly.
* Fixed a bug (before it's even been released!) in vt10x where tabstops didn't
  work correctly. Not many programs output tabs, I don't think, but sometimes
  "ls" does, e.g. when doing output in columns.

# 2024-03-01 - 2nd release; 2.0.0

## Before you run

Backup your database. The new version has a minor database update, and while
it seems safe to me, I'd hate to be wrong.

## New features

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
  connection" dialog.
* In the "choose connection" window, allow Cmd/Ctrl-Shift-W to close the
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
* Tweaked the "choose connection" window
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

# 2023-12-07 - Initial release; 1.0.0

vim:comments=fb\:-,fb\:*

