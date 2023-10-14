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

- `Func` 和模式类型做匹配，提取返回值到通过 `infer` 声明的局部变量 `ReturnType` 里返回。参数类型可以是任意类型，也就是 `any[]` ，但不能是 `unknown[]` 。这就是因为函数参数是逆变的，如果是 `unknown[]` ，那当 `Func` 是这个函数的子类型，它的参数得是 `unknown` 的父类型，这显然是不可能的，所以这里只能用 `any` 。

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

## 四、数组长度做计数

- TypeScript类型系统没有加减乘除运算符，所以我们通过数组类型的构造和提取，然后取长度的方式来实现数值运算。我们通过构造和提取数组类型实现了加减乘除，也实现了各种计数逻辑。

### 加

- 类型参数 `Length` 是要构造的数组的长度。类型参数 `Ele` 是数组元素，默认为 `unknown` 。类型参数 `Arr` 为构造出的数组，默认是 `[]` 。
- 如果 `Arr` 的长度到达了 `Length` ，就返回构造出的 `Arr` ，否则继续递归构造。

```typescript
type BuildArray<
    Length extends number, 
    Ele = unknown, 
    Arr extends unknown[] = []
> = Arr['length'] extends Length 
        ? Arr 
        : BuildArray<Length, Ele, [...Arr, Ele]>;

type Add<Num1 extends number, Num2 extends number> = 
    [...BuildArray<Num1>,...BuildArray<Num2>]['length'];

// type AddResult = 57
type AddResult = Add<32, 25>;
```

### 减

- 类型参数 `Num1` 、 `Num2` 分别是被减数和减数，通过 `extends` 约束为 `number` 。
- 构造 `Num1` 长度的数组，通过模式匹配提取出 `Num2` 长度个元素，剩下的放到 `infer` 声明的局部变量 `Rest` 里。
- 取 `Rest` 的长度返回，就是减法的结果。

```typescript
type BuildArray<
    Length extends number, 
    Ele = unknown, 
    Arr extends unknown[] = []
> = Arr['length'] extends Length 
        ? Arr 
        : BuildArray<Length, Ele, [...Arr, Ele]>;

type Subtract<Num1 extends number, Num2 extends number> = 
    BuildArray<Num1> extends [...arr1: BuildArray<Num2>, ...arr2: infer Rest]
        ? Rest['length']
        : never;

// type SubtractResult = 21
type SubtractResult = Subtract<33, 12>;
```

### 乘

- 类型参数 `Num1` 和 `Num2` 分别是被加数和加数。
- 因为乘法是多个加法结果的累加，我们加了一个类型参数 `ResultArr` 来保存中间结果，默认值是 `[]` ，相当于从 `0` 开始加。
- 每加一次就把 `Num2` 减一，直到 `Num2` 为 `0` ，就代表加完了。
- 加的过程就是往 `ResultArr` 数组中放 `Num1` 个元素。
- 这样递归的进行累加，也就是递归的往 `ResultArr` 中放元素。
- 最后取 `ResultArr` 的 `length` 就是乘法的结果。

```typescript
type BuildArray<
    Length extends number, 
    Ele = unknown, 
    Arr extends unknown[] = []
> = Arr['length'] extends Length 
        ? Arr 
        : BuildArray<Length, Ele, [...Arr, Ele]>;

// Subtract
type Subtract<Num1 extends number, Num2 extends number> = 
    BuildArray<Num1> extends [...arr1: BuildArray<Num2>, ...arr2: infer Rest]
        ? Rest['length']
        : never;

// Multiply
type Mutiply<Num1 extends number, Num2 extends number, Result extends unknown[] = []> =
    Num2 extends 0 ? Result['length']
        : Mutiply<Num1, Subtract<Num2, 1>, [...BuildArray<Num1>, ...Result]>;

// type MutiplyResult = 666
type MutiplyResult = Mutiply<3, 222>;
```

### 除

- 乘法是递归的累加，那除法不就是递归的累减么？
- 类型参数 `Num1` 和 `Num2` 分别是被减数和减数。
- 类型参数 `CountArr` 是用来记录减了几次的累加数组。
- 如果 `Num1` 减到了 `0` ，那么这时候减了几次就是除法结果，也就是 `CountArr['length']` 。
- 否则继续递归的减，让 `Num1` 减去 `Num2` ，并且 `CountArr` 多加一个元素代表又减了一次。

