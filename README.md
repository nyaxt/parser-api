Streaming HTML Parser API
===

Streaming HTML Parser API allows web developers to parse and construct DOM objects asynchronously through [Streams](https://streams.spec.whatwg.org/).

## Motivation
- Streaming HTML Parsing is faster than synchronous parsing (innerHTML setter)
  - Browser will block main thread while “innerHTML” parsing, so browser can’t execute javascript/generate new frame while parsing innerHTML.
  - This is a burden on 60FPS animation while loading.
- There is no clean/complete API for streaming HTML parsing: not modern && hard to use.
  - XMLHttpRequest.document
    - Blink’s XMLHTTPRequest implementation asynchronously parse incoming body in background when responseType=’document’ is specified.
    - [Con] Limited to resource fetch over HTTP/HTTPS
    - [Con] This feature is not available through Fetch API
    - [Con] Does not work on documents from service worker cache
  - ```<iframe srcdoc>```
    - [Pro] Firefox also supports this.
    - [Con] The iframe needs to be displayed, thus create full render-tree, not just DOM.

## Example use cases
- Background content loading without degrading UI responsiveness
  - e.g.) New tweets are loaded while user scroll down the page.

## Streams integration idea
### Input data as strings

```
var parser = new DocumentParser();
var writableStream = parser.writableStream;
var writer = writableStream.getWriter();
writer.write(chunk);
...
writer.write(chunk);
writer.close();

parser.ready.then(() => {
  process(parser.document);
});
```

### Input data directly from Fetch API
```
// alternate idea: var parser = new DocumentParser({ inputType: “bytes” })

var readableStream = response.body;
var encoder = new TextEncodingTransformer();
readableStream.pipeThrough(encoder).pipeTo(parser.writableStream);

parser.ready.then(() => {
process(parser.document);
});

// cancel / abort is needed?
```
