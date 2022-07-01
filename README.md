# TypeScript Conventions

Practices I follow when writing TypeScript code

## Types and Interfaces

if a Type is not abstract, name it what it is

```tsx
import * as React from "react";

type MyComponentProps = {
  children: React.ReactNode;
};

function MyComponent(props: MyComponentProps) {
  return <div {...props} />;
}
```

if a type is abstract, prefix it with a `T` or an `I` depending on if its a Type or an Interface.

```ts
interface IForm<Values> {
  values: Values;
  errors: Record<keyof Values, string>;
  touched: Record<keyof Values, boolean>;
  onSubmit: (values: Values) => void;
}

type MyForm = IForm<{ firstName: string; lastName: string }>;
```

Use an Interface when you want to take advantage of declaration merging.

```ts
// config.ts
export interface Config {}

// later in logging.ts
import type { Config } from "./config"; // if you need to use `Config`

declare module "./config" {
  interface Config {
    logging: {
      level: "debug" | "info" | "warn" | "error";
    };
  }
}

// later in database.ts

import type { Config } from "./config"; // if you need to use `Config`
declare module "./config" {
  interface Config {
    database: {
      adapter: "mysql" | "postgres";
    };
  }
}

// later in app.ts
import type { Config } from "./config";
import "./database";
import "./logging";

type Foo = Config;
// {
//   logging: {
//     level: "debug" | "info" | "warn" | "error";
//   };
//   database: {
//     adapter: "mysql" | "postgres";
//   };
// }
```

## Generics

When using Generics, name it after the argument, and prefix it with a `T` since it's a Type.

This will help avoid confusion with less descriptive types like `T` or `P`.

```ts
function identity<TValue>(value: TValue): TValue {
  return value;
}
```

## Shared Types

When you need to use multiple types, don't repeat yourself and follow the Open Closed Principle.

```tsx
import * as React from "react";

type MyComponentProps = {
  children: (value: number) => React.ReactNode;
};

function MyComponent({ children }: MyComponentProps) {
  return <>{children(42)}</>;
}

type MyWrappedComponentProps = {
  // we don't want to know what the children are,
  // because the usage is defined else where,
  // we just use the type to unobservably pass it down,
  // and if we break MyComponent the type system will save us
  // because the runtime of this component will change
  children: MyComponentProps["children"];
};

function MyWrappedComponent({ children }: MyWrappedComponentProps) {
  return (
    <div>
      <MyComponent>{children}</MyComponent>
    </div>
  );
}

function Usage() {
  <MyWrappedComponent>{(value) => <>{value}</>}</MyWrappedComponent>;
}
```
