# [import](https://import.pw)

`import` is a simple and fast import and module system for Bash and other Unix
shells.

Inspired by Go's import command, you specify the URI of the shell script,
and the `import` function downloads the file and caches it to `~/.import-cache`
_forever_.

The code will never change from below your feet and will work offline.


## Compatibility

`import` is unit tested against the following shell implementations:

 * [`ash`](https://en.wikipedia.org/wiki/Almquist_shell) (BusyBox's shell)
 * [`bash`](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) (GNU's Bourne Again SHell)
 * [`zsh`](https://en.wikipedia.org/wiki/Z_shell) (Z shell)


## 👋 Example

This gist https://git.io/f4SOX contains a simple `add` shell function:

```bash
add() {
  echo "$(( $1 + $2 ))"
}
```

You can use the `import` function to download, cache, and use that function in
your own script:

```bash
#!/bin/sh

# Bootstrap the `import` function
eval "`curl -sfLS import.pw`"

# The gist is downloaded once, cached forever, and then `source`d into your script
import "git.io/f4SOX"

add 7 11
# 18
```


## 🔑 Authentication

Because `import` uses `curl`, you can use the standard [`.netrc` file
format](https://ec.haxx.se/usingcurl-netrc.html) to define your username
and passwords to the server you are importing from.

For exampe, to make script files in private GitHub repos accessible, create a
`~/.netrc` file that contains something like:

```
machine raw.githubusercontent.com
login 231a4602aeb1fbcf164f7c444ae5a211c1451d95
password x-oauth-basic
```

The `login` token is a [GitHub "personal access token"](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/).
Follow the instructions in that link to create one for yourself.

After that, an `import` call to a private repo will work as expected:

```
import "import.pw/my-private-org/repo@1.0.0"
```

Your GitHub credentials **ARE NEVER** given to the `import.pw` server.
They are only used _locally_ by `curl` once import.pw redirects to the
private repo URL.


## 🐔🥚 Bootstrapping the `import.sh` script

Since the `import.sh` file itself defines the `import` function, you naturally
can not _use_ the `import` function to load the import script. This is a classic
chicken vs. egg problem!

The "quick and dirty" way is to simply `curl` + `eval` the import script:

```bash
eval "`curl -sfLS import.pw`"
```

However, this involves an HTTP request every time that the shell script is run,
and thus would not work offine and is not as optimized.

A more robust solution is to first cache the import script to the
filesystem, and then `source` it afterwards. For example:

```bash
test -f "$HOME/.import.sh" || curl -sfS https://import.pw > "$HOME/.import.sh"
source "$HOME/.import.sh"
```
