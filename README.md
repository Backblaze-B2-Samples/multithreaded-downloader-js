# MultiThreadedDownloader

A browser based multi-threaded downloader implemented in vanilla JavaScript.

-   Sends HTTP HEAD request to get the file info and calculate number of chunks
-   Sends HTTP GET requests with "Range: bytes=start-end" header for each chunk
-   Monitor the progress of each response stream
-   ~~Retry on fail or specific HTTP status codes (like 503 Service Unavailable)~~
-   ~~Concatenates each response stream (in order) into a final output stream~~
-   ~~Uses [StreamSaver.js](https://github.com/jimmywarting/StreamSaver.js) to simplify downloading the output stream.~~
-   Uses [bro-fs](https://github.com/vitalets/bro-fs)(HTML5 Filesystem api) to temporarily save each chunk.
-   Concatenates all chunks once complete and triggers a[download] with final file.
-   100% client side JavaScript, no plug-ins or proxy required

This project is under development and still has some bugs.

[Demos](https://backblaze-b2-samples.github.io/multithreaded-downloader-js/)

## Backblaze B2

If using the B2 command line tool, remember to escape JSON with single quotes.
The B2 bucket must be "Public" and have the following CORS rules applied:
```
    [{
      "corsRuleName": "someName",
      "allowedOrigins": [
        "*"
      ],
      "allowedOperations": [
        "b2_download_file_by_id",
        "b2_download_file_by_name"
      ],
      "allowedHeaders": [
        "*"
      ],
      "exposeHeaders": [
        "Access-Control-Allow-Origin",
        "Content-Length"
      ],
      "maxAgeSeconds": 10240
    }]
```

## Usage

The MultiThread constructor accepts an object:
```
new MultiThread({
  // The request url
  url: 'http://some-url/',

  // Request headers to pass-though
  headers: {
    'Authorization': `Bearer ${accessToken}`
  },

  // The final output fileName
  fileName: 'some-file.ext',

  // Number of concurrent request threads
  threads: 6,

  // Size of each chunk in MB
  chunkSize: 4,

  // Number of retry attempts
  retries: 2,

  // Delay before another retry attempt in ms
  retryDelay: 1000,
})
```

### Callbacks
The constructor options can also have onStart, onFinish, & onProgress callbacks for the main download,
and onChunkStart, onChunkFinish, & onChunkProgress for each chunk.
```
// Callbacks for main download
onStart({contentLength, chunks})
onFinish({contentLength})
onProgress({contentLength, loaded, started})

// Callback for each chunk
onChunkStart({contentLength, id})
onChunkFinish({contentLength, id})
onChunkProgress({contentLength, loaded, id})
```

### Goals:

Fetches parts of a file using the HTTP Range header and downloads those pieces in parallel. When the pieces have all been downloaded, the original file is re-assembled and saved in the browser's Downloads folder.

-   The downloader should fetch the file directly from the web browser. No server will be needed to proxy the file.
-   The download process should not need any client software to be installed. Nor will a browser plugin be required.
-   This project shall allow for resuming an interrupted download, or at least retrying a part of the file that was interrupted.
-   Optionally, this project will allow us to specify the number of download threads and the size of each request... so we can tune it for specific network conditions, if that is necessary.

#### Dependencies
-   ~~[Web Streams Polyfill](https://github.com/creatorrr/web-streams-polyfill)~~
-   ~~[StreamSaver](https://github.com/jimmywarting/StreamSaver.js)~~
-   [bro-fs](https://github.com/vitalets/bro-fs)

#### Reference
-   [Web Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API)
-   [Web Streams Spec](https://streams.spec.whatwg.org/)
-   [Parallel chunk requests in a browser via Service Workers](https://blog.ghaiklor.com/parallel-chunk-requests-in-a-browser-via-service-workers-7be10be2b75f)
-   [browser-server](https://github.com/mafintosh/browser-server)
-   [fetch-retry](https://github.com/jonbern/fetch-retry)
-   [Pipes.js](http://pipes.js.org/)
