# PSR

## Terminology

Fully qualified class name - class name with all of its namespaces;

## Overview

In 2009, _PHP-FIG_ was formed (PHP Framework Interop Group), the purpose of which was to find commonalities
between projects/frameworks to easily reuse their parts. These parts are called PSR.
Many frameworks/projects now follow standards written by PHP-FIG:
Composer, Magento, Slim, etc ([full list](https://www.php-fig.org/personnel/#member-projects)).

PSR (PHP Standards Recommendations) are used to standardize programming concepts in PHP.
The aim is to enable interoperability of components and to provide a common technical basis for
implementation of proven concepts for optimal programming and testing practices.
All PSR are available [here](https://www.php-fig.org/psr).

PSR can be split into 4 categories: autoloading, interfaces, http, coding styles.
Each standard has 1 of 4 statuses - accepted, deprecated, draft, abandoned.

## Autoloading standards

When writting OOP-code, each class is placed in separate file. When one class
needs to use other classes, it imports them. But the number of classes quickly grows
and including all classes becomes tedious. Thus two standards were created - PSR-0 and PSR-4.
Both standards deal with namespaces and class names. These PSR also describe where
to place files that will be autoloaded according to the specification.

### PSR-0: Autoloading Standard

_PSR-0_ (deprecated):

- each namespace must have a top-level namespace, called "vendor name";
- namespaces can have as many sub-namespaces as they wish;
- fully-qualified namespace and class are suffixed with `.php` when loading from the file system;
- each \_ character in the class name is converted to a `DIRECTORY_SEPARATOR` (/ or \\).

Examples:

- `\Symfony\Core\Request` => `/path/to/project/lib/vendor/Symfony/Core/Request.php`
- `\namespace\package_name\Class_Name` => `/path/to/project/lib/vendor/namespace/package_name/Class/Name.php`

PSR-0 has been deprecated and is now replaced/extended by PSR-4.

### PSR-4: Autoloader

_PSR-4_ (accepted):

- does not force you to have the whole namespace as a directory structure;
- does not convert underscores to directory separators;
- the terminating class name corresponds to a file name ending in `.php`. The file name must
  match the case of the terminating class name.

Examples:
| Class name | Namespace | Base directory | Resulting file path |
| ----------- | --------- | --------------------- | ---------------------------- |
| `\Zend\Acl` | `Zend` | `/usr/includes/Zend/` | `/usr/includes/Zend/Acl.php` |

## Interfaces

Some PSR define a set of commonly used interfaces. These interfaces are: Logger, Cache, SimpleCache,
Container, Link, EventDispatcher.

### PSR-3: Logger Interface

_PSR-3_ (accepted) describes simple and universal way to write logger interface.
The LoggerInterface exposes eight methods to write logs to the eight levels
(debug, info, notice, warning, error, critical, alert, emergency) and
one - to call one of those eight. I found one project, called [Monolog](https://github.com/Seldaek/monolog),
that implements this interface.

LoggerInterface:

```
interface LoggerInterface
{
    // System is unusable
    public function emergency($message, array $context = array());

    // Action must be taken immediately, e.g entire website down, database unavailable
    public function alert($message, array $context = array());

    // Application component unavailable, unexpected exception
    public function critical($message, array $context = array());

    // errors that do not require immediate action but should typically be logged and monitored
    public function error($message, array $context = array());

    // Use of deprecated APIs, poor use of an API
    public function warning($message, array $context = array());

    // Normal but significant events
    public function notice($message, array $context = array());

    // Interesting events
    // User logs in, SQL logs
    public function info($message, array $context = array());

    // Detailed debug information
    public function debug($message, array $context = array());

    // @param mixed $level
    public function log($level, $message, array $context = array());
}
```

### PSR-6: Caching Interface

Caching is a common way to improve the performance of any project.
This has lead to a situation where many libraries roll their own caching libraries.
These differences are causing developers to have to learn multiple systems which may or
may not provide the functionality they need.

_PSR-6_ describes two interfaces to implement such caching system: _CacheItemInterface_ and _CacheItemPoolInterface_.
CacheItemInterface defines an item inside a cache system. Each Item is associated with a specific key.

CacheItemInterface:

```
interface CacheItemInterface
{
    // Returns the key for the current cache item
    public function getKey();

    // Retrieves the value of the item from the cache associated with this object's key
    public function get();

    // Confirms if the cache item lookup resulted in a cache hit.
    public function isHit();

    // Sets the value represented by this cache item
    public function set($value);

    // Sets the expiration time for this cache item.
    // @param \DateTimeInterface|null $expiration - date after which item will expire.
    public function expiresAt($expiration);

    // Sets the expiration time for this cache item
    // @param int|\DateInterval|null $time - time in seconds after which item will expire.
    public function expiresAfter($time);
}
```

The primary purpose of CacheItemPoolInterface is to accept a key from the Calling Library
and return the associated CacheItemInterface object.

CacheItemPoolInterface:

```
interface CacheItemPoolInterface
{
    public function getItem($key);
    public function getItems(array $keys = array());
    public function hasItem($key);

    // Deletes all items in the pool
    public function clear();
    public function deleteItem($key);
    public function deleteItems(array $keys);
    public function save(CacheItemInterface $item);
    public function saveDeferred(CacheItemInterface $item);
    public function commit();
}
```

Implementation: [SymfonyCache](https://github.com/symfony/cache).

### PSR-16: Common Interface for Caching Libraries

_PSR-16_ describes one interface to cache data. PSR-16 solves the problem of standardization of framework-agnostic cache
in a too in-depth way for most use cases. PSR-16 gives us a layer of simplification.

Difference between them:

- PSR-6 can track for expired items
- PSR-6 has ability to queue/defer items (temporary store) and then commit them

CacheInterface:

```
interface CacheInterface
{
    public function get($key, $default = null);
    public function set($key, $value, $ttl = null);
    public function delete($key);
    public function clear();
    public function getMultiple($keys, $default = null);

    // @param iterable $values - A list of key => value pairs for a multiple-set operation.
    public function setMultiple($values, $ttl = null);
    public function deleteMultiple($keys);
    public function has($key);
}
```

Implemented by Symfony Cache, Laminas Cache.

### PSR-11: Container Interface

_PSR-11_ describes container interface. The goal set by the Container PSR is to standardize how frameworks and
libraries make use of a container to obtain objects and parameters.

ContainerInterface:

```
interface ContainerInterface
{
    public function get($id);
    public function has($id);
}
```

Implementations: PHP-DI.

### PSR-13: Hypermedia Links

_PSR-13_ aims to provide PHP developers with a simple, common way of representing a
hypermedia link independently of the serialization format that is used.

LinkInterface:

```
interface LinkInterface
{
    public function getHref();
    public function isTemplated();
    public function getRels();
    public function getAttributes();
}
```

LinkProviderInterface:

```
interface LinkProviderInterface
{
    public function getLinks();
    public function getLinksByRel($rel);
}
```

### PSR-14 Event Dispatcher

The goal of _PSR-14_ is to establish a common mechanism for event-based
extension and collaboration so that libraries.

EventDispatcherInterface:

```
interface EventDispatcherInterface
{
    // Provide all relevant listeners with an event to process.
    public function dispatch(object $event);
}
```

ListenerProviderInterface:

```
// Mapper from an event to the listeners that are applicable to that event
interface ListenerProviderInterface
{
    // An event for which to return the relevant listeners.
    public function getListenersForEvent(object $event) : iterable;
}
```

StoppableEventInterface:

```
// An Event whose processing may be interrupted when the event has been handled.
interface StoppableEventInterface
{
    // This will typically only be used by the Dispatcher to determine if the
    // previous listener halted propagation
    public function isPropagationStopped() : bool;
}
```

## HTTP interfaces

4 PSRs describe interfaces for working with HTTP.

### PSR-7: HTTP message interfaces

_PSR-7_ document describes common interfaces for representing HTTP messages.
PHP supports sending HTTP requests via several mechanisms, and for each of these
mechanism PSR-7 provides an interface.

The document defines 7 interfaces: MessageInterface, RequestInterface, ServerRequestInterface,
ResponseInterface, StreamInterface, UriInterface and UploadedFileInterface.

MessageInterface:

```
interface MessageInterface
{
    public function getProtocolVersion();
    public function withProtocolVersion($version);
    public function getHeaders();
    public function hasHeader($name);
    public function getHeader($name);
    public function getHeaderLine($name);
    public function withHeader($name, $value);
    public function withAddedHeader($name, $value);
    public function withoutHeader($name);
    public function getBody();
    public function withBody(StreamInterface $body);
}
```

Notice that RequestInterface, ResponseInterface and StreamInterface extend MessageInterface.
RequestInterface represents an outgoing, client-side request.

RequestInterface:

```
interface RequestInterface extends MessageInterface
{
    public function getRequestTarget();
    public function withRequestTarget($requestTarget);
    public function getMethod();
    public function withMethod($method);
    public function getUri();
    public function withUri(UriInterface $uri, $preserveHost = false);
}
```

ServerRequestInterface:

```
interface ServerRequestInterface extends RequestInterface
{
    public function getServerParams();
    public function getCookieParams();
    public function withCookieParams(array $cookies);
    public function getQueryParams();
    public function withQueryParams(array $query);
    public function getUploadedFiles();
    public function withUploadedFiles(array $uploadedFiles);
    public function getParsedBody();
    public function withParsedBody($data);
    public function getAttributes();
    public function getAttribute($name, $default = null);
    public function withAttribute($name, $value);
    public function withoutAttribute($name);
}
```

Implemented by Laminas Diactoros.

### PSR-15: HTTP Server Request Handlers

_PSR-15_ describes HTTP server request handlers and HTTP server middleware components.

RequestHandlerInterface:

```
// An HTTP request handler process an HTTP request in order to produce an HTTP response.
interface RequestHandlerInterface
{
    // Handles a request and produces a response.
    public function handle(ServerRequestInterface $request): ResponseInterface;
}
```

MiddlewareInterface:

```
// An HTTP middleware component participates in processing an HTTP message:
// by acting on the request, generating the response, or forwarding the
// request to a subsequent middleware and possibly acting on its response.
interface MiddlewareInterface
{
    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface;
}
```

Implemented by Mezzio.

### PSR-17: HTTP Factories

Describes how to create HTTP objects (request, response, ... - one factory per PSR-7 interface).
They are - RequestFactoryInterface, ResponseFactoryInterface, ServerRequestFactoryInterface,
StreamFactoryInterface, UploadedFileFactoryInterface и UriFactoryInterface.

Each interface has one function create\*, except StreamFactoryInterface - createStream, createStreamFromFile, createStreamFromResource.

```
interface RequestFactoryInterface
{
    public function createRequest(string $method, $uri): RequestInterface;
}
```

```
interface ResponseFactoryInterface
{
    public function createResponse(int $code = 200, string $reasonPhrase = ''): ResponseInterface;
}
```

### PSR-18: HTTP Client

Describes how to send requests and receive responses as well as handing exception.

```
// Sends a PSR-7 request and returns a PSR-7 response
interface ClientInterface
{
    public function sendRequest(RequestInterface $request): ResponseInterface;
}

interface ClientExceptionInterface extends \Throwable
{
}

// Exception for when a request failed
interface RequestExceptionInterface extends ClientExceptionInterface
{
    public function getRequest(): RequestInterface;
}

// Thrown when the request cannot be completed because of network issues.
// There is no response object as this exception is thrown when no response has been received.
interface NetworkExceptionInterface extends ClientExceptionInterface
{
    // The request object MAY be a different object from the one passed
    public function getRequest(): RequestInterface;
}
```

Implemented by Symfony HTTP-client.

## Coding styles

Coding style intend to help programmers to write programs in a consistent way.

### PSR-1: Basic Coding Standard

Coding style rules:

- PHP code MUST use the long `<?php ?>` tags or the short-echo `<?= ?>` tags
- Classes are written in 'StudlyCaps' - the first capital of each subword is capitalized;
- Class constants must be declared in all upper case with underscore separators;
- Method names must be declared in 'camelCase'
- Files should either declare symbols (classes, functions, constants, etc.) or cause side-effects (e.g. change .ini settings, include other files, etc.);

Tools to help follow these rules: `vscode-php-cs-fixer`.

### PSR-12

Specification extends, expands and replaces PSR-2. Code follows PSR-1.
Main rules:

- PHP files MUST use the Unix LF (linefeed) line ending only
- The closing `?>` tag MUST be omitted from files containing only PHP
- Lines SHOULD NOT be longer than 80 characters
- Visibility MUST be declared on all properties
- The keyword `elseif` should be used instead of `else if`

### PSR-2

PSR-2 is also coding standard, but has been deprecated, because it doesn't account for modern PHP features.
It extends PSR-1. PSR-12 additionally takes the following constructs into account:

- Traits
- `try-catch-finally`, compared to PSR-2's `try-catch`
- Anonymous classes
- Variadic three dot operator

## Summary

PSR standards solve code reusability problems. 4 PSR categories - interfaces, http, code style, autoloading

## Credicts

- https://www.php-fig.org/psr/
- https://habr.com/ru/post/458484/
- https://art-lemon.com/chto-takoe-php-fig
- https://en.wikipedia.org/wiki/PHP_Standard_Recommendation
- [PSR-2 vs PSR-12](https://orlando-thoeny.medium.com/differences-between-psr-12-and-psr-2-c859457d11a4)