```typescript
type BuildArray<
    Length extends number, 
    Ele = unknown, 
    Arr extends unknown[] = []
> = Arr['length'] extends Length 
        ? Arr 
        : BuildArray<Length, Ele, [...Arr, Ele]>;

// Subtract
type Subtract<Num1 extends number, Num2 extends number> = 
    BuildArray<Num1> extends [...arr1: BuildArray<Num2>, ...arr2: infer Rest]
        ? Rest['length']
        : never;

// Divide
type Divide<Num1 extends number, Num2 extends number, CountArr extends unknown[] = []> =
    Num1 extends 0 ? CountArr['length']
        : Divide<Subtract<Num1, Num2>, Num2, [unknown, ...CountArr]>;

// type DivideResult = 6
type DivideResult = Divide<30, 5>;
```

### 求字符串长度

- 类型参数 `Str` 是待处理的字符串。类型参数 `CountArr` 是做计数的数组，默认值 `[]` 代表从 `0` 开始。
- 每次通过模式匹配提取去掉一个字符之后的剩余字符串，并且往计数数组里多放入一个元素。递归进行取字符和计数。
- 如果模式匹配不满足，代表计数结束，返回计数数组的长度 `CountArr['length']` 。

```typescript
type StrLen<
    Str extends string,
    CountArr extends unknown[] = []
> = Str extends `${string}${infer Rest}` ? StrLen<Rest, [...CountArr, unknown]> : CountArr['length']

// type StrLenResult = 11
type StrLenResult = StrLen<'Hello World'>;
```

### 数值比较

- 类型参数 `Num1` 和 `Num2` 是待比较的两个数。
- 类型参数 `CountArr` 是计数用的，会不断累加，默认值是 `[]` 代表从 `0` 开始。
- 如果 `Num1 extends Num2` 成立，代表相等，直接返回 `false` 。
- 否则判断计数数组的长度，如果先到了 `Num2` ，那么就是 `Num1` 大，返回 `true` 。
- 反之，如果先到了 `Num1` ，那么就是 `Num2` 大，返回 `false` 。
- 如果都没到就往计数数组 `CountArr` 中放入一个元素，继续递归。

```typescript
type GreaterThan<
    Num1 extends number,
    Num2 extends number,
    CountArr extends unknown[] = []
> = Num1 extends Num2 
    ? false
    : (CountArr['length'] extends Num2
        ? true
        : (CountArr['length'] extends Num1
            ? false
            : GreaterThan<Num1, Num2, [...CountArr, unknown]>));


// type GreaterThanResult = false
type GreaterThanResult = GreaterThan<3, 4>;

// type GreaterThanResult2 = true
type GreaterThanResult2 = GreaterThan<6, 4>;

// type GreaterThanResult3 = false
type GreaterThanResult3 = GreaterThan<6, 6>;
```

### 斐波那契数列计算

- 类型参数 `PrevArr` 是代表之前的累加值的数组。类型参数 `CurrentArr` 是代表当前数值的数组。
- 类型参数 `IndexArr` 用于记录 `index` ，每次递归加一，默认值是 `[]` ，代表从 `0` 开始。
- 类型参数 `Num` 代表求数列的第几个数。
- 判断当前 `index` 也就是 `IndexArr['length']` 是否到了 `Num` ，到了就返回当前的数值 `CurrentArr['length']` 。
- 否则求出当前 `index` 对应的数值，用之前的数加上当前的数 `[...PrevArr, ... CurrentArr]` 。
- 然后继续递归， `index + 1` ，也就是 `[...IndexArr, unknown]` 。

```typescript
type FibonacciLoop<
    PrevArr extends unknown[], 
    CurrentArr extends unknown[], 
    IndexArr extends unknown[] = [], 
    Num extends number = 1
> = IndexArr['length'] extends Num
    ? CurrentArr['length']
    : FibonacciLoop<CurrentArr, [...PrevArr, ...CurrentArr], [...IndexArr, unknown], Num> 

type Fibonacci<Num extends number> = FibonacciLoop<[1], [], [], Num>;


// 1、1、2、3、5、8、13、21、34
type FibonacciResult = Fibonacci<8>;
```

