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

## 二、重新构造做变换

- TypeScript支持 `type` 、 `infer` 、类型参数来保存任意类型，相当于变量的作用。但其实也不能叫变量，因为它们是不可变的。想要变化就需要重新构造新的类型，并且可以在构造新类型的过程中对原类型做一些过滤和变换。数组、字符串、函数、索引类型等都可以用这种方式对原类型做变换产生新的类型。其中索引类型有专门的语法叫做映射类型，对索引做修改的 `as` 叫做重映射。

### 给元组或数组类型末尾添加一个新的类型

- 类型参数 `Arr` 是要修改的数组或元组类型，元素的类型任意，也就是  `unknown` 。
- 类型参数 `Ele` 是添加的元素的类型。
- 返回的是用 `Arr` 已有的元素加上 `Ele` 构造的新的元组类型。

```typescript
type Push<Arr extends  unknown[], Ele> = [...Arr, Ele];

// type PushResult = [1, 2, 3, 4]
type PushResult = Push<[1, 2, 3], 4>;
```

### 给元组或数组类型开头添加一个新的类型

```typescript
type Unshift<Arr extends  unknown[], Ele> = [Ele, ...Arr];

// type UnshiftResult = [0, 1, 2, 3]
type UnshiftResult = Unshift<[1, 2, 3], 0>;
```

### 双元素元组合并

- 两个类型参数 `One` 、 `Other` 是两个元组，类型是 `[unknown, unknown]` ，代表 `2` 个任意类型的元素构成的元组。
- 通过 `infer` 分别提取 `One` 和 `Other` 的元素到 `infer` 声明的局部变量 `OneFirst` 、`OneSecond` 、 `OtherFirst` 、 `OtherSecond` 里。用提取的元素构造成新的元组返回即可。

```typescript
type Zip<One extends [unknown, unknown], Other extends [unknown, unknown]> = 
    One extends [infer OneFirst, infer OneSecond]
        ? (Other extends [infer OtherFirst, infer OtherSecond]
            ? [[OneFirst, OtherFirst], [OneSecond, OtherSecond]] : [])
                : [];

// type ZipResult = [[1, "guang"], [2, "dong"]]                
type ZipResult = Zip<[1,2], ['guang', 'dong']>;
```

### 任意元素元组合并

- 类型参数 `One` 、 `Other` 声明为 `unknown[]` ，也就是元素个数任意，类型任意的数组。
- 每次提取 `One` 和 `Other` 的第一个元素 `OneFirst` 、 `OtherFirst` ，剩余的放到 `OneRest` 、 `OtherRest` 里。
- 用 `OneFirst` 、 `OtherFirst` 构造成新的元组的一个元素，剩余元素继续递归处理 `OneRest` 、 `OtherRest` 。

```typescript
type Zip2<One extends unknown[], Other extends unknown[]> = 
    One extends [infer OneFirst, ...infer OneRest]
        ? (Other extends [infer OtherFirst, ...infer OtherRest]
            ? [[OneFirst, OtherFirst], ...Zip2<OneRest, OtherRest>] : [])
                : [];

// type Zip2Result = [[1, "guang"], [2, "dong"], [3, "is"], [4, "best"], [5, "friend"]]
type Zip2Result = Zip2<[1,2,3,4,5], ['guang', 'dong', 'is', 'best', 'friend']>;
```

### 字符串字面量类型首字母大写

- 我们声明了类型参数 `Str` 是要处理的字符串类型，通过 `extends` 约束为 `string` 。
- 通过 `infer` 提取出首个字符到局部变量 `First` ，提取后面的字符到局部变量 `Rest` 。
- 然后使用 `TypeScript` 提供的内置高级类型 `Uppercase` 把首字母转为大写，加上 `Rest` ，构造成新的字符串类型返回。

```typescript
type CapitalizeStr<Str extends string> = Str extends `${infer First}${infer Rest}` ? `${Uppercase<First>}${Rest}` : Str;

// type CapitalizeResult = "Guang"
type CapitalizeResult = CapitalizeStr<'guang'>;
```

### 字符串字面量类型下划线转驼峰

