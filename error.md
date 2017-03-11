Applications running in Node.js will generally experience four categories of errors:

- Standard JavaScript errors such as:
  - `<EvalError>` : thrown when a call to `eval()` fails.
  - `<SyntaxError>` : thrown in response to improper JavaScript language syntax.
  - `<RangeError>` : thrown when a value is not within an expected range
  - `<ReferenceError>` : thrown when using undefined variables
  - `<TypeError>` : thrown when passing arguments of the wrong type
<URIError> : thrown when a global URI handling function is misused.
- System errors triggered by underlying operating system constraints such as attempting to open a file that does not exist, attempting to send data over a closed socket, etc;
- And User-specified errors triggered by application code.
- Assertion Errors are a special class of error that can be triggered whenever Node.js detects an exceptional logic violation that should never occur. These are raised typically by the assert module.
All JavaScript and System errors raised by Node.js inherit from, or are instances of, the standard JavaScript <Error> class and are guaranteed to provide at least the properties available on that class.

```js
// Throws with a ReferenceError because z is undefined
try {
  const m = 1;
  const n = m + z;
} catch (err) {
  // Handle the error here.
}
```
```js
require('url').parse(() => { });
  // throws TypeError, since it expected a string
```
Any use of the JavaScript throw mechanism will raise an exception that must be handled using try / catch or the Node.js process will exit immediately.

Errors that occur within Asynchronous APIs may be reported in multiple ways:
- Most asynchronous methods that accept a callback function will accept an Error object passed as the first argument to that function. If that first argument is not null and is an instance of Error, then an error occurred that should be handled.
```js
const fs = require('fs');
fs.readFile('a file that does not exist', (err, data) => {
  if (err) {
    console.error('There was an error reading the file!', err);
    return;
  }
  // Otherwise handle the data
});
```
- When an asynchronous method is called on an object that is an EventEmitter, errors can be routed to that object's 'error' event.
```js
const net = require('net');
const connection = net.connect('localhost');

// Adding an 'error' event handler to a stream:
connection.on('error', (err) => {
  // If the connection is reset by the server, or if it can't
  // connect at all, or on any sort of error encountered by
  // the connection, the error will be sent here.
  console.error(err);
});

connection.pipe(process.stdout);
```
A handful of typically asynchronous methods in the Node.js API may still use the throw mechanism to raise exceptions that must be handled using try / catch. There is no comprehensive list of such methods; please refer to the documentation of each method to determine the appropriate error handling mechanism required.

The JavaScript `try / catch` mechanism cannot be used to intercept errors generated by asynchronous APIs. A common mistake for beginners is to try to use throw inside a Node.js style callback:
```js
// THIS WILL NOT WORK:
const fs = require('fs');

try {
  fs.readFile('/some/file/that/does-not-exist', (err, data) => {
    // mistaken assumption: throwing here...
    if (err) {
      throw err;
    }
  });
} catch (err) {
  // This will not catch the throw!
  console.error(err);
}
```
This will not work because the callback function passed to `fs.readFile()` is called asynchronously. By the time the callback has been called, the surrounding code (including the `try { } catch (err) { }` block will have already exited. Throwing an error inside the callback can crash the Node.js process in most cases. If domains are enabled, or a handler has been registered with `process.on('uncaughtException')`, such errors can be intercepted.

####Exceptions vs. Errors
A JavaScript exception is a value that is thrown as a result of an invalid operation or as the target of a throw statement. While it is not required that these values are instances of Error or classes which inherit from Error, all exceptions thrown by Node.js or the JavaScript runtime will be instances of Error.

Some exceptions are unrecoverable at the JavaScript layer. Such exceptions will always cause the Node.js process to crash. Examples include `assert()` checks or `abort()` calls in the C++ layer.

####System Errors
System errors are generated when exceptions occur within the program's runtime environment.

####Common System Errors

This list is not exhaustive, but enumerates many of the common system errors encountered when writing a Node.js program. An exhaustive list may be found here.

- `EACCES` (Permission denied): An attempt was made to access a file in a way forbidden by its file access permissions.

- `EADDRINUSE` (Address already in use): An attempt to bind a server (net, http, or https) to a local address failed due to another server on the local system already occupying that address.

- `ECONNREFUSED` (Connection refused): No connection could be made because the target machine actively refused it. This usually results from trying to connect to a service that is inactive on the foreign host.

- `ECONNRESET` (Connection reset by peer): A connection was forcibly closed by a peer. This normally results from a loss of the connection on the remote socket due to a timeout or reboot. Commonly encountered via the http and net modules.

- `EEXIST` (File exists): An existing file was the target of an operation that required that the target not exist.

- `EISDIR` (Is a directory): An operation expected a file, but the given pathname was a directory.

- `EMFILE` (Too many open files in system): Maximum number of file descriptors allowable on the system has been reached, and requests for another descriptor cannot be fulfilled until at least one has been closed. This is encountered when opening many files at once in parallel, especially on systems (in particular, OS X) where there is a low file descriptor limit for processes. To remedy a low limit, run ulimit -n 2048 in the same shell that will run the Node.js process.

- `ENOENT` (No such file or directory): Commonly raised by fs operations to indicate that a component of the specified pathname does not exist -- no entity (file or directory) could be found by the given path.

- `ENOTDIR` (Not a directory): A component of the given pathname existed, but was not a directory as expected. Commonly raised by fs.readdir.

- `ENOTEMPTY` (Directory not empty): A directory with entries was the target of an operation that requires an empty directory -- usually fs.unlink.

- `EPERM` (Operation not permitted): An attempt was made to perform an operation that requires elevated privileges.

- `EPIPE` (Broken pipe): A write on a pipe, socket, or FIFO for which there is no process to read the data. Commonly encountered at the net and http layers, indicative that the remote side of the stream being written to has been closed.

- `ETIMEDOUT` (Operation timed out): A connect or send request failed because the connected party did not properly respond after a period of time. Usually encountered by `http` or `net` -- often a sign that a `socket.end()` was not properly called.