## 五、联合分散可简化

### 分布式条件类型

- 当类型参数为联合类型，并且在条件类型左边直接引用该类型参数的时候，TypeScript会把每一个元素单独传入来做类型运算，最后再合并成联合类型，这种语法叫做分布式条件类型。
- TypeScript之所以这样处理联合类型也很容易理解，因为联合类型的每个元素都是互不相关的，不像数组、索引、字符串那样元素之间是有关系的。所以设计成了每一个单独处理，最后合并。
- `A extends A` 不是没意义，意义是取出联合类型中的单个类型放入 `A` 。 `A extends A` 才是分布式条件类型， `[A] extends [A]` 就不是了，只有左边是单独的类型参数才可以。
- 联合类型的这种distributive的特性确实能简化类型编程，但是也增加了认知成本，不过这也是不可避免的事。

### 下划线转驼峰

```typescript
// Camelcase 下划线转驼峰
type Camelcase<Str extends string> = 
    Str extends `${infer Left}_${infer Right}${infer Rest}`
    ? `${Left}${Uppercase<Right>}${Camelcase<Rest>}`
    : Str;

// type CamelcaseResult = "aaAaAa"
type CamelcaseResult = Camelcase<'aa_aa_aa'>;

// 对字符串数组做 Camelcase
// 类型参数 Arr 为待处理数组。
// 递归提取每一个元素做 Camelcase，因为 Camelcase 要求传入 string，这里要 & string 来变成 string 类型。
type CamelcaseArr<
  Arr extends unknown[]
> = Arr extends [infer Item, ...infer RestArr]
  ? [Camelcase<Item & string>, ...CamelcaseArr<RestArr>]
  : [];

// type CamelcaseArrResult = ["aaAaAa", "bbBbBb", "ccCcCc"]
type CamelcaseArrResult = CamelcaseArr<['aa_aa_aa', 'bb_bb_bb', 'cc_cc_cc']>;

// 那如果是联合类型呢？
// 联合类型不需要递归提取每个元素，TypeScript 内部会把每一个元素传入单独做计算，之后把每个元素的计算结果合并成联合类型。
// 对联合类型的处理和对单个类型的处理没什么区别，TypeScript 会把每个单独的类型拆开传入。不需要像数组类型那样需要递归提取每个元素做处理。
type CamelcaseUnion<Item extends string> = 
  Item extends `${infer Left}_${infer Right}${infer Rest}` 
    ? `${Left}${Uppercase<Right>}${CamelcaseUnion<Rest>}` 
    : Item;

// type CamelcaseUnionResult = "aaAaAa" | "bbBbBb" | "ccCcCc"
type CamelcaseUnionResult = CamelcaseUnion<'aa_aa_aa' | 'bb_bb_bb' | 'cc_cc_cc'>;
```

### 判断一个类型是不是联合类型

- 类型参数 `A` 、 `B` 是待判断的联合类型， `B` 默认值为 `A` ，也就是同一个类型。
- `A extends A` 这段看似没啥意义，主要是为了触发分布式条件类型，让 `A` 的每个类型单独传入。
- `[B] extends [A]` 这样不直接写 `B` 就可以避免触发分布式条件类型，那么 `B` 就是整个联合类型。
- `B` 是联合类型整体，而 `A` 是单个类型，自然不成立，而其它类型没有这种特殊处理， `A` 和 `B` 都是同一个，怎么判断都成立。
- 利用这个特点就可以判断出是否是联合类型。
- 其中有两个点比较困惑，当 A 是联合类型时：
  - `A extends A` 这种写法是为了触发分布式条件类型，让每个类型单独传入处理的，没别的意义。
  - `A extends A` 和 `[A] extends [A]` 是不同的处理，前者是单个类型和整个类型做判断，后者两边都是整个联合类型，因为只有 `extends` 左边直接是类型参数才会触发分布式条件类型。

