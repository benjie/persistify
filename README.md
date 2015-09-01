[![NPM Version](http://img.shields.io/npm/v/persistify.svg?style=flat)](https://npmjs.org/package/persistify)

# persistify
`persistify` is a wrapper over watchify and browserify to make it easy to make incremental builds without having to rely on the `watch` mode for scenarios where a watch mode is not available. It reduces the build time from several seconds to milliseconds, without relying on a watch mode :)

## Motivation
Just wanted to have a good wrapper over browserify/watchify that allow me to make incremental builds even when not using the watch mode.

**DISCLAIMER**: this is done persisting the watchify arguments to disk using the [flat-cache](https://npmjs.org/package/flat-cache) and [file-entry-cache](https://npmjs.org/package/file-entry-cache) modules. The best results happen when only a few files were modified. Worse case scenario is when all the files are modified. In average you should experience a very noticeable reduction. As always keep in mind that your mileage may vary.

## TODO

Add unit tests

## Install

```bash
npm i -g persistify
```

## CLI options

Apart from all the browserify and watchify options the following are also available:

```bash
Standard Options:

  --outfile=FILE, -o FILE

    This option is required. Write the browserify bundle to this file. If
    the file contains the operators `|` or `>`, it will be treated as a
    shell command, and the output will be piped to it (not available on
    Windows).

  --verbose, -v                     [default: false]

    Show when a file was written and how long the bundling took (in
    seconds).

  --version

    Show the persistify, watchify and browserify versions with their module paths.

  --watch                          [default: false]

    if true will use watchify instead of browserify

  --recreate                       [default: false]

    if set will recreate the cache. Useful when transforms and cached files refuse to cooperate
```

## Examples

```bash
# this will browserify src/foo.js and move it to dist/foo.js
# the cache is constructed the first time the command is run so this might take a few
# seconds depending on the complexity of the files you want to browserify
persistify src/foo.js -o dist/foo.js

# next builds will be benefited by the cache
# noticeable reducing the building time
persistify src/foo.js -o dist/foo.js

# reconstruct the cache, this useful when a transform file has changed or
# the cache just started to behave like a spoiled child
persistify src/foo.js -o dist/foo.js --recreate

# this will use the cache and watchify to provide faster startup times on watch mode
persistify src/foo.js -o dist/foo.js --watch

# this will just use the cache and use a transform
# (all the parameters are just passed to browserify
# so it should work with any transform)
persistify src/foo.js -t babelify -o dist/foo.js --watch
```

## As a node module

```javascript
var persistify = require( 'persistify' );

var b = persistify( {
  //browserify options here. e.g
  // debug: true
  }, { watch: true } );

b.add( './demo/dep1.js' );

b.on( 'bundle:done', function ( time ) {
  console.log( 'time', time );
} );

b.on( 'error', function ( err ) {
  console.log( 'error', err );
} );

function doBundle() {
  b.bundle( function ( err, buff ) {
    if ( err ) {
      throw err;
    }
    require( 'fs' ).writeFileSync( './dist/bundle.js', buff.toString() );
  } );

}

doBundle();

b.on( 'update', function () {
  doBundle();
} );

```

## FAQ

### My less files are not detected as changed when using a transformation like `lessify`. Why?

In short, because those files are not loaded thru browserify and the cache will ignore them. use `--recreate` to recreate the cache. **TODO**: Add an option to never cache certain type of files so at least this works for the javascript ones.

Long answer below:
So far `persistify` will only save to disk the **browserify cache**, if some transform loads a file during the transformation
process the change to that will not be detected. Maybe we need to make the transforms to also be able to create entries in the browserify cache, at least to list the dependencies of the file being transformed (common case in transforms that handle `less`, `sass` or `jade` code).

### My build does not include the latest changes to my files! not detecting changed files?

Mmm... that's weird, but the option `--recreate` should destroy the cache and create it again which most of the times should fix your issue.

### I have added a new transform and the build is not using its magic!

Since persistify will only work on the files that have changed, and adding a transform
does not cause a file change it is better to just use `--recreate` after adding a new trasform or plugin

## Changelog

[Changelog](./changelog.md)

## License

[MIT](./LICENSE)