- 类型参数 `Str` 是待处理的字符串类型，约束为 `string` 。
- 提取 `_` 之前和之后的两个字符到 `infer` 声明的局部变量 `Left` 和 `Right` ，剩下的字符放到 `Rest` 里。
- 然后把右边的字符 `Right` 大写，和 `Left` 构造成新的字符串，剩余的字符 `Rest` 要继续递归的处理。

```typescript
type CamelCase<Str extends string> = 
    Str extends `${infer Left}_${infer Right}${infer Rest}`
        ? `${Left}${Uppercase<Right>}${CamelCase<Rest>}`
        : Str;

// type CamelCaseResult = "dongDongDong"
type CamelCaseResult = CamelCase<'dong_dong_dong'>;
```

### 删除字符串字面量类型中的某个子串

- 类型参数 `Str` 是待处理的字符串， `SubStr` 是要删除的字符串，都通过 `extends` 约束为 `string` 类型。
- 通过模式匹配提取 `SubStr` 之前和之后的字符串到 `infer` 声明的局部变量 `Prefix`、 `Suffix` 中。
- 如果不匹配就直接返回 `Str` 。
- 如果匹配，那就用 `Prefix` 、 `Suffix` 构造成新的字符串，然后继续递归删除 `SubStr` 。直到不再匹配，也就是没有 `SubStr` 了。

```typescript
type DropSubStr<Str extends string, SubStr extends string> = 
    Str extends `${infer Prefix}${SubStr}${infer Suffix}` 
        ? DropSubStr<`${Prefix}${Suffix}`, SubStr> : Str;

// type DropResult = "dong"
type DropResult = DropSubStr<'dong~~~', '~'>;
```

### 在已有的函数类型上添加一个参数

- 类型参数 `Func` 是待处理的函数类型，通过 `extends` 约束为 `Function` ， `Arg`` 是要添加的参数类型。
- 通过模式匹配提取参数到 `infer` 声明的局部变量 `Args` 中，提取返回值到局部变量 `ReturnType` 中。
- 用 `Args` 数组添加 `Arg` 构造成新的参数类型，结合 `ReturnType` 构造成新的函数类型返回。

```typescript
type AppendArgument<Func extends Function, Arg> = 
    Func extends (...args: infer Args) => infer ReturnType 
        ? (...args: [...Args, Arg]) => ReturnType : never;

// type AppendArgumentResult = (name: string, args_1: number) => boolean
type AppendArgumentResult  = AppendArgument<(name: string) => boolean, number>;
```

### 单元素映射成三元素元组

- 类型参数 `Obj` 是待处理的索引类型，通过 `extends` 约束为 `object` 。
- 用 `keyof` 取出 `Obj` 的索引，作为新的索引类型的索引，也就是 `Key in keyof Obj` 。
- 值的类型可以做变换，这里我们用之前索引类型的值 `Obj[Key]` 构造成了三个元素的元组类型 `[Obj[Key], Obj[Key], Obj[Key]]` 。

```typescript
type Mapping<Obj extends object> = { 
    [Key in keyof Obj]: [Obj[Key], Obj[Key], Obj[Key]]
}

// type res = {
//     a: [1, 1, 1];
//     b: [2, 2, 2];
// }
type res = Mapping<{ a: 1, b: 2}>;
```

### 把索引类型的Key变为大写

- 除了可以对 `Value` 做修改，也可以对 `Key` 做修改，使用 `as` ，这叫做重映射。
- 类型参数 `Obj` 是待处理的索引类型，通过 `extends` 约束为 `object` 。
- 新的索引类型的索引为 `Obj` 中的索引，也就是 `Key in keyof Obj` ，但要做一些变换，也就是 `as` 之后的。
- 通过 `Uppercase` 把索引 `Key` 转为大写，因为索引可能为 `string, number, symbol` 类型，而这里只能接受 `string` 类型，所以要 `& string` ，也就是取索引中 `string` 的部分。
- `value` 保持不变，也就是之前的索引 `Key` 对应的值的类型 `Obj[Key]` 。

```typescript
type UppercaseKey<Obj extends object> = { 
    [Key in keyof Obj as Uppercase<Key & string>]: Obj[Key]
}