```typescript
// IsUnion
type IsUnion<A, B = A> =
    A extends A
        ? [B] extends [A]
            ? false
            : true
        : never

// type IsUnionResult = true
type IsUnionResult = IsUnion<'a'|'b'|'c'|'d'>;

// type IsUnionResult2 = false
type IsUnionResult2 = IsUnion<[ 'a' | 'b' | 'c']>;

// TestUnion
type TestUnion<A, B = A> = A  extends A ? {a: A, b: B} : never;

// A 和 B 都是同一个联合类型，为啥值还不一样呢？
// 因为条件类型中如果左边的类型是联合类型，会把每个元素单独传入做计算，而右边不会
// 所以 A 是 'a' 的时候，B 是 'a' | 'b' | 'c'， A 是 'b' 的时候，B 是 'a' | 'b' | 'c'。。。
// type TestUnionResult = {
//     a: "a";
//     b: "a" | "b" | "c";
// } | {
//     a: "b";
//     b: "a" | "b" | "c";
// } | {
//     a: "c";
//     b: "a" | "b" | "c";
// }
type TestUnionResult = TestUnion<'a' | 'b' | 'c'>;
```

### 构造BEM

- `bem` 是 `css` 命名规范，用 `block__element--modifier` 的形式来描述某个区块下面的某个元素的某个状态的样式。
- 那么我们可以写这样一个高级类型，传入 `block, element, modifier` ，返回构造出的 `class` 名。
- 类型参数 `Block, Element, Modifier` 分别是 `bem` 规范的三部分，其中 `Element` 和 `Modifiers` 都可能多个，约束为 `string[]` 。
- 构造一个字符串类型，其中 `Element` 和 `Modifiers` 通过索引访问来变为联合类型。

```typescript
type BEM<
    Block extends string,
    Element extends string[],
    Modifiers extends string[]
> = `${Block}__${Element[number]}--${Modifiers[number]}`;

// type bemResult = "guang__aaa--warning" | "guang__aaa--success" | "guang__bbb--warning" | "guang__bbb--success"
type bemResult = BEM<'guang', ['aaa', 'bbb'], ['warning', 'success']>;
```

### 构造全组合

- 类型参数 `A` 、 `B` 是待组合的两个联合类型， `B` 默认是 `A` 也就是同一个。
- `A extends A` 的意义就是让联合类型每个类型单独传入做处理，上面我们刚学会。
- `A` 的处理就是 `A` 和 `B` 中去掉 `A` 以后的所有类型组合，也就是 `Combination<A, B 去掉 A 以后的所有组合>` 。
- 而 `B` 去掉 `A` 以后的所有组合就是 `AllCombinations<Exclude<B, A>>` ，所以全组合就是 `Combination<A, AllCombinations<Exclude<B, A>>>` 。

```typescript
// 希望传入 'A' | 'B' 的时候，能够返回所有的组合： 'A' | 'B' | 'BA' | 'AB'。
type Combination<A extends string, B extends string> =
    | A
    | B
    | `${A}${B}`
    | `${B}${A}`;
 
type AllCombinations<A extends string, B extends string = A> = A extends A
    ? Combination<A, AllCombinations<Exclude<B, A>>>
    : never;

// type AllCombinationsResult = "A" | "B" | "C" | "BC" | "CB" | "AB" | "AC" | "ABC" | "ACB" | "BA" | "CA" | "BCA" | "CBA" | "BAC" | "CAB"
type AllCombinationsResult = AllCombinations<'A' | 'B' | 'C'>;
```

## 六、特殊特性要记清

### 如何判断一个类型是any类型呢

- `any` 类型与任何类型的交叉都是 `any` ，也就是 `1 & any` 结果是 `any` ，可以用这个特性判断 `any` 类型。

```typescript
type IsAny<T> = 'dong' extends ('guang' & T) ? true : false

// type IsAnyResult = true
type IsAnyResult = IsAny<any>;

// type IsAnyResult2 = false
type IsAnyResult2 = IsAny<'guang'>;
```

### 如何判断两个类型是否相等

```typescript
// 之前的不能判断有any的情况
type IsEqual<A, B> = (A extends B ? 1 : 2) & (B extends A ? 1 : 2);

// type IsEqualRes = 1
type IsEqualRes = IsEqual<'a', any>;

type IsEqual2<A, B> = (<T>() => T extends A ? 1 : 2) extends (<T>() => T extends B ? 1 : 2)
    ? true : false;

// type IsEqual2Res = false
type IsEqual2Res = IsEqual2<'a', any>;
```

