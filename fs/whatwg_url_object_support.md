<!-- YAML
added: v7.6.0
-->

> 稳定的: 1 - 实验性的

对于大多数 `fs` 模块的函数， `path` 或者 `filename` 参数可以当作一个 WHATWG [`URL`][] 对象传入。
只有 [`URL`][] 对象使用被支持的 `file:` 协议。

```js
const fs = require('fs');
const { URL } = require('url');
const fileUrl = new URL('file:///tmp/hello');

fs.readFileSync(fileUrl);
```

*注意*： `file:` URLS 必须是绝对路径。

使用 WHATWG [`URL`][] 对象会不同的平台会有特定的行为。

在 Windows 上， 携带主机名的 `file:` URLs 被转换为 UNC 路径， 而有硬盘盘符的 `file:` URLs 会被转换成
本地绝对路径。既没有主机名，也没有盘符的 `file:` URLs 在转换时会抛出错误。

```js
// On Windows :

// - WHATWG file URLs with hostname convert to UNC path
// file://hostname/p/a/t/h/file => \\hostname\p\a\t\h\file
fs.readFileSync(new URL('file://hostname/p/a/t/h/file'));

// - WHATWG file URLs with drive letters convert to absolute path
// file:///C:/tmp/hello => C:\tmp\hello
fs.readFileSync(new URL('file:///C:/tmp/hello'));

// - WHATWG file URLs without hostname must have a drive letters
fs.readFileSync(new URL('file:///notdriveletter/p/a/t/h/file'));
fs.readFileSync(new URL('file:///c/p/a/t/h/file'));
// TypeError [ERR_INVALID_FILE_URL_PATH]: File URL path must be absolute
```

*注意*： 携带盘符的 `file:` URLs 必须使用 `:` 作为盘符后的分隔符。使用其他符号会抛出错误。

在其他所有的平台上， 都不支持携带主机名的 `file:` URLs，且会抛出错误。 

```js
// On other platforms:

// - WHATWG file URLs with hostname are unsupported
// file://hostname/p/a/t/h/file => throw!
fs.readFileSync(new URL('file://hostname/p/a/t/h/file'));
// TypeError [ERR_INVALID_FILE_URL_PATH]: must be absolute

// - WHATWG file URLs convert to absolute path
// file:///tmp/hello => /tmp/hello
fs.readFileSync(new URL('file:///tmp/hello'));
```

当 `file:` URL 包含已经编码的斜线符号会在所有平台抛出错误。
```js
// On Windows
fs.readFileSync(new URL('file:///C:/p/a/t/h/%2F'));
fs.readFileSync(new URL('file:///C:/p/a/t/h/%2f'));
/* TypeError [ERR_INVALID_FILE_URL_PATH]: File URL path must not include encoded
\ or / characters */

// On POSIX
fs.readFileSync(new URL('file:///p/a/t/h/%2F'));
fs.readFileSync(new URL('file:///p/a/t/h/%2f'));
/* TypeError [ERR_INVALID_FILE_URL_PATH]: File URL path must not include encoded
/ characters */
```

在 Windows 上， 携带已编码的反斜线 `file:` URLs 在编码是会抛出错误。 
```js
// On Windows
fs.readFileSync(new URL('file:///C:/path/%5C'));
fs.readFileSync(new URL('file:///C:/path/%5c'));
/* TypeError [ERR_INVALID_FILE_URL_PATH]: File URL path must not include encoded
\ or / characters */
```

