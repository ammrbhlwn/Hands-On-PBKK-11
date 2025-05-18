# Hands-On Migrasi Database Menggunakan Prisma ORM


# 🛠️ Tools

<table>
  <tr>
     <td>NextJs</td>
     <td>NextAuth</td>
     <td>Shadcn</td>
     <td>Prisma ORM</td>
     <td>Vercel</td>
  </tr>
  <tr>
    <td>
        <a href="https://nextjs.org/">
           <img src="https://marcbruederlin.gallerycdn.vsassets.io/extensions/marcbruederlin/next-icons/0.1.0/1723747598319/Microsoft.VisualStudio.Services.Icons.Default" alt="NextJs" width="200">
        </a>
    </td>
    <td>
        <a href="https://next-auth.js.org/">
           <img src="https://next-auth.js.org/img/logo/logo-sm.png" alt="NextAuth" width="200">
        </a>
    </td>
    <td>
        <a href="https://ui.shadcn.com/">
           <img src="https://avatars.githubusercontent.com/u/139895814?s=280&v=4" alt="Shadcn" width="200">
        </a>
    </td>
    <td>
        <a href="https://www.prisma.io/">
           <img src="https://cdnlogo.com/logos/p/67/prisma.svg" alt="Prisma" width="200">
        </a>
    </td>
    <td>
        <a href="https://vercel.com">
           <img src="https://static.wikia.nocookie.net/logopedia/images/a/a7/Vercel_favicon.svg/revision/latest?cb=20221026155821" alt="Vercel" width="200">
        </a>
    </td>
  </tr>
 </table>

## 1. Install NextJs 
[Instalasi NextJs](https://nextjs.org/docs/app/getting-started/installation)
```
npx create-next-app@latest
```

## 2. Jalankan Aplikasi
```
npm run dev
```

## 3. Setup Vercel Postgres Database
### Create postgres database (Serverless SQL)
- Buka Dashboard [Vercel](vercel.com)
- Buka menu storage
- Klik Create Database
- Pilih Serverless Potsgres
- Pilih region secara default
- Pilih free plan
- Buat nama database
- Klik Create

### Setup .env
- Copy .env.local
- Buat file baru di file project kalian dengan nama .env
- Paste .env.local

## 4. Install Prisma
[Instalasi Prisma](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-prismaPostgres)
```
npm install prisma
```

```
npx prisma init 
```

## 5. Buat data model ``prisma.ts``
- ### Buka File ``prisma/schema.prisma``

```
enum RoleUser {
  STUDENT
  ADMIN
}
```

```
model User {
  id        String @id @default(cuid())
  name      String
  email     String @unique
  password  String
  role      RoleUser
  sessions  Session[]
}
```

```
model Session {
  id        String @id
  userId    String
  expiresAt DateTime
  user      User @relation(references: [id], fields: [userId], onDelete: Cascade)
}
```

- ### Jalankan Migrasi Database
```
npx prisma migrate dev
```


## 6. Install Prisma Client
[Instalasi Prisma Client](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases/install-prisma-client-typescript-postgresql)

- ### Install Prisma Client
```
npm install @prisma/client
```

- ### Buat Folder lib/prisma.ts di root project
```
import { PrismaClient } from '@prisma/client';

let prisma: PrismaClient;

declare const globalThis: {
  prisma: PrismaClient;
};

if (process.env.NODE_ENV === 'production') {
  prisma = new PrismaClient();
} else {
  if (!globalThis.prisma) {
    globalThis.prisma = new PrismaClient();
  }

  prisma = globalThis.prisma;
}

export default prisma;
```

## 7. Buat File ``src/lib/auth.ts``
```
import NextAuth from "next-auth"
import { PrismaAdapter } from "@auth/prisma-adapter"
import prisma from "../../lib/prisma"
 
export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [],
})
```

## 8. Buat data seeder ``prisma/seed.ts``
[Dokumentasi seeding](https://www.prisma.io/docs/orm/prisma-migrate/workflows/seeding)

- ### Install Bcrypt
```
npm i bcrypt
```

- ### Isi File ``seed.ts``
```
const { PrismaClient } = require('@prisma/client');
const bcrypt = require('bcrypt');

const prisma = new PrismaClient();

async function main() {
  const passwordAdmin = bcrypt.hashSync('admin123', 10);
  const passwordMahasiswa = bcrypt.hashSync('mahasiswa123', 10);

  const userSeed = await prisma.user.createMany({
    data: [
      {
        email: 'admin@mail.com',
        name: 'Admin',
        role: 'ADMIN',
        passwordAdmin,
      },
      {
        email: 'mahasiswa@mail.com',
        name: 'Mahasiswa',
        role: 'STUDENT',
        passwordMahasiswa,
      },
    ],
    skipDuplicates: true,
  });

  console.log(userSeed)
}

main()
```

- ### Tambahkan di ``package.json``
```
"prisma": {
  "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
},
```

- ### Install ``ts-node``
```
npm install -D typescript ts-node @types/node
```

- ### Jalankan seeder
```
npx prisma db seed
```

## 10. Cek Data di Prisma Studio
```
npx prisma studio
```

- ### Buat Folder lib/prisma.ts di root project
```
import { PrismaClient } from '@prisma/client';

let prisma: PrismaClient;

declare const globalThis: {
  prisma: PrismaClient;
};

if (process.env.NODE_ENV === 'production') {
  prisma = new PrismaClient();
} else {
  if (!globalThis.prisma) {
    globalThis.prisma = new PrismaClient();
  }

  prisma = globalThis.prisma;
}

export default prisma;
```

## 7. Buat File ``src/lib/auth.ts``
```
import NextAuth from "next-auth"
import { PrismaAdapter } from "@auth/prisma-adapter"
import prisma from "../../lib/prisma"
 
export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [],
})
```

## 8. Buat data seeder ``prisma/seed.ts``
[Dokumentasi seeding](https://www.prisma.io/docs/orm/prisma-migrate/workflows/seeding)

- ### Install Bcrypt
```
npm i bcrypt
```

- ### Isi File ``seed.ts``
```
const { PrismaClient } = require('@prisma/client');
const bcrypt = require('bcrypt');

const prisma = new PrismaClient();

async function main() {
  const passwordAdmin = bcrypt.hashSync('admin123', 10);
  const passwordMahasiswa = bcrypt.hashSync('mahasiswa123', 10);

  const userSeed = await prisma.user.createMany({
    data: [
      {
        email: 'admin@mail.com',
        name: 'Admin',
        role: 'ADMIN',
        passwordAdmin,
      },
      {
        email: 'mahasiswa@mail.com',
        name: 'Mahasiswa',
        role: 'STUDENT',
        passwordMahasiswa,
      },
    ],
    skipDuplicates: true,
  });

  console.log(userSeed)
}

main()
```

- ### Tambahkan di ``package.json``
```
"prisma": {
  "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
},
```

- ### Install ``ts-node``
```
npm install -D typescript ts-node @types/node
```

- ### Jalankan seeder
```
npx prisma db seed
```

## 10. Cek Data di Prisma Studio
```
npx prisma studio
```