### 如何判断一个类型是不是联合类型

- 联合类型作为类型参数出现在条件类型左侧时，会分散成单个类型传入，最后合并。

```typescript
type IsUnion<A, B = A> =
    A extends A
        ? [B] extends [A]
            ? false
            : true
        : never

// type IsUnionResult = true
type IsUnionResult = IsUnion<'A' | 'B'>;

// type IsUnionResult2 = false
type IsUnionResult2 = IsUnion<'A'>;
```

### 如何判断一个类型是never类型

- `never` 作为类型参数出现在条件类型左侧时，会直接返回 `never` 。
- `any` 作为类型参数出现在条件类型左侧时，会直接返回 `trueType` 和 `falseType` 的联合类型。

```typescript
// x type TestNever<T> = T extends number ? 1 : 2;传入never会返回never
// 除此以外，any 在条件类型中也比较特殊，如果类型参数为 any，会直接返回 trueType 和 falseType 的合并
// type TestAny<T> = T extends number ? 1 : 2;
// type TestAnyRes = 1 | 2
// type TestAnyRes = TestAny<any>;
type IsNever<T> = [T] extends [never] ? true : false

// type IsNeverResult = true
type IsNeverResult = IsNever<never>;

// type IsNeverResult2 = false
type IsNeverResult2 = IsNever<any>;
```

### 判断一个类型是不是元组类型

- 元组类型也是数组类型，但 `length` 是数字字面量，而数组的 `length` 是 `number` 。可以用来判断元组类型。
- 类型参数 `T` 是要判断的类型。
- 首先判断 `T` 是否是数组类型，如果不是则返回 `false` 。如果是继续判断 `length` 属性是否是 `number` 。
- 如果是数组并且 `length` 不是 `number` 类型，那就代表 `T` 是元组。

```typescript
// type len = 3
type len = [1,2,3]['length'];

// type len2 = number
type len2 = number[]['length']

type IsTuple<T> = 
    T extends [...params: infer Eles] 
        ? NotEqual<Eles['length'], number> 
        : false;

type NotEqual<A, B> = 
    (<T>() => T extends A ? 1 : 2) extends (<T>() => T extends B ? 1 : 2)
        ? false : true;

// type IsTupleResult = true
type IsTupleResult = IsTuple<[1, 2, 3]>;

// type IsTupleResult2 = false
type IsTupleResult2 = IsTuple<number[]>;
```

### 联合类型转交叉类型

- 类型之间是有父子关系的，更具体的那个是子类型，比如 `A` 和 `B` 的交叉类型 `A & B` 就是联合类型 `A | B` 的子类型，因为更具体。
- 如果允许父类型赋值给子类型，就叫做逆变。
- 如果允许子类型赋值给父类型，就叫做协变。
- 在TypeScript中有函数参数是有逆变的性质的，也就是如果参数可能是多个类型，参数类型会变成它们的交叉类型。
- 类型参数 `U` 是要转换的联合类型。 `U extends U` 是为了触发联合类型的distributive的性质，让每个类型单独传入做计算，最后合并。利用 `U` 做为参数构造个函数，通过模式匹配取参数的类型。

```typescript
type UnionToIntersection<U> = 
    (U extends U ? (x: U) => unknown : never) extends (x: infer R) => unknown
        ? R
        : never;

// type UnionToIntersectionResult = {
//     guang: 1;
// } & {
//     dong: 2;
// }
type UnionToIntersectionResult = UnionToIntersection<{ guang: 1 } | { dong: 2 }>;
```

### 提取索引类型中的可选索引

