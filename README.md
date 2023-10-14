# 类型体操

## 一、模式匹配做提取

- 模式匹配是通过类型 `extends` 一个模式类型，把需要提取的部分放到通过 `infer` 声明的局部变量里，后面可以从这个局部变量拿到类型做各种后续处理。

### 获取Promise的返回值

```typescript
type GetValueType<P> = P extends Promise<infer Value> ? Value : never;

// type GetValueResult = "raingrain"
type GetValueResult = GetValueType<Promise<'raingrain'>>;
```

### 提取数组中第一个元素的类型

- 用它来匹配一个模式类型，提取第一个元素的类型到通过 `infer` 声明的局部变量里返回。
- 类型参数 `Arr` 通过 `extends` 约束为只能是数组类型，数组元素是 `unkown` 也就是可以是任何值。
- `any` 和 `unknown` 都代表任意类型，但是 `unknown` 只能接收任意类型的值，而 `any` 除了可以接收任意类型的值，也可以赋值给任意类型（除了 `never` ）。类型体操中经常用 `unknown` 接受和匹配任何类型，而很少把任何类型赋值给某个类型变量。
- 对 `Arr` 做模式匹配，把我们要提取的第一个元素的类型放到通过 `infer` 声明的 `First` 局部变量里，后面的元素可以是任何类型，用 `unknown` 接收，然后把局部变量 `First` 返回。

```typescript
type GetFirst<Arr extends unknown[]> = Arr extends [infer First, ...unknown[]] ? First : never;

// type GetFirstResult = 1
type GetFirstResult = GetFirst<[1,2,3]>;

// type GetFirstResult2 = never
type GetFirstResult2 = GetFirst<[]>;
```

### 提取数组中最后一个元素的类型
  
- 如果是空数组，就直接返回，否则匹配剩余的元素，放到 `infer` 声明的局部变量 `Rest` 里，返回 `Rest` 。

```typescript
type GetLast<Arr extends unknown[]> = Arr extends [...unknown[], infer Last] ? Last : never;

// type GetLastResult = 3
type GetLastResult = GetLast<[1,2,3]>;

// type GetLastResult = never
type GetLastResult2 = GetLast<[]>;
```

### 提取移除了最后一个元素的数组

```typescript
type PopArr<Arr extends unknown[]> = Arr extends [] ? [] : (Arr extends [...infer Rest, unknown] ? Rest : never);

// type PopResult = [1, 2]
type PopResult = PopArr<[1,2,3]>;

// type PopResult = []
type PopResult2 = PopArr<[]>;
```

### 提取移除了第一个元素的数组

```typescript
type ShiftArr<Arr extends unknown[]> = Arr extends [] ? [] : (Arr extends [unknown, ...infer Rest] ? Rest : never);

// type ShiftResult = [2, 3]
type ShiftResult = ShiftArr<[1,2,3]>;

// type ShiftResult = []
type ShiftResult2 = ShiftArr<[]>;
```

### 判断字符串是否以某个前缀开头
  
- 需要声明字符串 `Str` 、匹配的前缀 `Prefix` 两个类型参数，它们都是 `string` 。
- 用 `Str` 去匹配一个模式类型，模式类型的前缀是 `Prefix` ，后面是任意的 `string` ，如果匹配返回 `true` ，否则返回 `false` 。

```typescript
type StartsWith<Str extends string, Prefix extends string> = Str extends `${Prefix}${string}` ? true : false;

// type StartsWithResult = true
type StartsWithResult = StartsWith<'guang and dong', 'guang'>;

// type StartsWithResult = false
type StartsWithResult2 = StartsWith<'guang and dong', 'dong'>;
```

### 字符串替换
  
- 声明要替换的字符串 `Str` 、待替换的字符串 `From` 、替换成的字符串 `3` 个类型参数，通过 `extends` 约束为都是 `string` 类型。
- 用 `Str` 去匹配模式串，模式串由 `From` 和之前之后的字符串构成，把之前之后的字符串放到通过 `infer` 声明的局部变量 `Prefix` 、 `Suffix` 里。
- 用 `Prefix` 、 `Suffix` 加上替换到的字符串 `To` 构造成新的字符串类型返回。

```typescript
type ReplaceStr<Str extends string, From extends string, To extends string> =  Str extends `${infer Prefix}${From}${infer Suffix}` ? `${Prefix}${To}${Suffix}` : Str;

// type ReplaceResult = "Guangguang's best friend is Dongdong"
type ReplaceResult = ReplaceStr<"Guangguang's best friend is ?", "?", "Dongdong">;

// type ReplaceResult = "abc"
type ReplaceResult2 = ReplaceStr<"abc", "?", "Dongdong">;
```

### 去除空白字符
  
- 不过因为我们不知道有多少个空白字符，所以只能一个个匹配和去掉，需要递归。先实现 `TrimRight` ：
  - 类型参数 `Str` 是要 `Trim` 的字符串。
  - 如果 `Str` 匹配字符串加空白字符（空格、换行、制表符），那就把字符串放到 `infer` 声明的局部变量 `Rest` 里。
  - 把 `Rest` 作为类型参数递归 `TrimRight` ，直到不匹配，这时的类型参数 `Str` 就是处理结果。
- 同理可得 `TrimLeft` ， `TrimRight` 和 `TrimLeft` 结合就是 `Trim` 。

