# binc

binc file format [specification](SPECIFICATION.md)

## What is binc?

binc is a **bin**ary, **inc**remental data format for storing and transmitting structured data.

Similar to file formats like XML and RIFF, binc is a generic format that can be used to store any kind of data. 
But rather than storing the current state of the data, binc stores the history of changes which led to that state.

You could think of it is as a file which contains its own (simple) version control system.

## Why should you consider using binc? 

### Unified format makes things simpler

Features like undo/redo, revision history, distributed storage and real-time collaboration generally rely on the ability to incrementally update the application state. 
By having a storage model which itself is incremental, you don't need to "bolt" anything on top of it to support these use-cases.
Having a common format also brings the opportunity to share tools, libraries and infrastructure between applications.

### Efficient & Robust

binc uses a compact binary encoding which is efficient in terms of both bytes and CPU cycles. Since files are only ever appended to, and never modified, it's easy to recover from errors.

### Usable everywhere

binc can be implemented in any programming language, has MIT license. A binc can implementation can conceivably be implemented by a single developer in a short amount of time.

## Roadmap

The version 1 specification was deliberately kept to a minimal viable version of the format.

Various other features are considered for future versions, including:

* Snapshots
* Checksums
* Tags
* Comments
* More attribute types including arrays & BLOBs with partial updates
* Network-protocol & application agnostic server
* Embedding image & audio data
* Referencing other files
* Application-specific change types

## Implementations

* [binc-rs](https://github.com/kurasu/binc-rs) Rust implementation where experiments using the format are being done. 
* [binc-java](https://github.com/kurasu/binc-java) Java implementation of the exact v1 specification.
  
## Contributing

Contributions are welcome! Please open an issue or pull request.