- 利用可选索引的特性：可选索引的值为 `undefined` 和值类型的联合类型。
- 过滤可选索引，就要构造一个新的索引类型，过程中做过滤。
- 类型参数 `Obj` 为待处理的索引类型，类型约束为索引为 `string` 、值为任意类型的索引类型 `Record<string, any>` 。
- 用映射类型的语法重新构造索引类型，索引是之前的索引也就是 `Key in keyof Obj` ，但要做一些过滤，也就是 `as` 之后的部分。
- 过滤的方式就是单独取出该索引之后，判断空对象是否是其子类型。
- 这里的 `Pick` 是ts提供的内置高级类型，就是取出某个 `Key` 构造新的索引类型。
- 可选的意思是这个索引可能没有，没有的时候，那 `Pick<Obj, Key>` 就是空的，所以 `{} extends Pick<Obj, Key>` 就能过滤出可选索引。
- 值的类型依然是之前的，也就是 `Obj[Key]` 。

```typescript
type GetOptional<Obj extends  Record<string, any>> = {
    [
        Key in keyof Obj 
            as {} extends Pick<Obj, Key> ? Key : never
    ] : Obj[Key];
}

// type GetOptionalResult = {
//     age?: number | undefined;
// }
type GetOptionalResult = GetOptional<{
  name: string;
  age?: number;
}>;
```

### 过滤所有非可选的索引构造成新的索引类型

```typescript
type isRequired<Key extends keyof Obj, Obj> = 
    {} extends Pick<Obj, Key> ? never : Key;

type GetRequired<Obj extends Record<string, any>> = { 
    [Key in keyof Obj as isRequired<Key, Obj>]: Obj[Key] 
}

// type GetRequiredResult = {
//     name: string;
// }
type GetRequiredResult = GetRequired<{
  name: string;
  age?: number;
}>;

```

### 过滤可索引签名

- 索引签名不能构造成字符串字面量类型，因为它没有名字，而其他索引可以。
- 类型参数 `Obj` 是待处理的索引类型，约束为 `Record<string, any>` 。
- 通过映射类型语法构造新的索引类型，索引是之前的索引 `Key in keyof Obj` ，但要做一些过滤，也就是 `as` 之后的部分。
- 如果索引是字符串字面量类型，那么就保留，否则返回 `never` ，代表过滤掉。
- 值保持不变，也就是 `Obj[Key]` 。

```typescript
type RemoveIndexSignature<Obj extends Record<string, any>> = {
  [
      Key in keyof Obj 
          as Key extends `${infer Str}`? Str : never
  ]: Obj[Key]
}

// type RemoveIndexSignatureResult = {
//     sleep: () => void;
// }
type RemoveIndexSignatureResult = RemoveIndexSignature<{
  [key: string]: any;
  sleep(): void;
}>;
```

### 过滤出类的public属性

- `keyof` 只能拿到 `class` 的 `public` 的索引， `private` 和 `protected` 的索引会被忽略。
- 类型参数 `Obj` 为带处理的索引类型，类和对象都是索引类型，约束为 `Record<string, any>` 。
- 构造新的索引类型，索引是 `keyof Obj` 过滤出的索引，也就是 `public` 的索引。
- 值保持不变，依然是 `Obj[Key]` 。

```typescript
type ClassPublicProps<Obj extends Record<string, any>> = {
    [Key in keyof Obj]: Obj[Key]    
}

class Dong {
  public name: string;
  protected age: number;
  private hobbies: string[];

  constructor() {
    this.name = 'dong';
    this.age = 20;
    this.hobbies = ['sleep', 'eat'];
  }
}

// type ClassPublicPropsResult = {
//     name: string;
// }
type ClassPublicPropsResult = ClassPublicProps<Dong>;
```

### 推导字面量类型

- TypeScript默认推导出来的类型并不是字面量类型。但是类型编程很多时候是需要推导出字面量类型的，这时候就需要用 `as const` 。但是加上 `as const` 之后推导出来的类型是带有 `readonly` 修饰的，所以再通过模式匹配提取类型的时候也要加上 `readonly` 的修饰才行。（ `const` 是常量的意思，也就是说这个变量首先是一个字面量值，而且还不可修改，有字面量和 `readonly` 两重含义。所以加上 `as const` 会推导出 `readonly` 的字面量类型。）

