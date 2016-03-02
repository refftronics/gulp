<p align="center">
  <a href="http://gulpjs.com">
    <img height="257" width="114" src="https://raw.githubusercontent.com/gulpjs/artwork/master/gulp-2x.png">
  </a>
  <p align="center">The streaming build system</p>
</p>

[![NPM version][npm-image]][npm-url] [![Downloads][downloads-image]][npm-url] [![Build Status][travis-image]][travis-url] [![Coveralls Status][coveralls-image]][coveralls-url] [![Gitter chat][gitter-image]][gitter-url]

## What is gulp?

- **Automation** - gulp is a toolkit that helps you automate painful or time-consuming tasks in your development workflow.
- **Platform-agnostic** - Integrations are built into all major IDEs and people are using gulp with PHP, .NET, Node.js, Java, and other platforms.
- **Strong Ecosystem** - Use npm modules to do anything you want + over 2000 curated plugins for streaming file transformations
- **Simple** - By providing only a minimal API surface, gulp is easy to learn and simple to use

## Documentation

For a Getting started guide, API docs, recipes, making a plugin, etc. see the [documentation page](/docs/README.md)!

## Sample `gulpfile.js`

This file is just a quick sample to give you a taste of what gulp does.

```js
var gulp = require('gulp'),
  rename = require('gulp-rename'),
  less = require('gulp-less'),
  cssmin = require('gulp-cssmin'),
  concat = require('gulp-concat'),
  uglify = require('gulp-uglify'),
  del = require('del');

var paths = {
  styles: {
    src: 'src/styles/**/*.less',
    dest: 'assets/styles/'
  },
  scripts: {
    src: 'src/scripts/**/*.js',
    dest: 'assets/scripts/'
  }
};

/* Not all tasks need to use streams, a gulpfile is just another node program
 * and you can use all packages available on npm, but it must return either a
 * Promise, a Stream or take a callback and call it
 */
function clean() {
  // You can use multiple globbing patterns as you would with `gulp.src`,
  // for example if you are using del 2.0 or above, return its promise
  del([ 'assets' ]);
}

/* 
 * Define our tasks using plain functions
 */
function styles() {
  return gulp.src(paths.styles.src)
    .pipe(less())
    .pipe(cssmin())
    // pass in options to the task
    .pipe(rename({
          basename: 'main',
          suffix: '.min'
        }))
    .pipe(gulp.dest(paths.styles.dest));
}

function scripts() {
  return gulp.src(paths.scripts.src)
    .pipe(uglify())
    .pipe(concat(main.min.js))
    .pipe(gulp.dest(paths.scripts.dest));
}

function watch() {
  gulp.watch(paths.scripts.src, scripts);
  gulp.watch(paths.styles.src, styles);
}

/* 
 * Register some tasks to expose to the cli
 */
gulp.task(clean);
gulp.task(watch);

/* 
 * Notice that you can specify if tasks can run in parallel or not
 * using `gulp.series` and `gulp.parallel`
 */
gulp.task('build',
  gulp.series(clean),
  gulp.parallel(styles, scripts));

/*
 * Expose default task that can be called by just running `gulp` from cli
 */
gulp.task('default',
  gulp.series(build))
```

## Use last JavaScript version in your gulpfile

Node already supports a lot of **ES2015**, to avoid compatibility problem we suggest to install Babel and rename your `gulpfile.js` as `gulpfile.babel.js`.

```sh
npm install --save-dev babel-core -babel-preset-es2015
```

Here's the same sample from above written in **ES2015**.

```js
import gulp from 'gulp';
import rename from 'gulp-rename';
import less from 'gulp-less';
import cssmin from 'gulp-cssmin';
import concat from 'gulp-concat';
import uglify from 'gulp-uglify';
import del from 'del';

const paths = {
  styles: {
    src: 'src/styles/**/*.less',
    dest: 'assets/styles/'
  },
  scripts: {
    src: 'src/scripts/**/*.js',
    dest: 'assets/scripts/'
  }
};

function clean() {
  del([ 'assets' ]);
}

function styles() {
  return gulp.src(paths.styles.src)
    .pipe(less())
    .pipe(cssmin())
    .pipe(rename({
          basename: 'main',
          suffix: '.min'
        }))
    .pipe(gulp.dest(paths.styles.dest));
}

function scripts() {
  return gulp.src(paths.scripts.src)
    .pipe(uglify())
    .pipe(concat(main.min.js))
    .pipe(gulp.dest(paths.scripts.dest));
}

function watch() {
  gulp.watch(paths.scripts.src, scripts);
  gulp.watch(paths.styles.src, styles);
}

gulp.task(clean);
gulp.task(watch);

gulp.task('build',
  gulp.series(clean),
  gulp.parallel(styles, scripts));

gulp.task('default',
  gulp.series('build'))
```

## Incremental Builds

You can filter out unchanged files between runs of a task using
the `gulp.src` function's `since` option and `gulp.lastRun`:
```js
function images() {
  return gulp.src(paths.images, {since: gulp.lastRun('images')})
    .pipe(imagemin({optimizationLevel: 5}))
    .pipe(gulp.dest('build/img'));
}

function watch() {
  gulp.watch(paths.images, images);
}
```
Task run times are saved in memory and are lost when gulp exits. It will only
save time during the `watch` task when running the `images` task
for a second time.

If you want to compare modification time between files instead, we recommend these plugins:
- [gulp-changed];
- or [gulp-newer] - supports many:1 source:dest.

[gulp-newer] example:
```js
function images() {
  var dest = 'build/img';
  return gulp.src(paths.images)
    .pipe(newer(dest))  // pass through newer images only
    .pipe(imagemin({optimizationLevel: 5}))
    .pipe(gulp.dest(dest));
}
```

If you can't simply filter out unchanged files, but need them in a later phase
of the stream, we recommend these plugins:
- [gulp-cached] - in-memory file cache, not for operation on sets of files
- [gulp-remember] - pairs nicely with gulp-cached

[gulp-remember] example:
```js
function scripts() {
  return gulp.src(scriptsGlob)
    .pipe(cache('scripts'))    // only pass through changed files
    .pipe(header('(function () {')) // do special things to the changed files...
    .pipe(footer('})();'))     // for example,
                               // add a simple module wrap to each file
    .pipe(remember('scripts')) // add back all files to the stream
    .pipe(concat('app.js'))    // do things that require all files
    .pipe(gulp.dest('public/'))
}
```

## Want to contribute?

Anyone can help make this project better - check out our [Contributing guide](/CONTRIBUTING.md)!

[downloads-image]: http://img.shields.io/npm/dm/gulp.svg
[npm-url]: https://npmjs.org/package/gulp
[npm-image]: http://img.shields.io/npm/v/gulp.svg

[travis-url]: https://travis-ci.org/gulpjs/gulp
[travis-image]: http://img.shields.io/travis/gulpjs/gulp.svg

[coveralls-url]: https://coveralls.io/r/gulpjs/gulp
[coveralls-image]: http://img.shields.io/coveralls/gulpjs/gulp/master.svg

[gitter-url]: https://gitter.im/gulpjs/gulp
[gitter-image]: https://badges.gitter.im/gulpjs/gulp.png

[gulp-cached]: https://github.com/wearefractal/gulp-cached
[gulp-remember]: https://github.com/ahaurw01/gulp-remember
[gulp-changed]: https://github.com/sindresorhus/gulp-changed
[gulp-newer]: https://github.com/tschaub/gulp-newer
