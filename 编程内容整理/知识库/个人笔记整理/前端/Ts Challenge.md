##### typeof
```typescript
// 拿对象、枚举类型的key =》 keyof typeof 对象、枚举
// 拿函数传参类型和返回值类型 结合ReturnType<>和Parameters<>


// 对象
const obj = {
	a: 1,
  b: 2
}
type copy = typeof b  // copy => { a: number, b: number }
type copyKey = keyof typeof b // copyKey => 'a' | 'b'


// 枚举
enum artList = {
	zhangsan,
  lisi
}
type art = keyof typeof artList // art => 'zhangsan' | 'lisi'

// 属性名在对象key中挑，而且全都有才行
type MoviesByGenre = {
  action: 'Die Hard';
  comedy: 'Groundhog Day';
};
type MovieInfoByGenre<T> = {
	[K in keyof T]: {
		name: string;
		year: number;
		director: string;
	};
};

// 用[K in T[number]]拿数组中的每一项
// 用[K in keyof T] 拿对象中的每一项key
type MoviesByGenre = ['Die Hard','Groundhog Day'];
type MovieInfoByGenre<T> = {
	[K in T[number]]: {
		name: string;
		year: number;
		director: string;
	};
};
// 那个对象的key都要有，用[K in keyof T]
const test_MoviesInfoByGenre: MovieInfoByGenre<MoviesByGenre> = {
  action: {
    name: 'Die Hard',
    year: 1988,
    director: 'John McTiernan',
  },
  comedy: {
    name: 'Groundhog Day',
    year: 1993,
    director: 'Harold Ramis',
  }
};


// 我知道传进函数的参数是这个对象的key
const casettesByArtist = {
	'Alanis Morissette': 2,
  'Mariah Carey': 8,
}
const getCasetteCount = (artist: keyof typeof casettesByArtist) => {
  return casettesByArtist[artist];
}

// 还可以这样写
const d3ChartConfig = {
	data: [
    { category: 'A', value: 30 },
    { category: 'B', value: 45 },
    { category: 'C', value: 60 },
    { category: 'D', value: 25 },
    { category: 'E', value: 50 },
  ],
}
type Data = (typeof d3ChartConfig)["data"]; // { category: string, value: number }[]

```

    1. ![](https://cdn.nlark.com/yuque/0/2024/png/28792475/1704977888203-f0749342-50c1-4f3e-ba74-23ca5bb0da61.png)
    2. ![](https://cdn.nlark.com/yuque/0/2024/png/28792475/1704977975503-aca25f74-022a-416b-97ce-17c78f6b7e75.png)
    3. ![](https://cdn.nlark.com/yuque/0/2024/png/28792475/1704978042010-13bd51b9-5e5c-47ca-a43f-046946ca9408.png)
    4. ![](https://cdn.nlark.com/yuque/0/2024/png/28792475/1704978174055-a4f5c647-d37a-4b9d-a665-9fc71279d68a.png)

##### PropertyKey，代表能作为对象属性的类型，例如<T extends PropertyKey[]>
```typescript
type TupleToObject<T extends readonly PropertyKey[]> = {
	[K in T[number]]: K
}

TupleToObject<[[1, 2], {}]> //error,因为空数组不能作为对象值=key
```



##### 如何判断值为空：**<font style="color:rgb(99, 102, 241);background-color:rgb(238, 242, 255);">extends 0</font>**
```typescript
type First<T extends any[]> = T['length'] extends 0 ? never : T[0]

  Expect<Equal<First<[3, 2, 1]>, 3>>,
  Expect<Equal<First<[() => 123, { a: string }]>, () => 123>>,
  Expect<Equal<First<[]>, never>>,
  Expect<Equal<First<[undefined]>, undefined>>,
```

##### ts也可以用递归；infer 推断类型并返回该类型
```typescript
type First3<T extends any[]> = T extends [infer First, ...any[]]
? First
: never
```

```typescript
type MyAwaited<T extends PromiseLike<any>> = T extends PromiseLike<infer U> ? (U extends PromiseLike<any> ? MyAwaited<U> : U) : never 
```

##### ts也可用用模板字符串
```typescript
type Absolute<T extends number | string | bigint> = `${T}` extends `-${infer N}` ? N : `${T}`
```

##### Exclude
##### 可以赋予默认值<T, K extends keyof T = keyof T>
```typescript
// 第一个传类型{ title: string }类似，第二个传想让其变成readonly的类型title | msg,不传全变
type MyReadonly<T, K extends keyof T = keyof T> = {
	readonly [L in K]: T[L]
} & {
	[L in keyof Omit<T,K>]: T[L]
}
```



##### 常见的ts语法糖
<details class="lake-collapse"><summary id="uc261baa0"><span class="ne-text">Omits</span></summary><pre data-language="typescript" id="fgLTe" class="ne-codeblock language-typescript"><code>interface User {
    id: number;
    name: string;
    age: number;
    email: string;
}

// 现在我们想创建一个省略 'id' 和 'email' 属性的新类型
type NewUser = Omit&lt;User, 'id' | 'email'&gt;;

// NewUser 将是 { name: string; age: number; }</code></pre></details>
<details class="lake-collapse"><summary id="u8eca8be7"><span class="ne-text">Pick</span></summary><pre data-language="typescript" id="TXZqO" class="ne-codeblock language-typescript"><code>interface User {
    id: number;
    name: string;
    age: number;
    email: string;
}

// 现在我们想从 User 接口中选择 'name' 和 'age' 属性创建一个新类型
type UserInfo = Pick&lt;User, 'name' | 'age'&gt;;

// UserInfo 将是 { name: string; age: number; }</code></pre></details>