```typescript
const obj = {
    a: 1,
    b: 2
}

// type objType = {
//     a: number;
//     b: number;
// }
type objType = typeof obj;

const arr = [1, 2, 3]

// type arrType = number[]
type arrType = typeof arr;

const obj2 = {
    a: 1,
    b: 2
} as const;

// type objType2 = {
//     readonly a: 1;
//     readonly b: 2;
// }
type objType2 = typeof obj2;

const arr2 = [1, 2, 3] as const;

// type arrType2 = readonly [1, 2, 3]
type arrType2 = typeof arr2;

type ReverseArr<Arr> = Arr extends readonly [infer A, infer B, infer C] ? [C, B, A] : never;

// type ReverseArrRes = [3, 2, 1]
type ReverseArrRes = ReverseArr<arrType2>;
```

## 七、大型体操

### Query查询语句转对象类型

- `a=1&a=2&c=3&d=4` 这样的字符串明显是 `query param` 个数不确定的，遇到数量不确定的问题，条件反射的就要想到递归：递归解析出每一个 `query params` ，也就是 `&` 分隔的每个字符串，每个字符串单独去解析，构造成索引类型，最后把这些所有的单个索引类型合并就行。第一步并不知道有多少个 `a=1, b=2` 这种 `query param` ，要递归的做模式匹配来提取。然后每一个 `query param` 再通过模式匹配取出 `key` 和 `value` ，构造成索引类型。然后把每个索引类型合并成一个大的索引类型就可以了。

```typescript
// ParseParam 的实现就是提取和构造
// 类型参数 Param 类单个的 query param，比如 a=1 这种。
// 通过模式匹配提取 key 和 value 到 infer 声明的局部变量 Key、Value 里。
// 通过映射类型语法构造成索引类型返回：
type ParseParam<Param extends string> = 
    Param extends `${infer Key}=${infer Value}`
        ? {
            [K in Key]: Value 
        } : {};

// type ParseParamResult = {
//     a: "1";
// }
type ParseParamResult = ParseParam<'a=1'>;

// MegeValues 的合并逻辑就是如果两个值是同一个就返回一个，否则构造一个数组类型来合并：
// 类型参数 One、Other 是要合并的两个值。
// 如果两者是同一个类型，也就是 One extends Other，就返回任意一个。
// 否则，如果是数组就做数组合并，否则构造一个数组把两个类型放进去。
type MergeValues<One, Other> = 
    One extends Other 
        ? One
        : Other extends unknown[]
            ? [One, ...Other]
            : [One, Other];

// 每个 query param 处理完了，最后把这一系列构造出的索引类型合并成一个就行了：
// 类型参数 OneParam、OtherParam 是要合并的 query param，约束为索引类型（索引为 string，索引值为任意类型。
// 构造一个新的索引类型返回，索引来自两个的合并，也就是 Key in keyof OneParam | keyof OtherParam。
// 值也要做合并：
// 如果两个索引类型中都有，那就合并成一个，也就是 MergeValues<OneParam[Key], OtherParam[Key]>。
// 否则，如果是 OneParam 中的，就取 OneParam[Key]，如果是 OtherParam 中的，就取 OtherParam[Key]。
type MergeParams<
    OneParam extends Record<string, any>,
    OtherParam extends Record<string, any>
> = {
  [Key in keyof OneParam | keyof OtherParam]: 
    Key extends keyof OneParam
        ? Key extends keyof OtherParam
            ? MergeValues<OneParam[Key], OtherParam[Key]>
            : OneParam[Key]
        : Key extends keyof OtherParam 
            ? OtherParam[Key] 
            : never
}

// type MergeParamsResult = {
//     a: 1;
//     b: 2;
// }
type MergeParamsResult = MergeParams<{ a: 1 }, { b: 2 }>;

// 类型参数 Str 为待处理的 query 字符串，通过 extends 约束为 string 类型。
// 提取 & 分割的字符串到 infer 声明的局部变量 Param 里，后面的字符串放到 Rest 里。
// 通过 ParseParam 来处理单个的 query param，剩下 query 字符串也是一样的递归处理，然后把这些处理结果合并到一起，也就是 MergeParams。
// 当提取不出 & 分割的字符串时递归结束，把剩下的字符串也用 ParseParam 来处理。
type ParseQueryString<Str extends string> = 
    Str extends `${infer Param}&${infer Rest}`
        ? MergeParams<ParseParam<Param>, ParseQueryString<Rest>>
        : ParseParam<Str>;

// type ParseQueryStringResult = {
//     a: ["1", "2"];
//     b: "2";
//     c: "3";
// }
type ParseQueryStringResult = ParseQueryString<'a=1&a=2&b=2&c=3'>;
```

