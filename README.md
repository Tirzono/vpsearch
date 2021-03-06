# VP-tree nearest neighbor search

A relatively simple and readable Rust implementation of Vantage Point tree search algorithm.

The VP tree algorithm doesn't need to know coordinates of items, only distances between them. It can efficiently search multi-dimensional spaces and abstract things as long as you can define similarity between them (e.g. points, colors, and even images).

Please see [the API reference](https://docs.rs/vpsearch) or [examples](https://github.com/kornelski/vpsearch/tree/rust/examples) for details.

**This algorithm does not work with squared distances. When implementing Euclidean distance, you *MUST* use `sqrt()`**. Vantage Point trees require [metric spaces](https://en.wikipedia.org/wiki/Metric_space).

```Rust
#[derive(Copy, Clone)]
struct Point {
    x: f32, y: f32,
}

/// `MetricSpace` makes items comparable. It's a bit like Rust's `PartialOrd`.
impl vpsearch::MetricSpace for Point {
    type UserData = ();
    type Distance = f32;

    fn distance(&self, other: &Self, _: &Self::UserData) -> Self::Distance {
        let dx = self.x - other.x;
        let dy = self.y - other.y;

        // You MUST use sqrt here! The algorithm will give wrong results for squared distances.
        (dx*dx + dy*dy).sqrt()
    }
}

fn main() {
    let points = vec![Point{x:2.0,y:3.0}, Point{x:0.0,y:1.0}, Point{x:4.0,y:5.0}];

    let vp = vpsearch::Tree::new(&points);
    let (index, _) = vp.find_nearest(&Point{x:1.0,y:2.0});

    println!("The nearest point is at ({}, {})", points[index].x, points[index].y);
}
```

## Implementing `MetricSpace` for Rust built-in types

This library includes a workaround for orphan rules. You need to add your crate's type when implementing `MetricSpace`:

```rust
struct MyImpl; // it can be any type, as long as it's yours
impl vpsearch::MetricSpace<MyImpl> for Vec<u8> {
    // continue as usual
}
```

## Memory efficiency tip

`Tree` clones all the items and puts them in the tree. If the items are too big to clone and you'd rather keep the items elsewhere, you can!

Instead of storing the items, make the tree store indices into your items storage, and pass actual items as `user_data` to your `MetricSpace::distance()` function.

```rust
let items = /* your actual items are here */;
let indices: Vec<_> = (0...items.len() as u32).collect();
let tree = Tree::new_with_user_data_ref(&items, &indices);
let res = tree.find_nearest(&needle, &items);
```