// type UppercaseKeyResult = {
//     GUANG: 1;
//     DONG: 2;
// }
type UppercaseKeyResult = UppercaseKey<{ guang: 1, dong: 2}>;
```

### 给索引类型添加只读修饰

- 通过映射类型构造了新的索引类型，给索引加上了 `readonly` 的修饰，其余的保持不变，索引依然为原来的索引 `Key in keyof T` ，值依然为原来的值 `T[Key]` 。

```typescript
type ToReadonly<T> =  {
    readonly [Key in keyof T]: T[Key];
}

// type ReadonlyResult = {
//     readonly name: string;
//     readonly age: number;
// }
type ReadonlyResult = ToReadonly<{
    name: string;
    age: number;
}>;
```

### 给索引类型添加可选修饰符

```typescript
type ToPartial<T> = {
    [Key in keyof T]?: T[Key]
}

// type PartialResult = {
//     name?: string | undefined;
//     age?: number | undefined;
// }
type PartialResult = ToPartial<{
    name: string;
    age: number;
}>;
```

### 给索引类型的索引去掉只读修饰符

```typescript
type ToMutable<T> = {
    -readonly [Key in keyof T]: T[Key]
}

// type MutableResult = {
//     name: string;
//     age: number;
// }
type MutableResult =  ToMutable<{
    readonly name: string;
    age: number;
}>;
```

### 给索引类型的索引去掉可选的修饰

```typescript
type ToRequired<T> = {
    [Key in keyof T]-?: T[Key]
}

// type RequiredResullt = {
//     name: string;
//     age: number;
// }
type RequiredResullt = ToRequired<{
    name?: string;
    age: number;
}>;
```

### 构造新索引类型的时候根据值的类型做过滤

- 类型参数 `Obj` 为要处理的索引类型，通过 `extends` 约束为索引为 `string` ，值为任意类型的索引类型 `Record<string, any>` 。
- 类型参数 `ValueType` 为要过滤出的值的类型。
- 构造新的索引类型，索引为 `Obj` 的索引，也就是 `Key in keyof Obj` ，但要做一些变换，也就是 `as` 之后的部分。
- 如果原来索引的值 `Obj[Key]` 是 `ValueType` 类型，索引依然为之前的索引 `Key` ，否则索引设置为 `never` ， `never`` 的索引会在生成新的索引类型时被去掉。
- 值保持不变，依然为原来索引的值，也就是 `Obj[Key]` 。

```typescript
type FilterByValueType<Obj extends Record<string, any>, ValueType> = {
    [Key in keyof Obj 
        as (Obj[Key] extends ValueType ? Key : never)]
        : Obj[Key]
}

interface Person {
    name: string;
    age: number;
    hobby: string[];
}

// type FilterResult = {
//     name: string;
//     age: number;
// }
type FilterResult = FilterByValueType<Person, string | number>;
```

## 三、递归复用做循环

- TypeScript类型系统不支持循环，但支持递归。当处理数量（个数、长度、层数）不固定的类型的时候，可以只处理一个类型，然后递归的调用自身处理下一个类型，直到结束条件也就是所有的类型都处理完了，就完成了不确定数量的类型编程，达到循环的效果。
- 在类型体操中，遇到数量不确定的问题，要条件反射的想到递归。比如数组长度不确定、字符串长度不确定、索引类型层数不确定等。

### 提取不确定层数的Promise中的value类型

- 类型参数 `P` 是待处理的 `Promise` ，通过 `extends` 约束为 `Promise` 类型， `value` 类型不确定，设为 `unknown` 。
- 每次只处理一个类型的提取，也就是通过模式匹配提取出 `value` 的类型到 `infer` 声明的局部变量 `ValueType` 中。
- 然后判断如果 `ValueType` 依然是 `Promise` 类型，就递归处理。
- 结束条件就是 `ValueType` 不为 `Promise` 类型，那就处理完了所有的层数，返回这时的 `ValueType` 。
- 其实这个类型的实现可以进一步的简化：不再约束类型参数必须是 `Promise` ，这样就可以少一层判断。

