---
title: Gulp学习笔记
categories: programming
tags: javascript,gulp
---

# Gulp学习笔记

### 创建标准任务

利用`task`方法建立task，形如

```javascript
var gulp = require('gulp');

gulp.task('task-name', function() {
  // Stuff here
});
```

上述创建的task使用`gulp task-name`调用

标准task形如

```javascript
gulp.task('task-name', function () {
  return gulp.src('source-files') // Get source files with gulp.src
    .pipe(aGulpPlugin()) // Sends it through a gulp plugin
    .pipe(gulp.dest('destination')) // Outputs the file in the destination folder
})
```

比如我们写一个sass的precompiler，需要使用到`gulp-sass`

```javascript
var sass = require('gulp-sass');

gulp.task('scss', function(){
  return gulp.src('app/scss/**/*.scss')
    .pipe(sass()) // Using gulp-sass
    .pipe(gulp.dest('app/css'))
});
```

当我们`gulp scss`时就会将`app/scss`下及其子目录下的所有scss文件编译到`app/css`下，这里我们需要注意的是`app/scss/**/*.scss`这个写法，其实有以下4中匹配写法：

1. `*.scss`匹配当前目录下所有scss文件
2. `**/*.scss`匹配当前目录及其子目录下所有scss文件
3. `!not-me.scss`排除指定文件
4. `*.+(scss|sass)`类似正则，会匹配所有sass和scss文件

### 监控

`watch`方法可以创建监控，形如

```javascript
gulp.watch('files-to-watch', ['tasks', 'to', 'run']); 
```

比如说，我们要监控指定文件，如果有变化时自动执行刚刚的`scss`任务，可以写成

```javascript
gulp.task('watch', function(){
  gulp.watch('app/scss/**/*.scss', ['scss']); 
})
```

我们创建一个名为`watch`的task，然后调用`watch`方法监控`app/scss/**/*.scss`，一旦变化便执行`scss`任务

还有一种写法，形如

```javascript
gulp.task('watch', ['array', 'of', 'tasks', 'to', 'complete', 'before', 'watch'], function (){
  // ...
})
```

第二个参数里的数组会在执行watch任务前并行执行吗，下面会有具体例子

### 浏览器同步

可以利用`browser-sync`做到浏览器同步变化，下面创建一个browserSync的任务

```javascript
var browserSync = require('browser-sync').create();

gulp.task('browserSync', function() {
  browserSync.init({
    server: {
      baseDir: 'app'
    },
  })
})
```

然后对scss及watch任务稍作修改

```javascript
gulp.task('scss', function() {
  return gulp.src('app/scss/**/*.scss') // Gets all files ending with .scss in app/scss
    .pipe(sass())
    .pipe(gulp.dest('app/css'))
    .pipe(browserSync.reload({
      stream: true
    }))
});

gulp.task('watch', ['browserSync', 'scss'], function (){
  gulp.watch('app/scss/**/*.scss', ['scss']); 
  // Other watchers
});
```

这样在执行watch任务前会先（并行）执行browserSync和scss，然后（并不会等待执行完成）执行对指定文件的监控，而scss任务也会在编译后通知浏览器同步更新，至此我们就完成了浏览器同步

实际上同步是调用了`browserSync.reload`这个方法，比如我们可以增加对html及js文件的监控，在变化后通知浏览器同步更新

```javascript
gulp.task('watch', ['browserSync', 'scss'], function (){
  gulp.watch('app/scss/**/*.scss', ['scss']); 
  // Reloads the browser whenever HTML or JS files change
  gulp.watch('app/*.html', browserSync.reload); 
  gulp.watch('app/js/**/*.js', browserSync.reload); 
});
```

### 合并、优化文件

##### 合并文件

需用到`gulp-useref`，会用到如下语法

```html
<!-- build:<type> <path> -->
... HTML Markup, list of script / link tags.
<!-- endbuild -->
```

`type`可以是`js`/`css`/`remove`，用`remove`标记的段落不会出现在编辑出来的html文件中。`path`指定生成文件的目录及文件名，如

```html
<!--build:js js/main.min.js -->
<script src="js/lib/a-library.js"></script>
<script src="js/lib/another-library.js"></script>
<script src="js/main.js"></script>
<!-- endbuild -->

<!--build:remove -->
<link rel="stylesheet" href="css/no-need-stylesheet.css">
<!--endbuild-->

<!--build:css css/styles.min.css-->
<link rel="stylesheet" href="css/styles.css">
<link rel="stylesheet" href="css/another-stylesheet.css">
<!--endbuild-->
```

```javascript
var useref = require('gulp-useref');

gulp.task('useref', function(){
  return gulp.src('app/*.html')
    .pipe(useref())
    .pipe(gulp.dest('dist'))
});
```

执行`gulp useref`后会读取app目录下所有html文件，将其编译到dist目录下，而dist目录下会按配置生成拼接好的js和css文件及引入了拼接好的文件的html文件

##### 优化文件

可以使用`gulp-uglify`/`gulp-cssnano`分别对js/css文件进行压缩，还要使用到`gulp-if`用来指定文件特定处理

