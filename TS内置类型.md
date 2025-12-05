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
**`Exclude<UnionType, ExcludedMembers>`**
- 从联合类型 UnionType 中排除 ExcludedMembers 中的所有联合成员来构造一个新的类型。
```ts
type HZFEMemberProps = Exclude<"id" | "name" | "age", "age">;

const hzferProp: HZFEMemberProps = "age"; // Type '"age"' is not assignable to type 'HZFEMemberProps'.
```

```ts
type Exclude<T, U> = T extends U ? never : T;
```
**`Extract<UnionType, ExcludedMembers>`**
- 从联合类型中保留（包含）指定成员。
```ts
/**
 * Extract<T, U>：从类型 T 中保留可以赋值给类型 U 的成员
 * 实际上就是 Pick<T, keyof T & U> 的思路，但在联合类型上更直接。
 */
type Extract<T, U> = T extends U ? T : never;
```
**`Pick<Type, Keys>`**
- 从一个已有的类型 Type 中选择一组属性 Keys 来构造一个新的类型。
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
- 从一个已有的类型 Type 中移除一组属性 Keys 来构造一个新的类型。
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