```typescript

```

```typescript

```

```typescript

```

```typescript

```

```typescript

```


## 八、内置高级类型

```typescript

```

```typescript

```


```typescript

```

```typescript

```

```typescript

```

```typescript

```

### Uppercase、Lowercase、Capitalize、Uncapitalize

- 这四个类型是分别实现大写、小写、首字母大写、去掉首字母大写的。
- 它们的源码中的 `intrinsic` 是固有的意思，就像js里面的有的方法打印会显示 `[native code]` 一样。这部分类型不是在ts里实现的，而是编译过程中由js实现的。其实就是ts编译器处理到这几个类型时就直接用js给算出来了。为啥要这样做呢？因为快啊，解析类型是要处理 `AST` 的，性能比较差，用js直接给算出来那多快呀。

```typescript
type Uppercase<S extends string> = intrinsic;

type Lowercase<S extends string> = intrinsic;

type Capitalize<S extends string> = intrinsic;

type Uncapitalize<S extends string> = intrinsic;

// type UppercaseRes = "AAAA"
type UppercaseRes = Uppercase<'aaaa'>;

// type LowercaseRes = "aaa"
type LowercaseRes = Lowercase<'AAA'>;

// type CapitalizeRes = "Aaa"
type CapitalizeRes = Capitalize<'aaa'>;

// type UncapitalizeRes = "aaa"
type UncapitalizeRes = Uncapitalize<'Aaa'>;
```


## 九、infer extends

- Typescript支持 `infer` 类型，可以通过模式匹配的方式，提取一部分类型返回。
- 但是 `infer` 提取出的类型是 `unknown` ，后面用的时候需要类似和 `string` 取交叉类型，或者 `xxx extends string` 这样的方式来转换成别的类型来用。这样比较麻烦。
- 所以TS4.7实现了 `infer extends` 的语法，可以指定推导出的类型，这样简化了类型编程。
- 而且 `infer extends` 还可以用来做类型转换，比如 `string` 转 `number` 、转 `boolean` 等。
- 要注意的是，4.7的时候，推导出的只是 `extends` 约束的类型，比如 `number` 、 `boolean` ，但是4.8就能推导出字面量类型了，比如 `1` 、 `2` 、 `true` 、 `false` 这种。
- 有了 `infer extends` ，不但能简化类型编程，还能实现一些之前很难实现的类型转换。

```typescript
enum Code {
    a = 111,
    b = 222,
    c = 'abc'
}

type StrToNum<Str> =
  Str extends `${infer Num extends number}`
    ? Num
    : Str

// type res = "abc" | 111 | 222
type res = StrToNum<`${Code}`>;

type StrToBoolean<Str> =
  Str extends `${infer Bool extends boolean}`
    ? Bool
    : Str

// type res2 = true
type res2 = StrToBoolean<'true'>;

type StrToNull<Str> =
  Str extends `${infer Null extends null}`
    ? Null
    : Str

// type res3 = null
type res3 = StrToNull<'null'>;
```

## 十、变

### 型变

- 子类型可以赋值给父类型，叫做协变（covariant）。
- 父类型可以赋值给子类型，叫做逆变（contravariant）。函数的参数有逆变的性质（而返回值是协变的，也就是子类型可以赋值给父类型）。
- 在ts2.x之前支持父类型可以赋值给子类型，子类型可以赋值给父类型，既逆变又协变，叫做“双向协变”。但是这明显是有问题的，不能保证类型安全，所以之后ts加了一个编译选项 `strictFunctionTypes` ，设置为 `true` 就只支持函数参数的逆变，设置为 `false` 则是双向协变。

### 不变（invariant）

- 非父子类型之间不会发生型变，只要类型不一样就会报错。

### 类型父子关系的判断

- 像java里面的类型都是通过 `extends` 继承的，如果 `A extends B` ，那 `A` 就是 `B` 的子类型。这种叫做名义类型系统（nominal type）。
- 而ts里不看这个，只要结构上是一致的，那么就可以确定父子关系，这种叫做结构类型系统（structual type）。