```typescript
type DeepPromiseValueType<P extends Promise<unknown>> =
    P extends Promise<infer ValueType> 
        ? (ValueType extends Promise<unknown>
            ? DeepPromiseValueType<ValueType>
            : ValueType)
        : never;

// type DeepPromiseResult = {
//     [x: string]: any;
// }
type DeepPromiseResult = DeepPromiseValueType<Promise<Promise<Record<string, any>>>>;

type DeepPromiseValueType2<T> = 
    T extends Promise<infer ValueType> 
        ? DeepPromiseValueType2<ValueType>
        : T;

// type DeepPromiseValueType2Res = number
type DeepPromiseValueType2Res = DeepPromiseValueType2<Promise<Promise<Promise<number>>>>;
```

### 反转数组类型中的元素

- 类型参数 `Arr` 为待处理的数组类型，元素类型不确定，也就是 `unknown` 。
- 每次只处理一个元素的提取，放到 `infer` 声明的局部变量 `First` 里，剩下的放到 `Rest` 里。
- 用 `First` 作为最后一个元素构造新数组，其余元素递归的取。
- 结束条件就是取完所有的元素，也就是不再满足模式匹配的条件，这时候就返回 `Arr` 。

```typescript
type ReverseArr<Arr extends unknown[]> = 
    Arr extends [infer First, ...infer Rest] 
        ? [...ReverseArr<Rest>, First] 
        : Arr;

// type ReverseArrResult = [5, 4, 3, 2, 1]
type ReverseArrResult = ReverseArr<[1,2,3,4,5]>;
```

### 查找数组中是否存在某个元素

- 类型参数 `Arr` 是待查找的数组类型，元素类型任意，也就是 `unknown` 。 `FindItem` 待查找的元素类型。
- 每次提取一个元素到 `infer` 声明的局部变量 `First` 中，剩余的放到局部变量 `Rest` 。
- 判断 `First` 是否是要查找的元素，也就是和 `FindItem` 相等，是的话就返回 `true` ，否则继续递归判断下一个元素。
- 直到结束条件也就是提取不出下一个元素，这时返回 `false` 。
- 相等的判断就是 `A` 是 `B` 的子类型并且 `B` 也是 `A` 的子类型。

```typescript
type Includes<Arr extends unknown[], FindItem> = 
    Arr extends [infer First, ...infer Rest]
        ? (IsEqual<First, FindItem> extends true
            ? true
            : Includes<Rest, FindItem>)
        : false;

type IsEqual<A, B> = (A extends B ? true : false) & (B extends A ? true : false);

// type IncludesResult = true
type IncludesResult = Includes<[1, 2, 3, 4, 5], 4>;

// type IncludesResult2 = false
type IncludesResult2 = Includes<[1, 2, 3, 4, 5], 6>;
```

### 在数组中删除指定元素

- 类型参数 `Arr` 是待处理的数组，元素类型任意，也就是 `unknown[]` 。类型参数 `Item` 为待查找的元素类型。类型参数 `Result` 是构造出的新数组，默认值是 `[]` 。
- 通过模式匹配提取数组中的一个元素的类型，如果是 `Item` 类型的话就删除，也就是不放入构造的新数组，直接返回之前的 `Result` 。
- 否则放入构造的新数组，也就是再构造一个新的数组 `[...Result, First]` 。
- 直到模式匹配不再满足，也就是处理完了所有的元素，返回这时候的 `Result` 。

```typescript
type RemoveItem<Arr extends unknown[], Item, Result extends unknown[] = []> = 
    Arr extends [infer First, ...infer Rest]
        ? (IsEqual<First, Item> extends true
            ? RemoveItem<Rest, Item, Result>
            : RemoveItem<Rest, Item, [...Result, First]>)
        : Result;

// type RemoveItemResult = [1, 3]
type RemoveItemResult = RemoveItem<[1,2,2,3], 2>;
```

### 根据长度和元素类型构造指定数组

- 类型参数 `Length` 为数组长度，约束为 `number` 。类型参数 `Ele` 为元素类型，默认值为 `unknown` 。类型参数 `Arr` 为构造出的数组，默认值是 `[]` 。
- 每次判断下 `Arr` 的长度是否到了 `Length` ，是的话就返回 `Arr` ，否则在 `Arr` 上加一个元素，然后递归构造。

```typescript
type BuildArray<Length extends number, 
    Ele = unknown, 
    Arr extends unknown[] = []> =
    Arr['length'] extends Length 
        ? Arr 
        : BuildArray<Length, Ele, [...Arr, Ele]>;

// type BuildArrResult = [unknown, unknown, unknown, unknown, unknown]
type BuildArrResult = BuildArray<5>;
```

