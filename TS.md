**内置工具类型**
- `Partial<Type>`
- 通过将 Type 中的所有属性都设置为可选来构造一个新的类型。

```js
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

```js
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```
- `Required<Type>`
- 通过将 Type 中的所有属性都设置为必选来构造一个新的类型。和 Partial 相反。

```js
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

```js
type Required<T> = {
  [P in keyof T]-?: T[P];
};
```
- `Exclude<UnionType, ExcludedMembers>`
- 从联合类型 UnionType 中排除 ExcludedMembers 中的所有联合成员来构造一个新的类型。

```js
type HZFEMemberProps = Exclude<"id" | "name" | "age", "age">;

const hzferProp: HZFEMemberProps = "age"; // Type '"age"' is not assignable to type 'HZFEMemberProps'.
```
```js
type Exclude<T, U> = T extends U ? never : T;
```
- `Pick<Type, Keys>`
- 从一个已有的类型 Type 中选择一组属性 Keys 来构造一个新的类型。
```js
interface Member {
  id: number;
  name: string;
  age: number;
}

type HZFEMember = Pick<Member, "id" | "name">;

const hzfer: HZFEMember = {
  id: 1,
  name: "QingZhen",
  age: 18, // Object literal may only specify known properties, and 'age' does not exist in type 'HZFEMember'.
};
```
```js
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```
- `Omit<Type, Keys>`
- 从一个已有的类型 Type 中移除一组属性 Keys 来构造一个新的类型。
```js
interface Member {
  id: number;
  name: string;
  age: number;
}

type HZFEMember = Omit<Member, "id" | "age">;

const hzfer: HZFEMember = {
  id: 1, // Object literal may only specify known properties, and 'id' does not exist in type 'HZFEMember'.
  name: "QingZhen",
};
```
```js
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```
- `ReturnType<Type>`
- 构造一个由函数的返回值的类型 Type 组成的类型。

```js
interface GetHZFEMember {
  (id: number): {
    id: number;
    name: string;
    age: number;
  };
}

type HZFEMember = ReturnType<GetHZFEMember>; // type HZFEMember = { id: number; name: string; age: number; };
```
```js
type ReturnType<T extends (...args: any) => any> = T extends (
  ...args: any
) => infer R ? R : any;
```