```typescript
type TrimStrRight<Str extends string> = Str extends `${infer Rest}${' ' | '\n' | '\t'}` ? TrimStrRight<Rest> : Str;

// type TrimRightResult = "guang"
type TrimRightResult = TrimStrRight<'guang        '>;

type TrimStrLeft<Str extends string> = Str extends `${' ' | '\n' | '\t'}${infer Rest}` ? TrimStrLeft<Rest> : Str;

// type TrimRightResult = "dong"
type TrimLeftResult = TrimStrLeft<'      dong'>;

type TrimStr<Str extends string> = TrimStrRight<TrimStrLeft<Str>>;

// type TrimRightResult = "dong"
type TrimResult = TrimStr<'      dong   '>;
```

### 提取函数参数的类型

- 类型参数 `Func` 是要匹配的函数类型，通过 `extends` 约束为 `Function` 。`Func` 和模式类型做匹配，参数类型放到用 `infer` 声明的局部变量 `Args` 里，返回值可以是任何类型，用 `unknown` 。返回提取到的参数类型 `Args`。

```typescript
type GetParameters<Func extends Function> = Func extends (...args: infer Args) => unknown ? Args : never;

// type ParametersResult = [name: string, age: number]
type ParametersResult = GetParameters<(name: string, age: number) => string>;

// type ParametersResult2 = []
type ParametersResult2 = GetParameters<() => string>;
```

### 提取函数返回值的类型

- `Func` 和模式类型做匹配，提取返回值到通过 `infer` 声明的局部变量 `ReturnType` 里返回。参数类型可以是任意类型，也就是 `any[]` ，但不能是 `unknown[]` 。

```typescript
type GetReturnType<Func extends Function> = Func extends (...args: any[]) => infer ReturnType ? ReturnType : never;

// type ReturnTypeResullt = "dong"
type ReturnTypeResullt = GetReturnType<(name: string) => 'dong'>;
```

### 提取this的类型

- 类型参数 `T` 是待处理的类型。
- 用 `T` 匹配一个模式类型，提取 `this` 的类型到 `infer` 声明的局部变量 `ThisType` 中，其余的参数是任意类型，也就是 `any` ，返回值也是任意类型。
- 返回提取到的 `ThisType` 。

```typescript
class Dong {
    name: string;

    constructor() {
        this.name = "dong";
    }

    hello(this: Dong) {
        return 'hello, I\'m ' + this.name;
    }
}

const dong = new Dong();
dong.hello();

dong.hello.call({xxx: 1});

type GetThisParameterType<T> = T extends (this: infer ThisType, ...args: any[]) => any ? ThisType : unknown;

// type GetThisParameterTypeRes = Dong
type GetThisParameterTypeRes = GetThisParameterType<typeof dong.hello>;
```

### 提取构造器对应的实例类型

- 类型参数 `ConstructorType` 是待处理的类型，通过 `extends` 约束为构造器类型。
- 用 `ConstructorType` 匹配一个模式类型，提取返回的实例类型到 `infer` 声明的局部变量 `InstanceType` 里，返回 `InstanceType` 。

```typescript
type GetInstanceType<ConstructorType extends new (...args: any) => any> = ConstructorType extends new (...args: any) => infer InstanceType ? InstanceType : any;

interface Person {
    name: string;
}

interface PersonConstructor {
    new(name: string): Person;
}

// type GetInstanceTypeRes = Person
type GetInstanceTypeRes = GetInstanceType<PersonConstructor>;
```

### 提取构造器返回值类型

- 类型参数 `ConstructorType` 为待处理的类型，通过 `extends` 约束为构造器类型。
- 用 `ConstructorType` 匹配一个模式类型，提取参数的部分到 `infer` 声明的局部变量 `ParametersType` 里，返回 `ParametersType` 。

```typescript
type GetConstructorParameters<
    ConstructorType extends new (...args: any) => any
> = ConstructorType extends new (...args: infer ParametersType) => any
    ? ParametersType
    : never;

interface Person {
    name: string;
}

interface PersonConstructor {
    new(name: string): Person;
}

// type GetConstructorParametersRes = [name: string]
type GetConstructorParametersRes = GetConstructorParameters<PersonConstructor>;
```

### 提取Props里ref的类型

- 类型参数 `Props` 为待处理的类型。通过 `keyof Props` 取出 `Props` 的所有索引构成的联合类型，判断下 `ref` 是否在其中，也就是 `'ref' extends keyof Props` 。为什么要做这个判断，因为在 `ts3.0` 里面如果没有对应的索引， `Obj[Key]` 返回的是 `{}` 而不是 `never` ，所以这样做下兼容处理。如果有 `ref` 这个索引的话，就通过 `infer` 提取 `Value` 的类型返回，否则返回 `never` 。

```typescript
type GetRefProps<Props> = 
    'ref' extends keyof Props
        ? Props extends { ref?: infer Value | undefined}
            ? Value
            : never
        : never;

// type GetRefPropsRes = 1
type GetRefPropsRes = GetRefProps<{ ref?: 1, name: 'dong'}>;

// type GetRefPropsRes2 = undefined
type GetRefPropsRes2 = GetRefProps<{ ref?: undefined, name: 'dong'}>;
```



```typescript```



```typescript```