# Dotfiles - Origin and History
The origin of dotfiles is due to a bug in `ls`<sup>[1]</sup>. Unix creators wanted
`ls` to hide `.` and `..` but due to a bug in `ls`, it hid all the files
and directories starting with a dot.
Many people exploited this bug and it became a "feature".

## Side note
As a side note, in Plan 9, dotfiles were completely removed and replaced
by a sane alternative. Every configuration file (and other files you would hide)
in Plan 9 is stored in `$HOME/lib` akin to `$HOME/.config`.

# Freedesktop's XDG environment variables
Freedesktop's XDG env vars<sup>[2]</sup> like `XDG_CACHE_HOME`,
`XDG_CONFIG_HOME` and many more is a key to keep
your home directory tidy because nine out of ten
software follows them. If they don't follow those
conventions, most of them have a flag to change
paths the software use for something.

Make sure to set them as they tend to drastically reduce
the clutter in your home directory.

# More env vars
A lot of Unix tools are notorious for creating dotfiles and cluttering your
home directory (wget, less for example). But most of these read an env var
that points to the path of the configuration file or has a flag that
changes some kind of path. Some of these and their
env var/flag will be listed below for your benefit. If something that you use
isn't here, check the man page and make sure to make an issue so people
in the future can benefit too!

| Tool                  | Env var                       | Flag                      |
|-----------------------|-------------------------------|---------------------------|
| less                  | `LESSHISTFILE`                | -                         |
| gnupg                 | `GNUPGHOME`                   | -                         |
| readline              | `INPUTRC`                     | -                         |
| ksh<sup>*</sup>       | `ENV`                         | -                         |
| zsh<sup>*</sup>       | `ZDOTDIR`                     | -                         |
| dialog                | `DIALOGRC`                    | -                         |
| mail                  | `MAIL`, `MAILRC`              | -                         |
| shell-history         | `HISTFILE`                    | -                         |
| xauth                 | `XAUTHORITY`                  | -                         |
| xinit                 | `XINITRC`                     |                           |
| python                | `PYTHONPATH` `PYTHONUSERBASE` | -                         |
| pulseaudio            | `PULSE_COOKIE`                | -                         |
| GIMP                  | `GIMP2_DIRECTORY`             | -                         |
| pass                  | `PASSWORD_STORE_DIR`          | -                         |
| dunst                 | -                             | `-conf`                   |
| picom                 | -                             | `--config`                |
| sxhkd                 | -                             | `-c`                      |
| wget                  | -                             | `--no-hsts` `--hsts-file` |

- If you're a mksh user, when you compile, you can change the `MKSHRC_PATH` definition
- If you're a zsh user, you can set `ZDOTDIR` in `/etc/zsh/zshenv`<sup>[3]</sup>
- About .Xfiles - https://nixers.net/showthread.php?tid=2271
- As a last resort, you *can* manually edit `/usr/bin/startx` to use an `.xinitrc` (or any other name) file of your choice to execute when running `xinit`

# Workarounds
These are merely workarounds and should be avoided whenever possible
but still these are worth it as a last ditch effort when the software
that creates these pesky dotfiles is proprietary or if the compilation
time is large.

A simple way to shove all the dotfiles to a directory you desire is
to change the value of `HOME` env var at launch. A simple wrapper
like the following should do the job.

```
#!/bin/sh
HOME=${HOME}/some_other_dir firefox
```

If you want to change the directory Xfce apps use, then launch
`xfconfd` with the changed `HOME` directory. This is likely located
at `/lib/xfce4/xfconf/xfconfd`

Sometimes that simple trick may not work. So you have
to create another user and the software dumps all its file in the newly created
user's home directory instead. This is a rare case and most software
shouldn't need this.

# `rm` Daemon

Sometimes you can still have certain directories continue to pop up in your home folder, this can be taken care of quite easily using an "`rm` daemon". Here is an example of one I named `ch` that removes a few pesky folders for me:

```sh
#!/bin/sh

[ "$(pgrep -x ch)" = "$$" ] || exit 1

while :; do
  set --
  for i in "$HOME/.pki" \
	   "$HOME/.w3m" \
	   "$HOME/.bash*" \
	   "$HOME/.config" \
	   "$HOME/.lyrics" \
	   "$HOME/.xournalpp"; do
    [ -e "$i" ] && set -- "$@" "$i"
  done
  [ "$1" ] && rm -rf -- "$@"
  sleep 60
done
```

You can adjust the value of the `sleep` command at the end of the script to change the rate at which it cleans your home directory. Ideally you would want to execute this in either your window managers' startup file, `xinitrc`, or the equivalent for your display protocol.

# Patching
What if the software you're using doesn't have an env var and you do not want to use
ugly workarounds? Well, you can patch the software. Patching
software (especially when you're simply changing a variable) is simple and easy.
Sometimes, you might not even need to change the source code because
some software can set the configuration path (or whatever else) at
compile time. So instead of changing the source code, you have to change a
variable in the Makefile.

You can find the patches that I made for various software (mksh, emacs, dosbox) to change
the configuration (or whatever else) path here -
1. https://github.com/vizs/home/tree/master/etc/prog.d/patches
2. https://github.com/vizs/ports

## A rough outline of the process
`grep` and a bit of time is all you need to change the path. In most cases, all
you need to do is search for a string like `.software` in every file in the
source directory and change it. You can do this by simply running
`grep -R \.software` (assuming you're in the source directory). Then using your
favorite editor, change the necessary lines.

# Structure
Reducing the number of dotfiles is not the only
thing that matters to have an organized home directory.
A proper structure is the key to organization.

Decide which files go where and start enforcing your
rules to the files you have by moving them, declaring
env vars, etc.

Related read -> https://nixers.net/showthread.php?tid=1293

# References
[1] http://xahlee.info/UnixResource_dir/writ/unix_origin_of_dot_filename.html

[2] https://specifications.freedesktop.org/basedir-spec/latest/ar01s03.html

[3] https://github.com/vizs/declutter-home/issues/1
