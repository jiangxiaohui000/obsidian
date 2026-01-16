TypeScript 内置工具类型，核心解决的是：  
在不重复定义类型的前提下，对“既有类型”进行安全、可组合、可维护的派生与约束。

**`Partial<Type>`**
- 通过将 Type 中的所有属性都设置为可选来构造一个新的类型。
```ts
interface Member {
  id: number;
  name: string;
  age: number;
}
const hzfer: Member = {
  id: 1,
  name: "Qingzhen",
}; // Property 'age' is missing in type '{ id: number; name: string; }' but required in type 'Member'.

type HZFEMember = Partial<Member>;
const hzfer2: HZFEMember = {
  id: 1,
  name: "Qingzhen",
}; // No errors
```

```ts
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```
**`Required<Type>`**
- 通过将 Type 中的所有属性都设置为必选来构造一个新的类型。和 Partial 相反。
```ts
interface Member {
  id: number;
  name: string;
  age?: number;
}
const hzfer: Member = {
  id: 1,
  name: "Qingzhen",
}; // No errors

type HZFEMember = Required<Member>;
const hzfer2: HZFEMember = {
  id: 1,
  name: "Qingzhen",
}; // Property 'age' is missing in type '{ id: number; name: string; }' but required in type 'Required<Member>'
```

```ts
type Required<T> = {
  [P in keyof T]-?: T[P];
};
```
---
**`Exclude<UnionType, ExcludedMembers>`**
- 从联合类型 UnionType 中排除 ExcludedMembers 中的所有联合成员来构造一个新的类型。
- 注意：作用于联合类型，omit作用于对象
```ts
type HZFEMemberProps = Exclude<"id" | "name" | "age", "age">;
const hzferProp: HZFEMemberProps = "age"; // Type '"age"' is not assignable to type 'HZFEMemberProps'.
```

```ts
// 配合 `keyof` 使用（高频）
type Props = {
  id: string;
  name: string;
  onClick: () => void;
};
type DataProps = Exclude<keyof Props, 'onClick'>; // 'id' | 'name'
```

```ts
type Exclude<T, U> = T extends U ? never : T;
```
**`Extract<UnionType, ExtractMembers>`**
- 从联合类型中保留（包含）指定成员。

```ts
// 从联合类型中“选合法值”
type A = 'a' | 'b' | 'c';
type B = 'a' | 'd';

type C = Extract<A, B>; // 'a'
```

```ts
type Status = 'idle' | 'loading' | 'success' | 'error';
type AsyncStatus = Extract<Status, 'loading' | 'success'>;
// 'loading' | 'success'
```

```ts
/**
 * Extract<T, U>：从类型 T 中保留可以赋值给类型 U 的成员
 * 实际上就是 Pick<T, keyof T & U> 的思路，但在联合类型上更直接。
 */
type Extract<T, U> = T extends U ? T : never;
```

**关键点：**
- `T` 必须是一个 **对象结构类型**
- 与 `T` 是 `interface` 还是 `type` **没有关系**
- **操作的是：具有 `keyof` 的类型** 也就是说，只要这个类型：
	- 能被 `keyof` 枚举属性名
	- 属性是可索引的
	就可以使用 `Pick / Omit`
---
**`Pick<Type, Keys>`**
- 从一个已有的**对象类型** Type 中选择一组属性 Keys 来构造一个新的类型。
```ts
interface Member {
  id: number;
  name: string;
  age: number;
}

type HZFEMember = Pick<Member, "id" | "name">;

const hzfer: HZFEMember = {
  id: 1,
  name: "QingZhen",
  age: 18, // ❌ Object literal may only specify known properties, and 'age' does not exist in type 'HZFEMember'.
};
```

```ts
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```
**`Omit<Type, Keys>`**
- 从一个已有的**对象类型** Type 中移除一组属性 Keys 来构造一个新的类型。
```ts
interface Member {
  id: number;
  name: string;
  age: number;
}

type HZFEMember = Omit<Member, "id" | "age">;

const hzfer: HZFEMember = {
  id: 1, // ❌ Object literal may only specify known properties, and 'id' does not exist in type 'HZFEMember'.
  name: "QingZhen",
};
```

```ts
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```
---
**`ReturnType<Type>`**
- 构造一个由函数的返回值的类型 Type 组成的类型。
```ts
interface GetHZFEMember {
  (id: number): {
    id: number;
    name: string;
    age: number;
  };
}

type HZFEMember = ReturnType<GetHZFEMember>; // type HZFEMember = { id: number; name: string; age: number; };
```

```ts
// “`T extends (...args:any)=>infer R ? R : any` 是一个条件类型，用来判断 T 是否为函数类型。  
// 如果是函数类型，就从中推断出返回值类型 R；否则返回 any。
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```
**`Record<K, T>` **
- 用于构造一个对象类型：key 的集合由 **K** 来决定，value 的类型由 **T** 来决定
```ts
type Record<K extends keyof any, T> = {
	[P in K]: T;
}
```
- `K extends keyof any`：意味着 K 可以是 `string | number | symbol` 类型的集合
- `[P in K]`：把 K 中的每一个 key 都映射成一个属性
- `T`：所有 value 的类型
```ts
type Role = "admin" | "user" | "guest";

type RoleConfig = Record<Role, number>;

const config: RoleConfig = {
  admin: 1,
  user: 2,
  guest: 3
};
```
```ts
type UserInfoMap = Record<number, string>;

const users: UserInfoMap = {
  1: "Alice",
  2: "Bob"
};

```

TypeScript 的 `extends` 判断的是  
**“T 是否可以赋值给 U”**，而不是“是否相等”。

interface与type的区别
1，
- **interface**：主要用于定义 **对象（Object）** 的结构。它使用 `extends` 关键字来继承。
- **type**：可以定义任何类型（基本类型、联合类型、元组等）。它使用 **交叉类型（Intersection Types）** `&` 来实现类似继承的效果。
2，
`type` 可以定义 `interface` 无法表达的类型：
- **联合类型 (Union Types)**：`type Status = "success" | "error"`
- **基本类型别名**：`type Name = string`
- **元组 (Tuples)**：`type Data = [number, string]`
- **映射类型 (Mapped Types)**：利用 `keyof` 动态生成类型（这是 `interface` 做不到的）。
3，
多次定义同名的 `interface`，TS 会自动将它们合并。type会报错。
4，
在大型TS项目中，在编译性能上interface比type略有优势。

“Use interface for objects, type for everything else.”
