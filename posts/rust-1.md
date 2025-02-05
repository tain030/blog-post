---
title: 'Rust 입출력: 효율적인 데이터 처리 방법'
date: '2025-02-01'
category: 'rust'
---

### 목차

1. Rust 입출력 개요
2. 입력 처리 방법
   - 간단한 단일 입력: `io::stdin()`
   - 반복적인 입력: `io::stdin().lock()`
   - 대량 데이터/다중 입력 처리: `BufReader`
3. 출력 처리 방법
   - 간단한 단일 출력: `println!()` 또는 `io::stdout()`
   - 반복적인 출력: `io::stdout().lock()`
   - 대량 데이터/다중 출력 처리: `BufWriter`
4. 결론

---

https://blog.kakaocdn.net/dn/rAIay/btszbtfDapI/MkIZTYFnJYr9JuOVm5smgK/img.jpg

Rust의 입출력은 크게 3가지로 나눌 수 있습니다.

- **간단한 단일 입출력**
- **반복적인 입출력**
- **대량 데이터/다중 입출력 처리**

이 3가지 방법의 특징을 알아두고 상황에 맞게 알맞은 입출력 방식을 사용한다면, 알고리즘 문제에서 입출력으로 발생하는 '시간 초과'가 발생는 문제나 코드의 '비효율'을 방지할 수 있습니다.

입출력 시 비효율을 유발하는 주요 원인은 두 가지로 요약할 수 있습니다.

1. read, write할 때마다 lock을 잠그고 해제하는지 여부
2. 스트림으로 바로바로 입출력하는지, 버퍼에 쌓았다가 한번에 출력하는지 여부

먼저 입력에 대해 알아보겠습니다.


## 입력

### 간단한 단일 입력: `io::stdin()`

- 적합한 상황: "Hello World"와 같은 짧은 한두 줄의 입력 처리.
- 특징: 간단한 입력에 적합하지만, 반복적인 입력 처리에는 비효율적일 수 있습니다.

```rust
use std::io;

fn main() {
    let mut input = String::new();
    io::stdin().read_line(&mut input).unwrap();
    println!("You entered: {}", input);
}
```


### 반복적인 입력: `io::stdin().lock()`

- 적합한 상황: 몇십 줄에서 수백 줄의 텍스트, 1MB 미만의 데이터 처리.
- 특징: lock()을 사용하여 반복적인 입력 처리 시 lock/unlock 오버헤드를 줄입니다.

```rust
use std::io::{self, BufRead};

fn main() {
    let stdin = io::stdin();
    let handle = stdin.lock();

    for line in handle.lines() {
        println!("Read line: {}", line.unwrap());
    }
}
```


### 대량 데이터/다중 입력 처리: `BufReader`

- 적합한 상황: 로그 파일, 대형 CSV/JSON 파일, 스트림 데이터(수천 줄 이상), 1MB 이상의 데이터 처리.
- 특징: 버퍼링을 통해 대량의 데이터를 효율적으로 처리합니다.

```rust
use std::io::{self, BufRead, BufReader};
use std::fs::File;

fn main() {
    let stdin = io::stdin();
    let reader = BufReader::new(stdin.lock());

    for line in reader.lines() {
        println!("Read line: {}", line?);
    }
}
```

**입력 데이터가 남아있을 가능성**에 대한 걱정은 프로그램이 **정상적으로 종료되기 전에** 모든 데이터를 **읽어오는 로직**을 추가함으로써 방지할 수 있습니다. 예를 들어, 사용자가 입력을 다 마친 후 종료될 수 있도록 하거나, 데이터를 **반복적으로 읽어오는 로직**을 설계하는 방식이 있습니다.

### 입력 관련 메서드

- **한 줄 처리**: `read_line()` 또는 `lines()`.
- **다수의 줄**: `lines()` (효율적이고 간결).
- **바이너리 데이터**: `read()` 또는 `read_until()`.
- **전체 스트림 읽기**: `read_to_string()`.



## 출력

### 간단한 단일 출력: `println!()` 또는 `io::stdout()`

- 적합한 상황: 간단한 메시지 출력.
- 특징: 편리하지만, 반복적인 출력에는 비효율적일 수 있습니다.

```rust
fn main() {
    println!("Hello, World!");
}
```

### 반복적인 출력: `io::stdout().lock()`

- 적합한 상황: 반복적인 출력이 필요한 경우.
- 특징: lock()을 사용하여 반복적인 출력 시 lock/unlock 오버헤드를 줄입니다.

```rust
use std::io::{self, Write};

fn main() {
    let stdout = io::stdout();
    let mut handle = stdout.lock();

    for i in 0..5 {
        writeln!(handle, "Line: {}", i).unwrap();
    }
}
```


### 대량 데이터/다중 출력 처리: `BufWriter`

- 적합한 상황: 대량의 데이터를 반복적으로 출력해야 하는 경우.
- 특징: 기본적으로 8KB의 내부 버퍼를 사용하여 데이터를 모아 한 번에 출력합니다.

```rust
use std::io::{self, Write, BufWriter};

fn main() {
    let stdout = io::stdout();
    let mut writer = BufWriter::new(stdout.lock());

    for i in 0..5 {
        writeln!(writer, "Optimized Line: {}", i).unwrap();
    }

    writer.flush().unwrap(); // 버퍼 내용을 한 번에 출력
}
```

`BufWriter`는 기본적으로 8192 바이트(8 KB)의 내부 버퍼를 가지고 있습니다.

버퍼가 가득 차면 `BufWriter`는 자동으로 버퍼의 내용을 출력 스트림으로 내보냅니다. 즉, 버퍼가 가득 찼을 때 flush를 명시적으로 호출하지 않아도 데이터가 손실되거나 계속 쌓이는 문제는 없습니다.

`BufWriter`를 사용할 때 모든 데이터가 스트림에 기록되었는지 보장하기 위해 마지막에는 항상 `flush()`를 호출하세요. `Drop` 구현을 통해 스코프를 벗어나거나 프로그램이 종료될 때 남아 있는 데이터를 플러시하지만 명시적으로 `flush()`를 호출하는 것이 권장됩니다.


> `write!()`는 내부적으로 `Write` 트레이트의 `.write()`를 호출하므로, **어떤 스트림에 쓰는지**에 따라 출력 방식이 달라집니다.
>
> 즉, 버퍼링 되지 않은 출력을 사용하는 경우 `write!()`를 호출할 때마다 스트림에 바로 데이터가 쓰입니다. 하지만 버퍼링된 출력을 사용하는 경우 `write!()`가 데이터를 버퍼에 쓰고, 버퍼가 꽉 차거나 `flush()`가 호출될 때만 실제 출력이 발생합니다.


## 결론