```javascript
var uglify = require('gulp-uglify');
var gulpIf = require('gulp-if');

gulp.task('useref', function(){
  return gulp.src('app/*.html')
    .pipe(useref())
    // Minifies only if it's a JavaScript file
    .pipe(gulpIf('*.js', uglify()))
    .pipe(gulpIf('*.css', cssnano()))
    .pipe(gulp.dest('dist'))
});
```

##### 优化图片

可以使用`gulp-imagemin`对图片进行压缩，由于图片处理比较耗时，所以可以用`gulp-cache`达到只针对修改过的文件进行操作

```javascript
var imagemin = require('gulp-imagemin');
var cache = require('gulp-cache');

gulp.task('images', function(){
  return gulp.src('app/images/**/*.+(png|jpg|jpeg|gif|svg)')
  // Caching images that ran through imagemin
  .pipe(cache(imagemin({
      interlaced: true
    })))
  .pipe(gulp.dest('dist/images'))
});
```

### 复制文件

我们可以在执行构建任务时执行复制操作，譬如已经优化好的字体文件，一般只需复制到指定目录即可

```javascript
gulp.task('fonts', function() {
  return gulp.src('app/fonts/**/*')
  .pipe(gulp.dest('dist/fonts'))
})
```

### 清空目录

我们在进行构建任务前需要将上次构建的文件先清空，可以用`del`，如下创建任务`clean:dist`

```javascript
var del = require('del');

gulp.task('clean:dist', function() {
  return del.sync('dist');
})
```

如果需要清空上面提到的`gulp-cache`包产生的缓存，可以创建任务`cache:clear`

```javascript
gulp.task('cache:clear', function (callback) {
return cache.clearAll(callback)
})
```

### 合并串行任务得到workflow

上述任务如若不做处理则需每条单独手动执行，我们可以定义任务达到串行执行任务的目的，如上述的watch任务，可在执行任务前可以指定执行一批任务，但由于是并行的，故若对执行顺序有硬性要求（如必须在所有任务执行前执行完`clean:dist`清空命令）的话则不能用该方法。可以用`run-sequence`达到需求，可如下定义任务

```javascript
var runSequence = require('run-sequence');

// 按照顺序执行task-one、task-two、task-three
gulp.task('task-name', function(callback) {
  runSequence('task-one', 'task-two', 'task-three', callback);
});

// 先执行task-one，然后并行执行一批任务
gulp.task('task-name', function(callback) {
  runSequence('task-one', ['tasks','two','run','in','parallel'], 'task-three', callback);
});
```

针对我们上述的情况，可以这么定义

```javascript
gulp.task('build', function (callback) {
  runSequence('clean:dist', 
    ['sass', 'useref', 'images', 'fonts'],
    callback
  )
})
```

### default任务

我们可以定义一个名为default的任务，则若只执行`gulp`时会执行该任务

```javascript
gulp.task('default', function (callback) {
  runSequence(['sass','browserSync', 'watch'],
    callback
  )
})
```

### 其他gulp包

For development:

* Using [Autoprefixer](https://www.npmjs.com/package/gulp-autoprefixer) to write vendor-free CSS code
* Adding [Sourcemaps](https://www.npmjs.com/package/gulp-sourcemaps) for easier debugging
* Creating Sprites with [sprity](https://www.npmjs.com/package/sprity)
* Compiling only files that have changed with [gulp-changed](https://www.npmjs.com/package/gulp-changed)
* Writing ES6 with [Babel](https://www.npmjs.com/package/gulp-babel) or [Traceur](https://www.npmjs.com/package/gulp-traceur)
* Modularizing Javascript files with [Browserify](http://browserify.org/), [webpack](https://github.com/webpack/webpack), or [jspm](https://github.com/jspm)
* Modularizing HTML with template engines like [Handlebars](https://www.npmjs.com/package/gulp-handlebars) or [Swig](https://www.npmjs.com/package/gulp-swig)
* Splitting the gulpfile into smaller files with [require-dir](https://www.npmjs.com/package/require-directory)
* Generating a Modernizr script automatically with [gulp-modernizr](https://www.npmjs.com/package/gulp-modernizr)
* unit tests [gulp-jasmine](https://www.npmjs.com/package/gulp-jasmine)
* deploy [gulp-rsync](https://www.npmjs.com/package/gulp-rsync)

For optimization:

* Removing unused CSS with [unCSS](https://www.npmjs.com/package/gulp-uncss)
* Further optimizing CSS with [CSSO](https://www.npmjs.com/package/gulp-csso)
* Generating inline CSS for performance with [Critical](https://www.npmjs.com/package/critical)

---

##### 执行过的命令

```bash
npm install gulp -g
npm init
npm install gulp --save-dev
npm install gulp-sass --save-dev
npm install browser-sync --save-dev
npm install gulp-useref --save-dev
npm install gulp-uglify --save-dev 
npm install gulp-if --save-dev 
npm install gulp-imagemin --save-dev
npm install gulp-cache --save-dev
npm install del --save-dev
npm install run-sequence --save-dev
```

Ref: [Gulp for Beginners](https://css-tricks.com/gulp-for-beginners/)


