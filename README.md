
# Filewalker

## Fast and rock-solid asynchronous traversing of directories and files for node.js

The filewalker-module for node is designed to provide maximum
reliance paired with maximum throughput/performance and the
ability to throttle that throughput/performance.

# Installation

``npm install filewalker``

# Usage

Simple directory listing and disk-usage report:

``
var filewalker = require('filewalker');

filewalker('.')
  .on('dir', function(p) {
    console.log('dir:  %s', p);
  })
  .on('file', function(p, s) {
    console.log('file: %s, %d bytes', p, s.size);
  })
  .on('error', function(err) {
    console.error(err);
  })
  .on('done', function() {
    console.log('%d dirs, %d files, %d bytes', this.dirs, this.files, this.bytes);
  })
.walk();
``

Calculate md5-hash for every file:

``
var started = Date.now();

var createHash = require('crypto').createHash,
    filewalker = require('./lib/filewalker');

var options = {
  maxPending: 10, // throttle handles
};

filewalker('c:/', options)
  .on('stream', function(rs, p, s, fullPath) {
    var hash = createHash('md5');
    rs.on('data', function(data) {
      hash.update(data);
    });
    rs.on('end', function(data) {
      console.log(hash.digest('hex'), ('                '+s.size).slice(-16), p);
    });
  })
  .on('error', function(err) {
    console.error(err);
  })
  .on('done', function() {
    var duration = Date.now()-started;
    console.log('%d ms', duration);
    console.log('%d dirs, %d files, %d bytes', this.dirs, this.files, this.bytes);
  })
.walk();
``

# Class Filewalker

* inherits from events.EventEmitter

# Options

* maxPending (default: -1)
  Maximum asynchronous jobs. Usefull to throttle the number of
  simultaneous disk-operations.
  
* maxAttempts (default: 3)
  Maximum reattempts on error.
  Set to 0 to disable reattempts.
  Set to -1 for infinite reattempts.
  
* attemptTimeout (default: 5000 ms)
  Minimum time to wait before reattempt. In milliseconds.
  Usefull to let diskdrives remount, etc.
  
* matchRegExp (default: null)
  A RegExp-instance the path to a file must match in order to
  emit a "file" event. Set to ``null`` to emit all paths.

# Properties

* maxPending
* maxAttempts
* attemptTimeout
* matchRegExp

* pending
* dirs
* files
* total
* bytes
* errors
* attempts
* streamed
* streamErrors
* open
* detectedMaxOpen

# Methods

* walk()
* pause()
* resume()

# Events

* file
  * relative path
  * fs.Stats instance
  * absolute path
* dir
  * relative path
  * fs.Stats instance
  * absolute path
* stream
  * fs.ReadStream instance
  * relative path
  * fs.Stats instance
  * absolute path
* pause
* resume
* done
* error
  * instance of Error

Notice: There will be no fs.ReadStream created if no listener listens to the 'stream'-event.
