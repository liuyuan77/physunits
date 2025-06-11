# physunits · 物理单位库

**Type-safe physical quantities with dimensional analysis**  
**带量纲分析的类型安全物理量库**  

[![Crates.io](https://img.shields.io/crates/v/physunits)](https://crates.io/crates/physunits)
[![Docs.rs](https://img.shields.io/docsrs/physunits)](https://docs.rs/physunits)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

A Rust library for safe unit operations /  
Rust实现的类型安全单位计算库

## Key Advantages / 核心优势

1. **No dependencies** - Pure Rust implementation without external dependencies  
   **无依赖库** - 纯Rust实现，不依赖任何外部库

2. **Typenum-like constant calculation** - Full compile-time dimensional analysis with constant evaluation capabilities  
   **类Typenum的常量计算** - 完整的编译期量纲分析与常量计算能力

3. **Var structure bridge** - Seamless integration between variable and constant calculations  
   **Var结构桥接** - 完美衔接变量与常量的混合计算

4. **Complete operator overloading** - Supports all arithmetic operations with dimensional correctness  
   **完整运算符重载** - 支持所有算术运算并保持量纲正确性

## Core Architecture / 核心架构

### 1. Constant System / 常量系统

```rust
// 基础常量类型
pub struct B0<H>(PhantomData<H>);  // 二进制0节点
pub struct B1<H>(PhantomData<H>);  // 二进制1节点
pub struct Z0;                     // 零值
pub struct P1;                     // +1
pub struct N1;                     // -1

// 常量运算特性
pub trait Integer: Default + Sealed + Copy {
    fn to_i32() -> i32;  // 常量值转换
}

// 示例实现
impl Integer for P1 {
    fn to_i32() -> i32 { 1 }
}

### 2. Var Structure / 变量结构
```rust
/// 变量结构体，桥接常量与变量计算
#[derive(Debug, Clone, Copy)]
pub struct Var<T: Numeric>(pub T);

// 支持的运算类型
impl<T: Numeric> Add for Var<T> {
    type Output = Self;
    fn add(self, b: Self) -> Self {
        Var(self.0 + b.0)
    }
}

// 与常量的混合运算
impl<T: Numeric, C: Integer> Add<C> for Var<T> {
    type Output = C::Output;
    fn add(self, c: C) -> Self::Output {
        c + self  // 调用常量的加法实现
    }
}
```

### 3. Dimension System / 量纲系统

```rust
/// 国际单位制7大量纲
pub struct Dimension<
    METER: Integer,     // 长度
    KILOGRAM: Integer,  // 质量
    SECOND: Integer,    // 时间
    AMPERE: Integer,    // 电流
    KELVIN: Integer,    // 温度
    MOLE: Integer,      // 物质的量
    CANDELA: Integer    // 发光强度
>(PhantomData<(METER, KILOGRAM, SECOND, AMPERE, KELVIN, MOLE, CANDELA)>);

// 量纲运算示例
impl<M1, M2, KG1, KG2> Mul<Dimension<M2, KG2>> for Dimension<M1, KG1> {
    type Output = Dimension<Sum<M1, M2>, Sum<KG1, KG2>>;
    fn mul(self, _: Dimension<M2, KG2>) -> Self::Output {
        Dimension::new()  // 量纲指数相加
    }
}
```

### 4. Unit System / 单位系统

```rust
/// SI基础单位结构
pub struct Si<Value: Scalar, D: Dimensional, Pr: Prefixed>(
    pub Value,
    PhantomData<(D, Pr)>
);

/// 复合单位结构
pub struct Unit<S: Sied, R: Scaled>(pub S, PhantomData<R>);

// 单位运算示例
impl<V, D1, D2> Mul<Si<V, D2>> for Si<V, D1> {
    type Output = Si<V, Prod<D1, D2>>;
    fn mul(self, rhs: Si<V, D2>) -> Self::Output {
        Si(self.0 * rhs.0, PhantomData)
    }
}
```

## Features / 特性

| Feature | 功能描述 |
|---------|----------|
| 📏 Compile-time dimensional safety | 编译期量纲安全 |
| ⚡ Zero runtime overhead | 零运行时开销 |
| 🔢 Integer & float support | 支持整数和浮点数 |
| 🔄 Automatic unit conversion | 自动单位转换 |
| 🏷️ Runtime prefix handling | 运行时词头处理 |
| 🧮 Typenum-like constant math | 类Typenum的常量计算 |
| 🌉 Var-based mixed calculation | 基于Var的混合计算 |
| 🔧 Full operator overloading | 完整运算符重载 |

## Usage Examples / 使用示例

### Basic Operations / 基础运算

```rust
use physunits::{m, kg, s, N};

// 常量计算
let force = const（5） * kg * const(9) * m / (s * s);
println!("Force: {} N", force.value());

// 变量计算
let mass = Var(5.0) * N;
let acceleration = Var(9.0);
let force = mass * acceleration;
```

### Mixed Calculation / 混合计算

```rust
use physunits::{consts::*, Var};

// 编译时常量与运行时变量混合运算
let G = Const(6) * m3 / (kg * s * s);
let m1 = Var(5.972e24);  // 地球质量 (kg)
let m2 = Var(7.342e22);  // 月球质量 (kg)
let r = Var(3.844e8);    // 地月距离 (m)

let f = G * m1 * m2 / (r * r);
println!("Gravitational force: {} N", f.value());
```

### Temperature Conversion / 温度转换

```rust
use physunits::{Celsius, Fahrenheit};

let boiling = quantity::Si::<f64, Celsius>::new(100.0);
let fahr = boiling.convert::<Fahrenheit>();
println!("Water boils at {} °F", fahr.value()); 
```

### Unit Math / 单位运算

```rust
let speed = 60.0 * km / h;
let time = 30.0 * min;
let distance = speed * time;  // 自动推导为km单位
```

## Advanced Features / 高级特性

1. 常量计算系统
+ 二进制编码的常量类型 (B0, B1)

+ 基础常量值 (Z0, P1, N1)

+ 支持所有算术运算的trait实现

+ 常量到运行时的值转换 (to_i32())

2. Var结构桥接

+ 同时支持变量与常量的运算

+ 自动类型转换系统

+ 完整的运算符重载 (+, -, *, /, +=, -=, *=, /=)

+ 支持i64和f64基础类型

3. 量纲系统

+ 7大基本量纲的编译期检查

+ 量纲运算自动推导

+ 支持幂运算 (pow()方法)

+ 零开销抽象

## Comparison / 对比

| Feature         | PhysUnits | uom   | nalgebra |
|----------------|----------|-------|----------|
|Dim Safety	    |✅	| ✅ | ❌ |
|Integer Support|✅	| ⚠️ | ❌ |
|Runtime Prefix	|✅	| ❌ | ❌ |
|No Deps	    |✅	| ❌ | ❌ |
|Const Math	    |✅	| ⚠️ | ❌ |
|Var Bridge	    |✅	| ❌ | ❌ |


| 特性          | PhysUnits | uom   | nalgebra |
|--------------|----------|-------|----------|
| 量纲安全	 | ✅ | ✅ | ❌ |
| 整数支持	 | ✅ | ⚠️ | ❌ |
| 运行时词头 | ✅ | ❌ | ❌ |
| 无依赖	 | ✅ | ❌ | ❌ |
| 常量计算	 | ✅ | ⚠️ | ❌ |
| 变量桥接	 | ✅ | ❌ | ❌ |

## Installation / 安装

```toml
[dependencies]
physunits = "0.0.4"
```

## Contributing / 贡献指南

We welcome issues and PRs! / 欢迎提交 Issue 和 PR！

Key needs: / 重点需求：

- More unit definitions / 更多单位定义

- Real-world physics test cases / 实际物理测试案例

- Better error messages / 更好的错误提示

- Constant calculation optimization / 常量计算优化

- WASM compatibility / WASM兼容性
