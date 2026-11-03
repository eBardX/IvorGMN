# IvorGMN

A Guido Music Notation parser, normalizer, validator, and formatter.

[![Swift 6.3](https://img.shields.io/badge/Swift-6.3-orange.svg)](https://swift.org)
[![Platforms](https://img.shields.io/badge/platforms-iOS%20%7C%20macOS-lightgrey.svg)](https://developer.apple.com)
[![SwiftPM](https://img.shields.io/badge/SwiftPM-compatible-brightgreen.svg)](https://swift.org/package-manager/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/eBardX/IvorGMN/blob/main/LICENSE.md)

* [Overview](#overview)
* [Requirements](#requirements)
* [Installation](#installation)
    * [Swift Package Manager](#spm_installation)
* [Quick Start](#quick_start)
* [Documentation](#documentation)
* [Reference Documentation](#reference_documentation)
* [Credits](#credits)
* [License](#license)

## <a name="overview">Overview</a>

The IvorGMN framework provides a [Guido Music Notation (GMN)][gmn] parser,
normalizer, validator, and formatter written in Swift, with a
strict-concurrency-ready, value-type API.

IvorGMN's syntactic core turns GMN text into a typed, round-trippable abstract
syntax tree (`GMNScore` → `GMNVoice` → `GMNSymbol`) and back to text, without
interpreting the music. Tags are typed too: `GMNTag` is an enum over 59 payload
structs — `GMNClef` knows it has a type and a line, `GMNTempo` knows it has a
text and a metronome spec — with two untyped lanes that carry, losslessly,
everything else: `GMNReservedTag` for a name GMN reserves that the catalog
models no payload for, and `GMNCustomTag` for a name GMN does not reserve at
all.

Everything flows through a small, explicit pipeline of four value types, each
with a no-argument initializer:

 Stage     | Type            | Input → Output
:-----     |:----            |:--------------
 Parse     | `GMNParser`     | `Data` → `GMNScore`
 Normalize | `GMNNormalizer` | `GMNScore` → `GMNScore` (canonical spellings)
 Validate  | `GMNValidator`  | `GMNScore` → validated `GMNScore`
 Format    | `GMNFormatter`  | `GMNScore` → `Data`

The pipeline is gated by two flags on `GMNScore`: a score must be normalized
before it can be validated, and validated before it can be formatted. Unlike
ABC notation, which groups many tunes into a tunebook, GMN describes a single
score with multiple voices, so there is no tunebook-equivalent type. See the
[usage guide][guide] for a full walkthrough of the API.

## <a name="requirements">Requirements</a>

* iOS 18.0+ / macOS 15.0+
* Swift 6.3 toolchain
* Swift 6 language mode

## <a name="installation">Installation</a>

### <a name="spm_installation">Swift Package Manager</a>

IvorGMN is distributed exclusively through the [Swift Package Manager][spm].

To add IvorGMN to a Swift package, add it to the `dependencies` in your
`Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/eBardX/IvorGMN.git",
             .upToNextMinor(from: "0.1.0"))
]
```

Then add `IvorGMN` to the dependencies of any target that uses it:

```swift
.target(name: "MyTarget",
        dependencies: [.product(name: "IvorGMN",
                                package: "IvorGMN")])
```

To add IvorGMN to an Xcode project, choose **File ▸ Add Package Dependencies…**
and enter the repository URL:

```
https://github.com/eBardX/IvorGMN.git
```

IvorGMN depends on [XestiTokens][xestitokens] and [XestiTools][xestitools]; the
Swift Package Manager resolves both automatically.

## <a name="quick_start">Quick Start</a>

Take GMN text from `Data` through the full pipeline and back to `Data`:

```swift
import Foundation
import IvorGMN

let data = try Data(contentsOf: url)

// 1. Parse GMN text into a typed AST.
let (parsed, diagnostics) = try GMNParser().parse(data)

// 2. Normalize tag-name aliases and parameter renames to their canonical form.
let (normalized, changes) = GMNNormalizer().normalize(parsed)

// 3. Validate — required parameters, readable values, and tag bodies.
let (validated, issues) = try GMNValidator().validate(normalized)

guard issues.isEmpty
else { issues.forEach { print($0.message) }; return }

// 4. Format back to GMN text.
let output = try GMNFormatter().format(validated)
```

Each stage is independent, so you can stop at the AST or round-trip through
the formatter. For the complete story — the AST model and error handling — see
the [usage guide][guide].

## <a name="documentation">Documentation</a>

* [Using IvorGMN][guide] — a guide to using the public API, published as part
  of the DocC documentation.
* Every public declaration carries a DocC comment describing its Guido Music
  Notation behavior.

## <a name="reference_documentation">Reference Documentation</a>

Full [reference documentation][refdoc] is available courtesy of [DocC][docc].

## <a name="credits">Credits</a>

John Gary Pusey (ebardx@gmail.com)

## <a name="license">License</a>

IvorGMN is available under [the MIT license][license].

[docc]:         https://www.swift.org/documentation/docc/
[gmn]:          https://guidodoc.grame.fr
[guide]:        https://eBardX.github.io/ivor-packages-docs/documentation/ivorgmn/usingivorgmn
[license]:      https://github.com/eBardX/IvorGMN/blob/main/LICENSE.md
[refdoc]:       https://eBardX.github.io/ivor-packages-docs/documentation/ivorgmn
[spm]:          https://swift.org/package-manager/
[xestitokens]:  https://github.com/eBardX/XestiTokens
[xestitools]:   https://github.com/eBardX/XestiTools
