https://www.youtube.com/watch?v=RebA5J-rlwg

# Initialization

`npx prisma init --datasource-provider postgresql`

# vscode settings

`npx prisma format`

settings.json

```json
 "[prisma]": {
    "editor.defaultFormatter": "Prisma.prisma",
    "editor.formatOnSave": true
  },
```

# Migration

- database는 미리 만들어져 있어야 된다.
- `npx prisma migrate dev --name init`
- migrate를 하는 순간 db에 migration이 진행된다. client update도 같이 진행된다.

```
migrations/
  └─ 20240115022103_init/
    └─ migration.sql

Your database is now in sync with your schema.

✔ Generated Prisma Client (v5.8.0) to ./node_modules/.pnpm/@prisma+client@5.8.0_prisma@5.8.0/node_modules/@prisma/client in 151ms
```

`npx prisma generate`

- client를 다시 만드는 명령어.

`npx prisma migrate dev --create-only`

- migration file만 만들고, migration은 적용 안한 상태
- migration file 수정이후 `npx prisma migrate dev`로 migration 적용 가능

`npx prisma migrate deploy`

- production env. 에서 진행

# Schema file이 single source of the truth

```
model User {
  id    Int     @id @default(autoincrement())
  name  String?
  email String
}
```

?는 optional

# One to Many relations

```
model User {
  id      Int     @id @default(autoincrement())
  ...
  Post Post[]
}

model Post {
  ...
  authorId Int
  author User @relation(fields: [authorId], references: [id])
}

```

# More than one relations

```
model User {
  id      String  @id @default(uuid())
  ...

  writtenPosts  Post[] @relation("WrittePosts")
  favoritePosts Post[] @relation("FavoritePosts")
}

model Post {
  id     Int   @id @default(autoincrement())
  ...

  authorId String
  author   User   @relation("WrittePosts", fields: [authorId], references: [id])

  favoriatedById String
  favoriatedBy   User   @relation("FavoritePosts", fields: [favoriatedById], references: [id])

}

```

# Many to Many relations

```
model Post {
  id     String @id @default(uuid())
  ...

  categories Category[]
}

model Category {
  id    String @id @default(uuid())
  posts Post[]
}

```

# Save하면 자동으로 relation이 만들어진다.

```prisma

model UserPreference {
  id           String  @id @default(uuid())
  emailUpdates Boolean
  user         User
  userId       String
}

==> Save

model User {
  UserPreference UserPreference[]
}

model UserPreference {
  id           String  @id @default(uuid())
  emailUpdates Boolean
  user         User    @relation(fields: [userId], references: [id])
  userId       String
}
```

# One to One relation

```prisma
model User {
  ...
  UserPreference UserPreference?
}

model UserPreference {
  id           String  @id @default(uuid())
  emailUpdates Boolean
  user         User    @relation(fields: [userId], references: [id])
  userId       String  @unique
}
```

# `updateAt`

```prisma
model User {
  updatedAt DateTime @updatedAt
}
```

# default

```prisma

model User {
  createdAt DateTime  @default(now())
}

```

# block level unique

```prisma
model User {
  @@unique([age, name])
}
```

age, name이 같은 record가 존재할 수 없음

# id를 column으로

```
model Post {
  @@id([title, authorId])
}
```

`const prisma = new PrismaClient();`
하나의 PrismaClient를 사용한다.

# .create

```javascript
const user = await prisma.user.create({
  data: {
    name: "sg",
    email: "sg@test.com",
    age: 38,
    isAdmin: true,
    userPreference: {
      create: {
        emailUpdates: true,
      },
    },
  },
  // include: { userPreference: true },
  select: {
    name: true,
    userPreference: { select: { id: true } },
  },
});
```

include로 relation을 추가해서 object를 받을 수도 있고
select로 attribute을 선택해서 해당 attribute만 가져올 수도 있음

# .createMany

