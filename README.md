<div align="center">

  <h1><code>Arbitrary</code></h1>

  <p><strong>The trait for generating structured data from arbitrary, unstructured input.</strong></p>

  <img alt="GitHub Actions Status" src="https://github.com/rust-fuzz/rust_arbitrary/workflows/Rust/badge.svg"/>

</div>

## About

The primary purpose of this crate is to provide some way to generate structures
out of the raw byte buffers that various fuzzers produce. This allows you to
combine structure-aware test case generation with coverage-guided,
mutation-based fuzzers.

## Example

Say you're writing a color conversion library, and you have an `Rgb` struct to
represent RGB colors. You might want to implement `Arbitrary` for `Rgb` so that
you could take arbitrary `Rgb` instances in a test function that asserts some
property (for example, asserting that RGB converted to HSL and converted back to
RGB always ends up exactly where we started).

```rust
use arbitrary::{Arbitrary, Unstructured};

pub struct Rgb {
    pub r: u8,
    pub g: u8,
    pub b: u8,
}

impl Arbitrary for Rgb {
    fn arbitrary<U>(raw: &mut U) -> Result<Self, U::Error>
    where
        U: Unstructured + ?Sized>,
    {
        let r = u8::arbitrary(raw)?;
        let g = u8::arbitrary(raw)?;
        let b = u8::arbitrary(raw)?;
        Ok(Rgb { r, g, b })
    }
}
```

## Shrinking

To assist with test case reduction, where you want to find the smallest and most
easily understandable test case that still demonstrates a bug you've discovered,
the `Arbitrary` trait has a `shrink` method. The `shrink` method returns an
iterator of "smaller" instances of `self`. The provided, default implementation
returns an empty iterator.

We can override the default for our `Rgb` struct above by shrinking each of its
components and then gluing them back together again:

```rust
impl Arbitrary for Rgb {
    // ...

    fn shrink(&self) -> Box<dyn Iterator<Item = Self>> {
        let rs = self.r.shrink();
        let gs = self.g.shrink();
        let bs = self.b.shrink();
        Box::new(rs.zip(gs).zip(bs).map(|((r, g), b)| Rgb { r, g, b }))
    }
}
```

## License

Licensed under dual MIT or Apache-2.0 at your choice.

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this project by you, as defined in the Apache-2.0 license,
shall be dual licensed as above, without any additional terms or conditions.
