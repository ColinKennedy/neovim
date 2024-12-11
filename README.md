I didn't add everything because of some reason or another. If curious, please
see the section below

<details>
<summary>Some type annotations should use `---@generic`</summary>

LuaLS supports "generic" types. See
https://github.com/LuaLS/lua-language-server/wiki/Annotations#generic

For example `:h deepcopy` takes argument `deepcopy({expr} [, {noref}])` and
returns a copy of it. So the function should ideally not return `any` like it
currently does but instead return the type of whatever `{expr}` is.

`scripts/gen_eval_files.lua` would probably need to be updated to handle that
and I considered it out of scope for this PR.

Here some functions that could benefit from `---@generic`.

- `:h deepcopy`
- `:h get`
- `:h items`
- `:h keys`
- `:h max`
- `:h min`
- `:h reduce`
- `:h sort`
- `:h values`

A future PR for these would be good.
</details>

<details>
<summary>Unsure how to handle primitive literals</summary>

A number of functions have explicit return types like "returns 0 on success or -1 on
failure". (Many) others are "return 0 as FALSE and 1 as TRUE". Some do the opposite. And
rare functions even have non-specific returns like `:h win_splitmove` that returns 0 for
success but non-zero for failure (there's no specificly defined failure value)

Anyway I was inclined to change the return values to be literal like
`---@returns 0 | 1` where-appropriate but I didn't want to assume anything. So
I opted out on this PR

For reference, these are the functions I found so far:

- `did_filetype` - 0 == FALSE, 1 == TRUE
- `inputrestore` - "Returns TRUE when there is nothing to restore, FALSE otherwise."
    - TRUE means "did nothing" and FALSE means "did something"
- `inputsave` - "Returns TRUE when out of memory, FALSE otherwise."
    - This means that unlike the others, TRUE actually is a fail condition
- `jobstart` - 0 / -1 == failure, 1-or-more if success
    - I guess in this one case `integer` would actually be correct
- `mkdir` - 0 == FALSE, 1 == TRUE
- `serverstop` - 0 == FALSE, 1 == TRUE
- `setbufline` - 1 == fail, 0 == success
- `setqflist` - 0 == success -1 == failure
- `wildmenumode` - 0 == FALSE, 1 == TRUE
- `win_move_separator` - 0 == FALSE, 1 == TRUE
- `win_move_statusline` - 0 == FALSE, 1 == TRUE
- `win_splitmove` - "Returns zero for success, non-zero for failure."
- `writefile` - 0 == success -1 == failure
</details>

<details>
<summary>Recommended To Use Custom Types</summary>

Some functions could use a proper `---@class` definition. For example `:h
vim.api.nvim_get_hl()` returns `vim.api.keyset.get_hl_info`.

I assumed I would define those in `vim/_meta/api_keysets_extra.lua` but didn't
know if there were preferred naming conventions. So I figured a follow-up PR for
that would be best.

These are the functions that could use custom class types, assuming an
equivalent doesn't already exist:

- `:h getcellwidths()`
- `:h getcursorcharpos()`
- `:h getmatches()`
- `:h gettagstack()`
- `:h menu_get()`
- `:h setcellwidths()`
- `:h taglist()`
- `:h timer_info()`
- `:h win_id2tabwin()`
- `:h win_screenpos()`
- `:h wordcount()`
</details>

<details>
<summary>Not Sure</summary>

All situations where I wasn't sure of the type (and an explanation of why)

- `digraph_get` - technically the type is char but lua has only string. So I guess just
  string?

--- @return anytttt
function vim.fn.funcref(name, arglist, dict) end

`sqrt()` can return a number or NaN. Would we define a fake NaN class and
include that as a return type. e.g. `number | NaN`?

</details>


```
make doc
```

```
```

## Needs Custom Types
mkdir
 - technically TRUE or FALSE (0 or 1)

serverstop
 - technically TRUE or FALSE (0 or 1)

```
--- Returns |TRUE| when the wildmenu is active and |FALSE|
--- @return anytttt
function vim.fn.wildmenumode() end
```
--- @return anytttt
function vim.fn.setbufline(buf, lnum, text) end
 - returns = 'integer',
 - technically int but really is a bool? 0-1

setqflist
 - 0 success -1 failure

```
--- @return anytttt
function vim.fn.wordcount() end
```

```
--- When the write fails -1 is returned, otherwise 0.  There is an
--- error message if the file can't be created or when writing
--- fails.
writefile
```

--- @param expr any
--- @param filename? string
--- @return anytttt
function vim.fn.taglist(expr, filename) end
echo taglist("asdfsf", "asdffsd")
 - has some weird list-of-dict type

```
technically a bool
--- 'winminwidth'). Returns TRUE if the window can be found and
--- FALSE otherwise.
--- This will fail for the rightmost window and a full-width
--- window, since it has no separator on the right.
--- Only works for the current tab page. *E1308*
---
--- @param nr integer
--- @param offset integer
--- @return anytttt
function vim.fn.win_move_separator(nr, offset) end
```
```
technically bool
function vim.fn.win_move_statusline(nr, offset) end
```
```
tuple
--- Return the screen position of window {nr} as a list with two
--- numbers: [row, col].  The first window always has position
--- [1, 1], unless there is a tabline, then it is [2, 1].
--- {nr} can be the window number or the |window-ID|.  Use zero
--- for the current window.
--- Returns [0, 0] if the window cannot be found.
---
--- @param nr integer
--- @return anytttt
function vim.fn.win_screenpos(nr) end
```

```
literals?
--- Returns zero for success, non-zero for failure.
---
function vim.fn.win_splitmove(nr, target, options) end
```

```
tuple int
--- Return a list with the tab number and window number of window
--- with ID {expr}: [tabnr, winnr].
--- Return [0, 0] if the window cannot be found.
---
--- @param expr integer
--- @return anytttt
function vim.fn.win_id2tabwin(expr) end
```

jobstart
 - returns 0, or -1, 1-or-more if success

function vim.fn.timer_info(id) end
 - returns dict

```
function vim.fn.getcursorcharpos(winid) end
getcurpos
- needs custom type
```

function vim.fn.menu_get(path, modes) end

```
function vim.fn.getcellwidths(list) end
- technically is a hex[][] - not sure how to type that

function vim.fn.setcellwidths(list) end
- technically is a hex[][] - not sure how to type that
```

function vim.fn.getmatches(win) end

function vim.fn.gettagstack(winnr) end



## Needs Overrides And/Or Custom Types
function vim.fn.getloclist(nr, what) end

```
function vim.fn.get(list, idx, default) end

--- Get byte {idx} from |Blob| {blob}.  When this byte is not
--- available return {default}.  Return -1 when {default} is
--- omitted.
---
--- @param blob string
--- @param idx integer
--- @param default? any
--- @return anytttt
function vim.fn.get(blob, idx, default) end

--- Get item with key {key} from |Dictionary| {dict}.  When this
--- item is not available return {default}.  Return zero when
--- {default} is omitted.  Useful example: >vim
---   let val = get(g:, 'var_name', 'default')
--- <This gets the value of g:var_name if it exists, and uses
--- "default" when it does not exist.
---
--- @param dict table<string,any>
--- @param key string
--- @param default? any
--- @return anytttt
function vim.fn.get(dict, key, default) end

--- Get item {what} from |Funcref| {func}.  Possible values for
--- {what} are:
---   "name"    The function name
---   "func"    The function
---   "dict"    The dictionary
---   "args"    The list with arguments
---   "arity"   A dictionary with information about the number of
---       arguments accepted by the function (minus the
---       {arglist}) with the following fields:
---     required    the number of positional arguments
---     optional    the number of optional arguments,
---           in addition to the required ones
---     varargs     |TRUE| if the function accepts a
---           variable number of arguments |...|
---
---     Note: There is no error, if the {arglist} of
---     the Funcref contains more arguments than the
---     Funcref expects, it's not validated.
---
--- Returns zero on error.
---
--- @param func function
--- @param what string
--- @return any
function vim.fn.get(func, what) end

--- @param buf? integer|string
--- @return vim.fn.getbufinfo.ret.item[]
function vim.fn.getbufinfo(buf) end
```

--- @return anytttt
function vim.fn.getqflist(what) end

--- @return anytttt
function vim.fn.gettabinfo(tabnr) end

function vim.fn.getwinpos(timeout) end

function vim.fn.glob(expr, nosuf, list, alllinks) end
- Needs two overrides, maybe? Not sure if I even can

function vim.fn.globpath(expr, nosuf, list, alllinks) end
- Needs two overrides, maybe? Not sure if I even can


Needs 2 overrides and each have different types
function vim.fn.items(dict) end
 - returns

## Don't know
function vim.fn.timer_stop(timer) end

<details>
<summary>Ignored</summary>
I skipped over the `rpc*` functions. Same with `job*`. And deprecated functions.
  I don't know enough about those to be confident in typing them.

</details>

## Ignored
setcursorcharpos
rpc* stuff
function vim.fn.winrestview(dict) end

setcellwidths
setcharpos
setcharsearch
function vim.fn.jobsend(...) end
function vim.fn.jobresize(job, width, height) end
```
--- @deprecated
--- Obsolete name for |hlID()|.
---
--- @param name string
--- @return anytttt
function vim.fn.highlightID(name) end

--- @deprecated
--- Obsolete name for |hlexists()|.
---
--- @param name string
--- @return anytttt
function vim.fn.highlight_exists(name) end


--- @deprecated
--- Use |input()| instead.
---
--- @param ... any
--- @return anytttt
function vim.fn.inputdialog(...) end


-Not sure for this one
--- @return anytttt
function vim.fn.inputlist(textlist) end

--- @deprecated
--- Obsolete name for |chanclose()|
---
--- @param ... any
--- @return anytttt
function vim.fn.jobclose(...) end

--- @deprecated
--- Obsolete name for bufnr("$").
---
--- @return anytttt
function vim.fn.last_buffer_nr() end
```

## Bool-y
```
--- @return anytttt
vim.fn.inputrestore()
vim.fn.inputsave()
```

## Skipped Because It's Hard
function vim.fn.timer_start(time, callback, options) end
reltime
function vim.fn.reltimefloat(time) end
reltimestr

mode - not sure wht to do about that one

range

sinh
 - it returns a number or inf, not sure what to do about that

function vim.fn.searchpos(pattern, flags, stopline, timeout, skip) end
function vim.fn.searchdecl(name, global, thisblock) end
function vim.fn.searchcount(options) end
function vim.fn.search(pattern, flags, stopline, timeout, skip) end
function vim.fn.screenpos(winid, lnum, col) end

readdir
function vim.fn.prompt_getprompt(buf) end
prompt_* stuff
function vim.fn.pum_getpos() end
pumvisible

function vim.fn.range(expr, max, stride) end
function vim.fn.rand(expr) end

function vim.fn.match(expr, pat, start, count) end
function vim.fn.mapset(dict) end
function vim.fn.mapset(mode, abbr, dict) end
function vim.fn.mapnew(expr1, expr2) end
function vim.fn.mapcheck(name, mode, abbr) end
function vim.fn.map(expr1, expr2) end
function vim.fn.insert(object, item, idx) end
function vim.fn.libcallnr(libname, funcname, argument) end
function vim.fn.libcall(libname, funcname, argument) end
function vim.fn.matchaddpos(group, pos, priority, id, dict) end
All the match* stuff

function vim.fn.menu_info(name, mode) end
function vim.fn.msgpackdump(list, type) end
function vim.fn.msgpackparse(data) end

'or'
function vim.fn.perleval(expr) end


## Special
virtcol
 - needs two signatures

visualmode
 - probably returns specific string types


```
--- Returns a status integer:
---   0 if the condition was satisfied before timeout
---   -1 if the timeout was exceeded
---   -2 if the function was interrupted (by |CTRL-C|)
---   -3 if an error occurred
function vim.fn.wait(timeout, condition, interval) end
```

reltime

invert should be an int, number is not allowed

--- @return anytttt
function vim.fn.list2blob(list) end
 - custom blob type?

function vim.fn.nr2char(expr, utf8) end
- technically it's a char?

readblob
 - technically needs blobl type? Or just string? check

setcmdline
 - technically bool?

setcmdpos
 - technically bool?

function vim.fn.setqflist(list, action, what) end
 - returns = 'integer',
 - technically is 0 or -1 (if failure)


```
function vim.fn.spellbadword(sentence) end
--- The return value is a list with two items:
--- - The badly spelled word or an empty string.
--- - The type of the spelling error:
```

```
function vim.fn.rpcnotify(channel, event, ...) end
returns = 'integer',
```

```
--- Returns:
---   - The channel ID on success (greater than zero)
---   - 0 on invalid arguments or connection failure.
---
--- @param mode string
--- @param address string
--- @param opts? table
--- @return anytttt
function vim.fn.sockconnect(mode, address, opts) end
```

## Generic
--- @param dict any
--- @return anytttt
function vim.fn.values(dict) end
 - make sure that custom structs would be okay with this (or not)

function vim.fn.keys(dict) end
 - make sure that custom structs would be okay with this (or not)

--- @param expr any
--- @param noref? boolean
--- @return anytttt
function vim.fn.deepcopy(expr, noref) end

--- Get item {idx} from |List| {list}.  When this item is not
--- available return {default}.  Return zero when {default} is
--- omitted.
---
--- @param list any[]
--- @param idx integer
--- @param default? any
--- @return anytttt
function vim.fn.get(list, idx, default) end

```
--- <
---
--- @param list any
--- @param how? string|function
--- @param dict? any
--- @return anytttt
function vim.fn.sort(list, how, dict) end
```

```
--- @param dict any
--- @return anytttt
function vim.fn.items(dict) end
```
 - Maybe it would be a bad idea to restrict? Actually maybe it's fine because
   someone could still do table<any, any>

```
--- @param expr any
--- @return anytttt
function vim.fn.max(expr) end
 - Needs to query the T value of a dict or list
 - This one will be tricky. And so will min() for the same reason

function vim.fn.min(expr) end
 - Needs to query the T value of a dict or list
 - This one will be tricky. And so will max() for the same reason

---
--- @param object any
--- @param func function
--- @param initial? any
--- @return anytttt
function vim.fn.reduce(object, func, initial) end

--- @param list any
--- @param how? string|function
--- @param dict? any
--- @return anytttt
function vim.fn.sort(list, how, dict) end
```

<h1 align="center">
  <img src="https://raw.githubusercontent.com/neovim/neovim.github.io/master/logos/neovim-logo-300x87.png" alt="Neovim">

  <a href="https://neovim.io/doc/">Documentation</a> |
  <a href="https://app.element.io/#/room/#neovim:matrix.org">Chat</a>
</h1>

[![Coverity Scan analysis](https://scan.coverity.com/projects/2227/badge.svg)](https://scan.coverity.com/projects/2227)
[![Packages](https://repology.org/badge/tiny-repos/neovim.svg)](https://repology.org/metapackage/neovim)
[![Debian CI](https://badges.debian.net/badges/debian/testing/neovim/version.svg)](https://buildd.debian.org/neovim)
[![Downloads](https://img.shields.io/github/downloads/neovim/neovim/total.svg?maxAge=2592001)](https://github.com/neovim/neovim/releases/)

Neovim is a project that seeks to aggressively refactor [Vim](https://www.vim.org/) in order to:

- Simplify maintenance and encourage [contributions](CONTRIBUTING.md)
- Split the work between multiple developers
- Enable [advanced UIs] without modifications to the core
- Maximize [extensibility](https://neovim.io/doc/user/ui.html)

See the [Introduction](https://github.com/neovim/neovim/wiki/Introduction) wiki page and [Roadmap]
for more information.

Features
--------

- Modern [GUIs](https://github.com/neovim/neovim/wiki/Related-projects#gui)
- [API access](https://github.com/neovim/neovim/wiki/Related-projects#api-clients)
  from any language including C/C++, C#, Clojure, D, Elixir, Go, Haskell, Java/Kotlin,
  JavaScript/Node.js, Julia, Lisp, Lua, Perl, Python, Racket, Ruby, Rust
- Embedded, scriptable [terminal emulator](https://neovim.io/doc/user/terminal.html)
- Asynchronous [job control](https://github.com/neovim/neovim/pull/2247)
- [Shared data (shada)](https://github.com/neovim/neovim/pull/2506) among multiple editor instances
- [XDG base directories](https://github.com/neovim/neovim/pull/3470) support
- Compatible with most Vim plugins, including Ruby and Python plugins

See [`:help nvim-features`][nvim-features] for the full list, and [`:help news`][nvim-news] for noteworthy changes in the latest version!

Install from package
--------------------

Pre-built packages for Windows, macOS, and Linux are found on the
[Releases](https://github.com/neovim/neovim/releases/) page.

[Managed packages] are in [Homebrew], [Debian], [Ubuntu], [Fedora], [Arch Linux], [Void Linux], [Gentoo], and more!

Install from source
-------------------

See [BUILD.md](./BUILD.md) and [supported platforms](https://neovim.io/doc/user/support.html#supported-platforms) for details.

The build is CMake-based, but a Makefile is provided as a convenience.
After installing the dependencies, run the following command.

    make CMAKE_BUILD_TYPE=RelWithDebInfo
    sudo make install

To install to a non-default location:

    make CMAKE_BUILD_TYPE=RelWithDebInfo CMAKE_INSTALL_PREFIX=/full/path/
    make install

CMake hints for inspecting the build:

- `cmake --build build --target help` lists all build targets.
- `build/CMakeCache.txt` (or `cmake -LAH build/`) contains the resolved values of all CMake variables.
- `build/compile_commands.json` shows the full compiler invocations for each translation unit.

Transitioning from Vim
--------------------

See [`:help nvim-from-vim`](https://neovim.io/doc/user/nvim.html#nvim-from-vim) for instructions.

Project layout
--------------

    ├─ cmake/           CMake utils
    ├─ cmake.config/    CMake defines
    ├─ cmake.deps/      subproject to fetch and build dependencies (optional)
    ├─ runtime/         plugins and docs
    ├─ src/nvim/        application source code (see src/nvim/README.md)
    │  ├─ api/          API subsystem
    │  ├─ eval/         Vimscript subsystem
    │  ├─ event/        event-loop subsystem
    │  ├─ generators/   code generation (pre-compilation)
    │  ├─ lib/          generic data structures
    │  ├─ lua/          Lua subsystem
    │  ├─ msgpack_rpc/  RPC subsystem
    │  ├─ os/           low-level platform code
    │  └─ tui/          built-in UI
    └─ test/            tests (see test/README.md)

License
-------

Neovim contributions since [b17d96][license-commit] are licensed under the
Apache 2.0 license, except for contributions copied from Vim (identified by the
`vim-patch` token). See LICENSE for details.

    Vim is Charityware.  You can use and copy it as much as you like, but you are
    encouraged to make a donation for needy children in Uganda.  Please see the
    kcc section of the vim docs or visit the ICCF web site, available at these URLs:

            https://iccf-holland.org/
            https://www.vim.org/iccf/
            https://www.iccf.nl/

    You can also sponsor the development of Vim.  Vim sponsors can vote for
    features.  The money goes to Uganda anyway.

[license-commit]: https://github.com/neovim/neovim/commit/b17d9691a24099c9210289f16afb1a498a89d803
[nvim-features]: https://neovim.io/doc/user/vim_diff.html#nvim-features
[nvim-news]: https://neovim.io/doc/user/news.html
[Roadmap]: https://neovim.io/roadmap/
[advanced UIs]: https://github.com/neovim/neovim/wiki/Related-projects#gui
[Managed packages]: ./INSTALL.md#install-from-package
[Debian]: https://packages.debian.org/testing/neovim
[Ubuntu]: https://packages.ubuntu.com/search?keywords=neovim
[Fedora]: https://packages.fedoraproject.org/pkgs/neovim/neovim/
[Arch Linux]: https://www.archlinux.org/packages/?q=neovim
[Void Linux]: https://voidlinux.org/packages/?arch=x86_64&q=neovim
[Gentoo]: https://packages.gentoo.org/packages/app-editors/neovim
[Homebrew]: https://formulae.brew.sh/formula/neovim

<!-- vim: set tw=80: -->
