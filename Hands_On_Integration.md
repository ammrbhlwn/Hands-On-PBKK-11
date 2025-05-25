# Hands-On Integrasi Authentication Page

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

## 1. Clone Template Project
Clone template project melalui repository berikut: **[template-authentication](https://github.com/ammrbhlwn/template-authentication)**

## 2. Install Dependencies
Setelah berhasil melakukan clone project. Silahkan jalankan perintah berikut:
```
npm install
```

## 3. Install NextAuth
[Instalasi NextAuth](https://next-auth.js.org/getting-started/example)
```
npm install next-auth
```

## 4. Buat file ```route.ts``` pada folder ```app/api/auth/[...nextauth]```
[Instalasi NextAuth](https://next-auth.js.org/configuration/nextjs)
```
import NextAuth from 'next-auth'
import { authOptions } from '@/lib/authOptions'

const handler = NextAuth(authOptions)

export { handler as GET, handler as POST }
```

## 5. Buat file ```authOption.ts``` pada folder ```src/lib```
[Konfigurasi NextJs](https://next-auth.js.org/configuration/nextjs)
```
/* eslint-disable @typescript-eslint/no-explicit-any */
import { NextAuthOptions } from 'next-auth'
import CredentialsProvider from 'next-auth/providers/credentials'
import { prisma } from '../../lib/prisma'
const bcrypt = require('bcrypt');

export const authOptions: NextAuthOptions = {
  session: {
    strategy: 'jwt',
  },
  secret: process.env.NEXTAUTH_SECRET,
  providers: [
    CredentialsProvider({
      name: 'Credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) return null

        const user = await prisma.user.findUnique({
          where: { email: credentials.email },
        })

        if (!user) return null

        const isPasswordValid = await bcrypt.compare(
          credentials.password,
          user.password,
        )

        if (!isPasswordValid) return null

        return user
      },
    }),
  ],
  callbacks: {
    async jwt({ token, user, account }) {
      if (account?.provider === 'credentials') {
        token.id = user.id
        token.name = user.name
        token.email = user.email
      }
      return token
    },
    async session({ session, token }: any) {
      if ('email' in token) {
        session.user.email = token.email
      }
      if ('name' in token) {
        session.user.name = token.name
      }
      return session
    },
  },
  pages: {
    signIn: '/login',
  },
}
```

## 6. Buat file ```middleware.ts``` pada folder ```src```
[Konfigurasi NextJs](https://next-auth.js.org/configuration/nextjs)
```
import { NextRequest, NextResponse } from 'next/server'
import { getToken } from 'next-auth/jwt'

export async function middleware(req: NextRequest) {
  const { pathname } = req.nextUrl
  const token = await getToken({ req, secret: process.env.NEXTAUTH_SECRET })

  if (
    pathname.startsWith('/_next/') ||
    pathname.startsWith('/static/') ||
    pathname.match(/\.(png|jpg|jpeg|gif|webp|svg|ico|css|js|woff2|woff|ttf)$/)
  ) {
    return NextResponse.next()
  }

  if (!token) {
    return NextResponse.redirect(new URL('/login', req.url))
  }

  return NextResponse.next()
}

export const config = {
  matcher: [
    '/',
  ],
}
```

## 7. Install Zod
[Instalasi Zod](https://zod.dev/)
```
npm install zod
```

## 8. Install React Hook Form
[Instalasi React Hook Form](https://react-hook-form.com/get-started)
```
npm install react-hook-form
```

## 9. Install Hook Form Resolver
[Instalasi Hook Form Resolver](https://www.npmjs.com/package/@hookform/resolvers)
```
npm install @hookform/resolvers
```

## 10. Sesuaikan component ```LoginForm.tsx``` pada folder ```(auth)/login/components```
```
import { z } from 'zod'

const loginSchema = z.object({
    email: z.string().email('Invalid email address'),
    password: z.string().min(6, 'Password must be at least 6 characters'),
})

type LoginSchema = z.infer<typeof loginSchema>
```

Tambahkan parameter pada function ```LoginForm.tsx```
```
{
  ...props
}: React.ComponentPropsWithoutRef<'form'>
```

Tambahkan form handler dalam function ```LoginForm.tsx```
```
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'

const {
  register,
  handleSubmit,
  formState: { errors },
} = useForm<LoginSchema>({
  resolver: zodResolver(loginSchema),
})
```

```
'use client'
import { useRouter } from 'next/navigation'
import { useState } from 'react'

const router = useRouter()
const [loading, setLoading] = useState(false)
const [loginError, setLoginError] = useState<string | null>(null)
```

Tambahkan function submit form
```
import { signIn } from 'next-auth/react'

const onSubmit = async (data: LoginSchema) => {
  setLoading(true)
  setLoginError(null)

  const res = await signIn('credentials', {
    redirect: false,
    email: data.email,
    password: data.password,
    callbackUrl: '/',
  })

  setLoading(false)

  if (res?.ok && !res.error) {
    router.push('/')
  } else {
    setLoginError('Invalid email or password')
  }
}
```

Lengkapi component form sebagai berikut:
```
<form
  onSubmit={handleSubmit(onSubmit)}
  className="flex flex-col gap-6 w-full"
  {...props}
>

....

<Input
  id='email'
  type='email'
  placeholder='helloworld@example.com'
  {...register('email')}
/>
{errors.email && (
  <p className='text-sm text-red-500'>{errors.email.message}</p>
)}

....

<Input 
  id='password' 
  type='password' 
  placeholder='********'
  {...register('password')} 
/>
{errors.password && (
  <p className='text-sm text-red-500'>{errors.password.message}</p>
)}

....

<Button
  type='submit'
  className='w-full'
  disabled={loading}
>
  {loading ? 'Logging in...' : 'Login'}
</Button>
{loginError && (
    <p className='text-sm text-red-500 text-center'>{loginError}</p>
)}
```

## 11. Lengkapi file ```page.tsx``` pada folder ```src/app```
```
'use client'

import { Button } from "@/components/ui/button";
import { useSession, signOut } from "next-auth/react";

export default function Home() {
  const { data: session } = useSession()

  ....

  <Button
    variant="destructive"
    size="lg"
    onClick={() => {
      if (session) {
        signOut()
      }
    }}
  >
    Logout
  </Button>
}
```

## 12. Tambahkan file ```provider.tsx``` pada folder ```src/app```
```
'use client'

import { SessionProvider } from 'next-auth/react'

export default function Providers({ children }: { children: React.ReactNode }) {
  return (
    <SessionProvider>{children}</SessionProvider>
  )
}
```

## 13. Sesuaikan file ```layout.tsx``` pada folder ```src/app```
```
import Providers from "./providers";

<html lang="en">
  <body
    className={`${geistSans.variable} ${geistMono.variable} antialiased`}
  >
    <Providers>{children}</Providers>
  </body>
</html>
```
