# <i>H</i>ydro

> Ultra-pure, lag-free prompt with async Git status. Designed only for [Fish](https://fishshell.com).

[![](https://user-images.githubusercontent.com/56996/103166797-f807ee00-4868-11eb-9818-c661584274c8.gif)](#hydro)

## Installation

Install with [Fisher](https://github.com/jorgebucaran/fisher):

```console
fisher install kriansa/hydro
```

> This is a fork of [jorgebucaran/hydro](https://github.com/jorgebucaran/hydro). See
> [Fork changes](#fork-changes) for what differs from upstream.

## Features

One prompt symbol to rule them all. [Change it](#configuration)?

<pre>
<b>~</b> ❱ ⎢
</pre>

Display Git branch name and status—prompt repaints asynchronously! ✨

<pre>
~/p/<b>hydro</b> main ❱ touch Solution
~/p/<b>hydro</b> main• ❱ ⎢
</pre>

> `•` indicates that there are staged, unstaged or untracked files.

Display how many commits ahead and/or behind you are of your upstream—prompt repaints asynchronously!

<pre>
~/p/<b>hydro</b> main• ↓2 ❱ git commit -am Hotfix
~/p/<b>hydro</b> main ↑1 ↓2 ❱ git pull --rebase && git push
~/p/<b>hydro</b> main ❱ ⎢
</pre>

Display [`$CMD_DURATION`](https://fishshell.com/docs/current/language.html?highlight=cmd_duration#envvar-CMD_DURATION) when > `1` second. [Configurable](#configuration).

<pre>
~/p/<b>hydro</b> main ❱ git push --quiet
~/p/<b>hydro</b> main 1.1s ❱ ⎢
</pre>

Display the last non-zero [exit status](https://fishshell.com/docs/current/tutorial.html#exit-status) (or statuses) using [`$pipestatus`](https://fishshell.com/docs/current/language.html?highlight=cmd_duration#envvar-pipestatus).

<pre>
~/p/<b>hydro</b> main ❱ false
~/p/<b>hydro</b> main | <b>1</b> ❱ ⎢
~/p/<b>hydro</b> main ❱ true | false | false
~/p/<b>hydro</b> main | <b>0</b> <b>1</b> <b>1</b> ❱ ⎢
</pre>

Truncate [`$PWD`](https://fishshell.com/docs/current/language.html?highlight=cmd_duration#envvar-PWD) segments except for the basename and root of Git repos.

<pre>
<b>~</b> ❱ projects/hydro/
~/p/<b>hydro</b> ❱ functions/share/
~/p/hydro/f/<b>share</b> ❱ ⎢
</pre>

Display the current bindings mode.

<pre>
<i>I</i> <b>~</b> ❱ <kbd>Esc</kbd>
<i>N</i> <b>~</b> ❱ <kbd>R</kbd>
<i>R</i> <b>~</b> ❱ ⎢
</pre>

## Performance

Blazing fast would be an understatement considering that the [LLVM repo](https://github.com/llvm/llvm-project) has over 375,000 commits!

<pre>
~/<b>llvm-project</b> main ❱ time fish_prompt
~/<b>llvm-project</b> main ❱
________________________________________________________
Executed in   79.00 micros    fish           external
   usr time   71.00 micros   71.00 micros    0.00 micros
   sys time    9.00 micros    9.00 micros    0.00 micros
</pre>

## Configuration

Modify variables using `set --universal` from the command line or `set --global` in your `config.fish` file.

### Symbols

| Variable                  | Type   | Description                     | Default |
| ------------------------- | ------ | ------------------------------- | ------- |
| `hydro_symbol_start`      | string | Prompt start symbol.            |         |
| `hydro_symbol_prompt`     | string | Prompt symbol.                  | `❱`     |
| `hydro_symbol_git_dirty`  | string | Dirty repository symbol.        | `•`     |
| `hydro_symbol_git_ahead`  | string | Ahead of your upstream symbol.  | `↑`     |
| `hydro_symbol_git_behind` | string | Behind of your upstream symbol. | `↓`     |

### Colors

> Any argument accepted by [`set_color`](https://fishshell.com/docs/current/cmds/set_color.html).

| Variable               | Type  | Description                    | Default              |
| ---------------------- | ----- | ------------------------------ | -------------------- |
| `hydro_color_pwd`      | color | Color of the pwd segment.      | `$fish_color_normal` |
| `hydro_color_git`      | color | Color of the git segment.      | `$fish_color_normal` |
| `hydro_color_start`    | color | Color of the start symbol.     | `$fish_color_normal` |
| `hydro_color_error`    | color | Color of the error segment.    | `$fish_color_error`  |
| `hydro_color_prompt`   | color | Color of the prompt symbol.    | `$fish_color_normal` |
| `hydro_color_duration` | color | Color of the duration section. | `$fish_color_normal` |
| `hydro_color_jobs`     | color | Color of the background jobs segment. | `$fish_color_normal` |

### Flags

| Variable          | Type    | Description                                  | Default |
| ----------------- | ------- | -------------------------------------------- | ------- |
| `hydro_fetch`     | boolean | Fetch git remote in the background.          | `false` |
| `hydro_multiline` | boolean | Display prompt character on a separate line. | `false` |

### Misc

| Variable                       | Type    | Description                                                                                                              | Default |
| ------------------------------ | ------- | ------------------------------------------------------------------------------------------------------------------------ | ------- |
| `fish_prompt_pwd_dir_length`   | numeric | The number of characters to display when path shortening. Set it to `0` to display only the topmost (current) directory. | `1`     |
| `hydro_ignored_git_paths`      | strings | Space separated list of paths where no git info should be displayed.                                                     | `""`    |
| `hydro_cmd_duration_threshold` | numeric | Minimum command duration, in milliseconds, after which command duration is displayed.                                    | `1000`  |

## Fork changes

Everything below is specific to this fork. Forked from upstream at `f130b55`.

### The Git segment no longer travels through a universal variable

Upstream reads the repository in a background job and passes the result back to the
shell by setting a universal variable named after the shell's process id. Fish saves
every universal variable to `fish_variables`, so a shell running this prompt keeps
rewriting that file for as long as it lives. That is invisible in a normal setup, but
it makes `fish_variables` unusable as a versioned file, and shells that are killed
outright leave their entry behind forever.

This fork passes the result through a file private to the shell, at
`$XDG_RUNTIME_DIR/hydro-<pid>` (falling back to `$TMPDIR`, then `/tmp`). The
background job writes the file and sends `SIGUSR1`; the shell reads it and repaints.
No universal variable is ever written. The file is removed when the shell exits and on
`hydro_uninstall`, and it is truncated at startup so a recycled process id cannot
inherit a previous shell's branch.

The variable holding the rendered segment is `$_hydro_git_value`, replacing upstream's
`$$_hydro_git` indirection. A prompt that renders the Git segment must use the new
name.

A shell killed with `SIGKILL` leaves a small file behind, in a directory the operating
system already cleans.

### The path segment is no longer rebuilt on every prompt

Upstream rebuilds the path segment whenever `fish_prompt_pwd_dir_length` changes, so a
new value is picked up immediately. Fish's default `fish_title` calls `prompt_pwd` on
every prompt, and `prompt_pwd` assigns that same variable as a **local**, which fish
reports to the handler exactly like a real change. Rebuilding the segment runs
`git rev-parse`, so upstream waits on a git process before each prompt can appear.

A local assignment cannot be distinguished from a global one inside the handler, so
this fork stops watching that variable and reads it when the path is built instead.
The path is rebuilt on `cd` and when `hydro_ignored_git_paths` changes.

Consequence: changing `fish_prompt_pwd_dir_length` in a running shell takes effect on
the next `cd` rather than immediately. Setting it in `config.fish` behaves as before.

### Background jobs segment

Displays the number of running background jobs, excluding the prompt's own Git job.
Colored with [`hydro_color_jobs`](#colors).

<pre>
~/p/<b>hydro</b> main ❱ sleep 30 &
~/p/<b>hydro</b> main <b>[1]</b> ❱ ⎢
</pre>

### Blank line between commands

When [`hydro_multiline`](#flags) is enabled, a blank line is printed above each
prompt, skipping the first prompt of the session so a new shell does not open with a
stray empty line.

## License

[MIT](LICENSE.md)
