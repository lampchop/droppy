<p align="center">
  <img src="https://cdn.rawgit.com/silverwind/droppy/master/client/images/readme-logo.svg"/>
</p>
<p align="center">
  <a href="https://www.npmjs.org/package/droppy"><img src="https://img.shields.io/npm/v/droppy.svg"></a>
  <a href="https://raw.githubusercontent.com/silverwind/droppy/master/LICENSE"><img src="https://img.shields.io/badge/licence-bsd-blue.svg"></a>
  <a href="https://www.npmjs.org/package/droppy"><img src="https://img.shields.io/npm/dm/droppy.svg"></a>
</p>

droppy is a self-hosted file storage server with a web interface and capabilites to edit files and view media directly in the browser. It is particularly well-suited to be run on low-end hardware like the Raspberry Pi.

## Features (try the <a target="_blank" href="https://droppy.silverwind.io">demo</a>)
* Fully responsive HTML5 interface
* Realtime updates of changes
* Directory upload support
* Drag & drop and swipe gestures
* Side-by-Side mode
* Shareable public download links
* Zip download of directories
* Powerful editor for text files
* Image and video gallery, audio player
* Fullscreen support for media galleries
* Supports installing to the homescreen

## Installation
Note that two directories will be used by droppy:

- `config` directory: set with `-c <dir>`, default `~/.droppy/config`.
- `files` directory: set with `-f <dir>`, default `~/.droppy/files`.

Make sure these directories are owned by the user running the application/container.

