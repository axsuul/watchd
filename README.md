Watchd
=======

Watchd is a daemon that forwards local filesystem change events over a network socket. It's designed to be run on a development machine to relay filesystem events into a virtual machine or container.

The protocol is compatible with [guard/listen](), [watch-network](https://github.com/efacilitation/watch-network), [GoListen](https://github.com/ekino/golisten). See [compatible watchers](#Compatible-Watchers) for more.

It's compatible with OSX, Windows and Linux, or any combination of those. Binaries are available for each platform.

## Usage

On your host machine (say OSX, or Windows):

```bash
$ watchd -bind :4000 ~/Projects/watchd
```

Then, running via [watch-network](https://github.com/efacilitation/watch-network) in a container:

```javascript
gulp.task('watch-network', function() {
  var watch = WatchNetwork({
    gulp: gulp,
    host: '192.168.0.1',
    port: '4000',
    configs: [{
      tasks: 'build',
      onLoad: true
    }, {
      patterns: ['scripts/*.jsx', 'scripts/*/*.jsx'],
      tasks: 'browserify'
    }, {
      patterns: 'styles/*.scss',
      tasks: 'styles'
    }]
  })

  watch.initialize();
});
```

```bash
$ gulp watch-network
```

## Why?

With the rise of Docker containers and virtual machine based development many developers are relying on filesystems like NFS and VirtualBox Folder Sharing to mount their sourcecode into the development environment. This causes problems when file watchers are used, as filesystem change events aren't propagated to the inner host. The end result is that the watcher tools are typically run on the host system, which largely defeats the point of having a repeatably built development environment.

## Rewriting File Paths

As the paths change between the host environment and the target environment, watchd offers a variety of ways to rewrite paths, either on the host, or when running in Proxy Mode on the target environment.

### Relative Paths

The simplest approach is to make paths relative to a given path:

```bash
$ watchd -bind :4000 -relative ~/Projects/watchd
```

### Rewriting Paths

Another approach is to rewrite the paths to the path expected by the target environment:

```bash
$ watchd -bind :4000 -rewrite ~/Projects:/ ~/Projects/watchd
```

The above will rewrite paths to `/watchd/...`.

## Proxy Mode

Proxy mode can be used to both receive remote events and provide a local tcp socket or socket file. The usecase for this is to run inside a Virtual Machine and then into containers, as it's often not possible for containers to connect back to the host machine.

On the host:

```bash
$ watchd -bind :4000 ~/Projects/watchd
```

On the virtual machine:

```
$ watchd -connect 192.168.0.1:4000 -bind :4000 -rewrite /Users/lachlan/Projects:/
```

## Protocol

The protocol is a very simple JSON over TCP protocol, originally devised in the guard/listen project.

```
uint32be(length) + json encoded array
```

The payload is a JSON array, with the following values as strings:

  - "file", a constant from the original project
  - "event", either added, removed, renamed or modified
  - "dir", the directory of the file
  - "file", the file that changed
  - "{}", A hash of optional properties, the contents of which are left undefined

## Compatible Watchers

 - [guard/listen 2.x](https://github.com/guard/listen/tree/v2.10.1) (ruby)
 - [watch-network](https://github.com/efacilitation/watch-network) (javascript / gulp)
 - [GoListen](https://github.com/ekino/golisten) (golang / generic)
 - [rerun](https://github.com/lox/rerun) (golang)






