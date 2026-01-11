Serde YAML
==========

[<img alt="github" src="https://img.shields.io/badge/github-AndreaBozzo/serde--yaml-8da0cb?style=for-the-badge&labelColor=555555&logo=github" height="20">](https://github.com/AndreaBozzo/serde-yaml)
[<img alt="build status" src="https://img.shields.io/github/actions/workflow/status/AndreaBozzo/serde-yaml/ci.yml?branch=master&style=for-the-badge" height="20">](https://github.com/AndreaBozzo/serde-yaml/actions?query=branch%3Amaster)

Rust library for using the [Serde] serialization framework with data in [YAML]
file format.

[Serde]: https://github.com/serde-rs/serde
[YAML]: https://yaml.org/

## About this fork

This is a personal fork of the original [serde-yaml](https://github.com/dtolnay/serde-yaml) by David Tolnay, forked from commit [`2009506`](https://github.com/dtolnay/serde-yaml/commit/2009506) (Release 0.9.34). The original repository was archived and is no longer maintained.

This fork aims to provide continued maintenance with minimal, focused changes. It ports improvements from [serde-yaml-ng](https://github.com/acatton/serde-yaml-ng), whose maintainer [works on his own terms](https://github.com/acatton/serde-yaml-ng#why) as a personal project. This fork exists as a parallel effort for my own use. But as i firmly believe into open formats, data and cooperation, this is open to contributors, we might even try to release a build after migrating to libyaml-safer.

**Changes from original:**
- Rust 1.82+ required (was 1.64)
- `impl From<Error> for io::Error` for easier error handling
- Resolved internal FIXMEs using stable APIs (`Box::new_uninit`)
- Clippy fixes and removed unnecessary dev-dependencies
- Simplified CI (fmt, clippy, test) — `clippy::pedantic` disabled as warnings are style-only (lifetime elision, `let...else` syntax) with no functional issues


## Dependency

```toml
[dependencies]
serde = "1.0"
serde_yaml = "0.9"
```

Release notes are available under [GitHub releases].

[GitHub releases]: https://github.com/AndreaBozzo/serde-yaml/releases

## Using Serde YAML

[API documentation is available in rustdoc form][docs.rs] but the general idea
is:

[docs.rs]: https://docs.rs/serde_yaml

```rust
use std::collections::BTreeMap;

fn main() -> Result<(), serde_yaml::Error> {
    // You have some type.
    let mut map = BTreeMap::new();
    map.insert("x".to_string(), 1.0);
    map.insert("y".to_string(), 2.0);

    // Serialize it to a YAML string.
    let yaml = serde_yaml::to_string(&map)?;
    assert_eq!(yaml, "x: 1.0\ny: 2.0\n");

    // Deserialize it back to a Rust type.
    let deserialized_map: BTreeMap<String, f64> = serde_yaml::from_str(&yaml)?;
    assert_eq!(map, deserialized_map);
    Ok(())
}
```

It can also be used with Serde's derive macros to handle structs and enums
defined in your program.

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_yaml = "0.9"
```

Structs serialize in the obvious way:

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, PartialEq, Serialize, Deserialize)]
struct Point {
    x: f64,
    y: f64,
}

fn main() -> Result<(), serde_yaml::Error> {
    let point = Point { x: 1.0, y: 2.0 };

    let yaml = serde_yaml::to_string(&point)?;
    assert_eq!(yaml, "x: 1.0\ny: 2.0\n");

    let deserialized_point: Point = serde_yaml::from_str(&yaml)?;
    assert_eq!(point, deserialized_point);
    Ok(())
}
```

Enums serialize using YAML's `!tag` syntax to identify the variant name.

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, PartialEq, Debug)]
enum Enum {
    Unit,
    Newtype(usize),
    Tuple(usize, usize, usize),
    Struct { x: f64, y: f64 },
}

fn main() -> Result<(), serde_yaml::Error> {
    let yaml = "
        - !Newtype 1
        - !Tuple [0, 0, 0]
        - !Struct {x: 1.0, y: 2.0}
    ";
    let values: Vec<Enum> = serde_yaml::from_str(yaml).unwrap();
    assert_eq!(values[0], Enum::Newtype(1));
    assert_eq!(values[1], Enum::Tuple(0, 0, 0));
    assert_eq!(values[2], Enum::Struct { x: 1.0, y: 2.0 });

    // The last two in YAML's block style instead:
    let yaml = "
        - !Tuple
          - 0
          - 0
          - 0
        - !Struct
          x: 1.0
          y: 2.0
    ";
    let values: Vec<Enum> = serde_yaml::from_str(yaml).unwrap();
    assert_eq!(values[0], Enum::Tuple(0, 0, 0));
    assert_eq!(values[1], Enum::Struct { x: 1.0, y: 2.0 });

    // Variants with no data can be written using !Tag or just the string name.
    let yaml = "
        - Unit  # serialization produces this one
        - !Unit
    ";
    let values: Vec<Enum> = serde_yaml::from_str(yaml).unwrap();
    assert_eq!(values[0], Enum::Unit);
    assert_eq!(values[1], Enum::Unit);

    Ok(())
}
```

<br>

#### License

<sup>
Licensed under either of <a href="LICENSE-APACHE">Apache License, Version
2.0</a> or <a href="LICENSE-MIT">MIT license</a> at your option.
</sup>

<br>

<sub>
Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this crate by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
</sub>