### Local Installation :package:
With [`Node.js`](https://nodejs.org) >= 4.0.0 (>= 6.4.0 on Windows) and `npm` installed, run:

```sh
# Install latest version and dependencies.
$ npm install -g droppy

# Start with `/srv/droppy/config` for config and `/srv/droppy/files` for files.
$ droppy start -c /srv/droppy/config -f /srv/droppy/files
# Open http://localhost:8989/
```

### Docker installation :whale:
```sh
# Pull and start the container, forwarding port 8989 to localhost:8989
$ docker run --name droppy -p 127.0.0.1:8989:8989 silverwind/droppy
# Open http://localhost:8989/
```
The image provides automatic volumes for the two mount points /config and /files which can be overridden through `-v /srv/droppy/config:/config` and `-v /srv/droppy/files:/files` respectively. If you're using existing files, it's adviceable to use the UID and GID container environment variables to get files written with correct ownership, e.g `-e UID=1000` and `-e GID=1000`.

## Configuration
By default, the server listens on all IPv4 and IPv6 interfaces on port 8989. On first startup, a prompt to create login data for the first account will appear. Once it's created, login credentials are enforced. Additional accounts can be created in the options interface or the command line. Configuration is done in `config/config.json`, which is created with these defaults:

```javascript
{
  "listeners" : [
      {
          "host"     : ["0.0.0.0", "::"],
          "port"     : 8989,
          "protocol" : "http"
      }
  ],
  "public"          : false,
  "timestamps"      : true,
  "linkLength"      : 5,
  "logLevel"        : 2,
  "maxFileSize"     : 0,
  "updateInterval"  : 1000,
  "pollingInterval" : 0,
  "keepAlive"       : 20000,
  "allowFrame"      : false,
  "readOnly"        : false
}
```

## Options
- `listeners` *Array* - Defines on which network interfaces, port and protocols the server will listen. See [listener options](#listener-options) below. `listeners` has no effect when droppy is used as a module.
- `public` *Boolean* - When enabled, no user authentication is performed.
- `timestamps` *Boolean* - When enabled, adds timestamps to log output.
- `linkLength` *Number* - The amount of characters in a shared link.
- `logLevel` *Number* - Logging amount. `0` is no logging, `1` is errors, `2` is info (HTTP requests), `3` is debug (Websocket communication).
- `maxFileSize` *Number* - The maximum file size in bytes a user can upload in a single file.
- `updateInterval` *Number* - Interval in milliseconds in which a single client can receive update messages through changes in the file system.
- `pollingInterval` *Number* - Interval in milliseconds in which the file system is polled for changes, which is likely **necessary for files on external or network-mapped drives**. This is CPU-intensive! Corresponds to chokidar's [usePolling](https://github.com/paulmillr/chokidar#performance) option. `0` disables polling.
- `keepAlive` *Number* - Interval in milliseconds in which the server sends keepalive message over the websocket, which may be necessary with proxies. `0` disables keepalive messages.
- `allowFrame` *Boolean* - Allow the page to be loaded into a `<frame>` or `<iframe>`.
- `readOnly` *Boolean* - All served files will be treated as being read-only.
- `dev` *Boolean* - Enable developer mode, skipping resource minification and enabling live reload.

<a name="listener-options" />
### Listener Options

`listeners` defines on which network interfaces, ports and protocol(s) the server will listen. For example:

```javascript
"listeners": [
    {
        "host"     : [ "0.0.0.0", "::" ],
        "port"     : 80,
        "protocol" : "http"
    },
    {
        "host"     : "0.0.0.0",
        "port"     : 443,
        "protocol" : "https",
        "key"      : "~/certs/tls.key",
        "cert"     : "~/certs/tls.crt",
        "ca"       : "~/certs/tls.ca",
        "dhparam"  : "~/certs/tls.dhparam",
        "hsts"     : 31536000
    }
]
```
The above configuration will result in:
- HTTP listening on all IPv4 and IPv6 interfaces, port 80.
- HTTPS listening on all IPv4 interfaces, port 443, with 1 year of HSTS duration, using the provided SSL/TLS files.

A listener object accepts these options:
- `host` *String/Array* - Network interface(s) to listen on. Required.
- `port` *Number/Array* - Network port(s) to listen on. Required.
- `protocol` *String* - Protocol to use, `http` or `https`. Required.

For SSL/TLS these additional options are available:
- `key` *String* - Path to PEM-encoded SSL/TLS private key file. Required.
- `cert` *String* - Path to PEM-encoded SSL/TLS certificate file. Required.
- `ca` *String* - Path to PEM-encoded SSL/TLS intermediate certificate file.
- `dhparam` *String* - Path to PEM-encoded SSL/TLS Diffie-Hellman parameters file. If not provided, new 2048 bit parameters will generated and saved for future use.
- `hsts` *Number* - Length of the [HSTS](http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) header in seconds. Set to `0` to disable HSTS.

*Note: Unless given absolute, SSL/TLS paths are relative to the config folder. If your certificate file includes an concatenated intermediate certificate, it will be detected and used, there's no need to specify `ca` in this case.*

## API
droppy can be used with frameworks like [express](https://github.com/strongloop/express):
```js
var app    = require("express")();
var droppy = require("droppy")({
  configdir: "~/droppy/config"
  filesdir: "~/droppy/files",
  log: "~/droppy/log",
  logLevel: 0
});

app.use("/", droppy);
app.listen(process.env.PORT || 8989);
```
See the [express example](https://github.com/silverwind/droppy/blob/master/examples/express.js) for a working example.

### droppy([options])
- **options** {object}: [Options](#Options). Extends [config.json](#Configuration). In addition to above listed options, `configdir`, `filesdir` and `log` are present on the API.

Returns `function onRequest(req, res)`. All arguments are optional.

## Installation guides
- [Systemd-based distributions](https://github.com/silverwind/droppy/wiki/Systemd-Installation)
- [Debian (Pre-Jessie)](https://github.com/silverwind/droppy/wiki/Debian-Installation-(Pre-Jessie))
- [Nginx reverse proxy](https://github.com/silverwind/droppy/wiki/Nginx-reverse-proxy)
- [Apache reverse proxy](https://github.com/silverwind/droppy/wiki/Apache-reverse-proxy)

### Upgrading a local installation :package:
```sh
$ [sudo] npm install -g droppy
```

### Upgrading a Docker installation :whale:
```sh
$ docker pull silverwind/droppy
$ docker stop droppy
$ docker rm droppy
$ docker run --name droppy -p 8989:8989 -v /srv/droppy/config:/config -v /srv/droppy/files:/files silverwind/droppy
```

## Note about startup performance
droppy is currently optimized for a moderate amount of files. To aid in performance, all directories are indexed into memory once on startup. The downside of this is that the startup will take considerable time on slow storage with hundreds of thousands of files present.

## Note about wget
For correct download filenames of shared links, use `--content-disposition` or add this to `~/.wgetrc`:

```ini
content-disposition = on
```
© [silverwind](https://github.com/silverwind), distributed under BSD licence.