### 把字符串字面量类型中的某个字符全部替换为另外一个字符

- 类型参数 `Str` 是待处理的字符串类型， `From` 是待替换的字符， `To` 是替换到的字符。
- 通过模式匹配提取 `From` 左右的字符串到 `infer` 声明的局部变量 `Left` 和 `Right` 里。
- 用 `Left` 和 `To` 构造新的字符串，剩余的 `Right` 部分继续递归的替换。
- 结束条件是不再满足模式匹配，也就是没有要替换的元素，这时就直接返回字符串 `Str` 。

```typescript
type ReplaceAll<Str extends string, 
    From extends string, 
    To extends string> = 
        Str extends `${infer Left}${From}${infer Right}`
            ? `${Left}${To}${ReplaceAll<Right, From, To>}`
            : Str;

// type ReplaceAllResult = "dong dong dong"
type ReplaceAllResult = ReplaceAll<'guang guang guang', 'guang', 'dong'>;
```

### 把字符串字面量类型的每个字符都提取出来组成联合类型

- 类型参数 `Str` 为待处理的字符串类型，通过 `extends` 约束为 `string` 。
- 通过模式匹配提取第一个字符到 `infer` 声明的局部变量 `First` ，其余的字符放到局部变量 `Rest` 。
- 用 `First` 构造联合类型，剩余的元素递归的取。

```typescript
type StringToUnion<Str extends string> = 
    Str extends `${infer First}${infer Rest}`
        ? First | StringToUnion<Rest>
        : never;

// type StringToUnionResult = "h" | "e" | "l" | "o"
type StringToUnionResult = StringToUnion<'hello'>;
```

### 字符串类型的反转

- 类型参数 `Str` 为待处理的字符串。类型参数 `Result` 为构造出的字符，默认值是空串。
- 通过模式匹配提取第一个字符到 `infer` 声明的局部变量 `First` ，其余字符放到 `Rest` 。
- 用 `First` 和之前的 `Result` 构造成新的字符串，把 `First` 放到前面，因为递归是从左到右处理，那么不断往前插就是把右边的放到了左边，完成了反转的效果。
- 直到模式匹配不满足，就处理完了所有的字符。

```typescript
type ReverseStr<Str extends string, 
    Result extends string = ''> = 
    Str extends `${infer First}${infer Rest}` 
        ? ReverseStr<Rest, `${First}${Result}`> 
        : Result;

// type ReverseStrResult = "olleh"
type ReverseStrResult = ReverseStr<'hello'>;
```

### 给嵌套对象添加只读修饰符

- 类型参数 `Obj` 是待处理的索引类型，约束为 `Record<string, any>` ，也就是索引为 `string` ，值为任意类型的索引类型。
- 索引映射自之前的索引，也就是 `Key in keyof Obj` ，只不过加上了 `readonly` 的修饰。
- 值要做下判断，如果是 `object` 类型并且还是 `Function` ，那么就直接取之前的值 `Obj[Key]` 。
- 如果是 `object` 类型但不是 `Function` ，那就是说也是一个索引类型，就递归处理 `DeepReadonly<Obj[Key]>` 。
- 否则，值不是 `object` 就直接返回之前的值 `Obj[Key]` 。

```typescript
type DeepReadonly<Obj extends Record<string, any>> =
    Obj extends any
        ? {
            readonly [Key in keyof Obj]:
                Obj[Key] extends object
                    ? (Obj[Key] extends Function
                        ? Obj[Key] 
                        : DeepReadonly<Obj[Key]>)
                    : Obj[Key]
        }
        : never;

type obj = {
    a: {
        b: {
            c: {
                f: () => 'dong',
                d: {
                    e: {
                        guang: string
                    }
                }
            }
        }
    }
}

// type DeepReadonlyResult = {
//     readonly a: {
//         readonly b: {
//             readonly c: {
//                 readonly f: () => 'dong';
//                 readonly d: {
//                     readonly e: {
//                         readonly guang: string;
//                     };
//                 };
//             };
//         };
//     };
// }
type DeepReadonlyResult = DeepReadonly<obj>;
```

```typescript

```


```typescript

```


```typescript

```