```javascript
const user = await prisma.user.createMany({
  data: [
    {
      name: "sg",
      email: "sg@test.com",
      age: 38,
      isAdmin: true,
    },
    {
      name: "sg1",
      email: "sg1@test.com",
      age: 38,
      isAdmin: true,
    },
  ],
});
```

# .findUnique

```js
const user = await prisma.user.findUnique({
  where: {
    // email: "sg@test.com",
    age_name: {
      age: 38,
      name: "sg",
    },
  },
  include: { userPreference: true },
});
```

schema에서 unique로 선언된 attribute만 where에서 사용가능
unique가 아니라면 아래의 findFirst 사용

# .findFirst

```js
const user = await prisma.user.findFirst({
  where: {
    name: "sg",
  },
  include: { userPreference: true },
});
```

# .findMany

```js
const user = await prisma.user.findMany({
  where: {
    name: "Sally",
  },
  orderBy: {
    age: "desc",
  },
  // distinct: ["name", "age"],
  take: 2,
  skip: 1,
});
```

distinct는 중복제거, take은 가져올 record 수, skip은 가져오지 않을 record 수

# where

```js
const user = await prisma.user.findMany({
  where: {
    name: { not: "Sally" },
  },
});
const user = await prisma.user.findMany({
  where: {
    name: { in: ["Sally"] },
  },
});
const user = await prisma.user.findMany({
  where: {
    name: { notIn: ["Sally"] },
  },
});
const user = await prisma.user.findMany({
  where: {
    name: "Sally",
    age: { lt: 20 },
  },
});
const user = await prisma.user.findMany({
  where: {
    name: "sg",
    age: { gte: 20 },
  },
});
const user = await prisma.user.findMany({
  where: {
    email: { contains: "@test.com" },
  },
});

const user = await prisma.user.findMany({
  where: {
    // email: { endsWith: "@test1.com" },
    email: { startsWith: "sg" },
  },
});

const user = await prisma.user.findMany({
  where: {
    AND: [{ email: { startsWith: "sg" } }, { name: "Sally" }],
  },
});
const user = await prisma.user.findMany({
  where: {
    OR: [{ email: { startsWith: "sally" } }, { age: { gt: 20 } }],
  },
});

const user = await prisma.user.findMany({
  where: {
    NOT: { email: { startsWith: "sally" } },
  },
});
```

# Query on relationships

```js
const user = await prisma.user.findMany({
  where: {
    userPreference: {
      emailUpdates: true,
    },
  },
});
const user = await prisma.post.findMany({
  where: {
    author: {
      is: {
        age: 27,
      },
    },
  },
});
```

# .update

```js
const user = await prisma.user.update({
  where: {
    email: "sally@test1.com",
  },
  data: {
    email: "sally@test4.com",
  },
});
const user = await prisma.user.update({
  where: {
    email: "sg@test.com",
  },
  data: {
    age: {
      decrement: 1,
    },
  },
});
const user = await prisma.user.update({
  where: {
    email: "sg@test.com",
  },
  data: {
    userPreference: {
      create: {
        emailUpdates: true,
      },
    },
  },
});
const user = await prisma.user.update({
  where: {
    email: "sg@test.com",
  },
  data: {
    userPreference: {
      connect: {
        id: "624b348f-4ce2-4f9b-963b-a943c8a72f68",
      },
    },
  },
});
const user = await prisma.user.update({
  where: {
    email: "sg@test.com",
  },
  data: {
    userPreference: {
      disconnect: {
        id: "624b348f-4ce2-4f9b-963b-a943c8a72f68",
      },
    },
  },
});
```

# .updateMany

```js
const user = await prisma.user.updateMany({
  where: {
    name: "Sally",
  },
  data: {
    name: "New Sally",
  },
});
```

# .delete

```js
const user = await prisma.user.delete({
  where: {
    email: "sg@test.com",
  },
});
```

# .deleteAll

```js
const user = await prisma.user.deleteMany({
  where: {
    age: { gt: 20 },
  },
});
```
