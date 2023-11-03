---
title: "Rustで区間(interval)を扱うcrateの調査"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Rust]
published: true
---

## 
rustでinterval setのcrateについて調べた。

次の視点で評価した。
- 整数・浮動小数点に限らず全順序集合ならなんでも扱いたい
- 閉区間・開区間・片閉区間などなんでも扱えると良い
- 交差・合併などの演算ができるか

### 結論

長期間放置されているものが多い。これさえ使っておけばいいというのは見つかってない。


## crate紹介

### interval_set

- https://crates.io/crates/interval_set
- u32しかできない
- 最終更新2018年
- 閉区間のみ
- 演算はIntervalSetだとできる
- 単独のIntervalだと演算はできない。ちょっと設計がこなれてない感はある

```rust
use interval_set::{Interval, ToIntervalSet};

fn test() {
    let i = Interval::new(0, 20);
    println!("{:?}", i.get_sup());
    // 20

    let si = vec![(0, 20)].to_interval_set();
    let sj = vec![(10, 30)].to_interval_set();
    let intersect = si.intersection(sj);
    println!("{:?}", intersect);
    // IntervalSet { intervals: [Interval(10, 20)] }

    let si = vec![(0, 20)].to_interval_set();
    let sj = vec![(20, 30)].to_interval_set();
    let intersect = si.intersection(sj);
    println!("{:?}", intersect);
    // IntervalSet { intervals: [Interval(20, 20)] }

    let si = vec![(0, 10), (20, 30)].to_interval_set();
    let sj = vec![(5, 25)].to_interval_set();
    let intersect = si.intersection(sj);
    println!("{:?}", intersect);
    // IntervalSet { intervals: [Interval(5, 10), Interval(20, 25)] }
}
```

### nested_intervals

- nested_intervals
- 最終更新2019年
- u32のみ
- 閉区間のみ
- 区間が交差したり被覆した状態を表現できるようだが交差が未実装というのは使いにくい

```rust
use nested_intervals::IntervalSet;
use std::ops::Range;


fn test() {
    let mut i = IntervalSet::new(&[0..10, 20..30]).unwrap();
    let r: Range<u32> = 5..15;
    println!("{}", i.has_overlap(&r).unwrap());
    // IntervalSet { intervals: [0..20], ids: [[0, 1]], root: None }
    // true
    let r: Range<u32> = 10..15;
    println!("{}", i.has_overlap(&r).unwrap());
    // false

    let j = IntervalSet::new(&[0..10, 10..20]).unwrap();
    println!("{}", j.any_overlapping());
    // false

    let j = IntervalSet::new(&[0..10, 5..20]).unwrap();
    println!("{}", j.any_overlapping());
    // true
    println!("{:?}", j.merge_hull());
    // IntervalSet { intervals: [0..20], ids: [[0, 1]], root: None }
}
```

### intervallum

- https://crates.io/crates/intervallum
- 最終更新2021年
- Genericになっているが整数型しかダメっぽい
- 閉区間のみ
- 演算の用意は多い

```rust
use gcollections::ops::{Intersection, Join, Overlap, Union};
use interval::interval_set::*;
use interval::ops::*;
use interval::Interval;


fn test() {
    let x = Interval::new(0, 5);
    let y = Interval::new(3, 7);
    assert_eq!(x.intersection(&y), Interval::new(3, 5));

    let x = Interval::new(0, 3);
    let y = Interval::new(5, 7);
    assert_eq!(x.hull(&y), Interval::new(0, 7));
    assert_eq!(x.join(y), Interval::new(5, 3)); // なにこれ

    let x = IntervalSet::new(0, 3);
    let y = IntervalSet::new(5, 7);
    assert_eq!(x.union(&y), vec![(0, 3), (5, 7)].to_interval_set());

    let x = Interval::new(0, 5);
    let y = Interval::new(5, 7);
    assert_eq!(x.intersection(&y), Interval::new(5, 5));
    assert_eq!(x + y, Interval::new(5, 12));
    assert_eq!(x.join(y), Interval::new(5, 5));
    assert_eq!(x.hull(&y), Interval::new(0, 7));

    let a = vec![(1, 4), (6, 10)].to_interval_set();
    assert!(a.overlap(&IntervalSet::new(7, 8)));

    let b = vec![(3, 5), (9, 12)].to_interval_set();
    assert_eq!(a.intersection(&b), vec![(3, 4), (9, 10)].to_interval_set());
}
```

### normalize_interval

- https://crates.io/crates/normalize_interval
- 更新2020年
- 整数型のみ対応
- 端点の設定は柔軟
    - しかし整数型のみなのでいまいち
- 複数区間はイテレータで表現されてinterval set自体を表すデータ構造はない

```rust
 use itertools::Itertools;
use normalize_interval::Bound::{Exclude, Include};
use normalize_interval::Interval;


fn test() {
    // NG
    // let f: Interval<f64> = Interval::new(Include(3.0), Exclude(7.0));
    // let f: Interval<Utc> =
    //     Interval::new(Include(Utc::now()), Exclude(Utc::now().add(Hour)));

    let i: Interval<i32> = Interval::new(Include(3), Exclude(7));
    let j: Interval<i32> = Interval::closed(7, 10);
    assert_eq!(i.intersect(&j), Interval::empty());

    let i: Interval<i32> = Interval::new(Include(3), Include(7));
    let j: Interval<i32> = Interval::right_open(7, 10);
    assert_eq!(i.intersect(&j), Interval::point(7));

    let i: Interval<i32> = Interval::open(1, 5);
    assert!(!i.intersects(&Interval::open(2, 3)));
    assert!(i.intersects(&Interval::open(2, 6)));

    assert_eq!(
        Interval::open(1, 5)
            .union(&Interval::open(8, 10))
            .collect_vec(),
        vec![Interval::closed(2, 4), Interval::open(8, 10)]
    );

    assert_eq!(
        Interval::open(1, 5)
            .union(&Interval::open(1, 5))
            .collect_vec(),
        vec![Interval::closed(2, 4)]
    );
    assert_eq!(
        Interval::open(1, 5)
            .union(&Interval::open(1, 5))
            .collect_vec(),
        vec![Interval::open(1, 5)]
    );
}
```

### interval-general

- https://crates.io/crates/intervals-general
- 最終更新2019
- データ構造は工夫されている
    - 半開区間や上界なしなどなんでもある
- 整数型だけでなく浮動小数点やchronoの日時型も対応
- 演算機能が貧弱。unionもない
- interval set相当のものがない

```rust
        use chrono::{DateTime, Duration, Utc};
use intervals_general::bound_pair::BoundPair;
use intervals_general::interval::Interval;

#[test]
fn test() {
    let i: Interval<i64> = Interval::Open {
        bound_pair: BoundPair::new(10, 20).unwrap(),
    };
    let j: Interval<i64> = Interval::Open {
        bound_pair: BoundPair::new(15, 25).unwrap(),
    };
    assert_eq!(
        i.intersect(&j),
        Interval::Open {
            bound_pair: BoundPair::new(15, 20).unwrap()
        }
    );

    let i: Interval<f64> = Interval::Singleton { at: 1.0 };
    let j: Interval<f64> = Interval::Open {
        bound_pair: BoundPair::new(0.5, 1.5).unwrap(),
    };
    assert!(j.contains(&i));

    let now = Utc::now();
    let p: Interval<DateTime<Utc>> = Interval::Singleton { at: Utc::now() };
    let i: Interval<DateTime<Utc>> = Interval::Open {
        bound_pair: BoundPair::new(now - Duration::hours(1), now + Duration::hours(1))
            .unwrap(),
    };
    let j: Interval<DateTime<Utc>> = Interval::Open {
        bound_pair: BoundPair::new(
            now - Duration::minutes(30),
            now + Duration::minutes(90),
        )
            .unwrap(),
    };
    assert!(i.intersect(&j).contains(&p))
}

```

### honestintervals

- https://crates.io/crates/honestintervals
- 最終更新2017
- 整数型はダメ？
- 文字列のパースもできる
- 型だけ作って演算が作られてない
- mpfrという外部ライブラリが必要

```rust

use honestintervals::{Interval, IntervalSet};

#[test]
fn test() {
    let i = Interval::new(0.0, 10.0);
    let j = Interval::new(5.0, 15.0);

    let i = IntervalSet::new(0.0, 10.0);
    let j = IntervalSet::new(5.0, 15.0);
}
```