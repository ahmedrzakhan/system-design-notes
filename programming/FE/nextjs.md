# Next.js Interview Questions & Answers

## 1. What is Next.js and why use it?

**Q: Explain Next.js**

A: Next.js is a React framework for production that provides hybrid static & server-side rendering, TypeScript support, and built-in optimization.

**Key features**:

- Server-side rendering (SSR)
- Static site generation (SSG)
- Incremental Static Regeneration (ISR)
- API routes
- Image optimization
- Font optimization
- File-based routing
- Built-in CSS and Sass support
- TypeScript support
- Fast refresh (instant feedback)
- Dynamic imports
- Automatic code splitting
- Production ready

**Why use Next.js**:

- Better performance (optimized bundle)
- SEO friendly (SSR)
- Static generation for speed
- Full-stack with API routes
- Great developer experience
- Excellent community
- Backed by Vercel
- Scales well

**vs Create React App**: CRA is client-side only. Next.js has SSR, SSG, API routes, better performance.

**vs Gatsby**: Gatsby for static sites. Next.js for dynamic, hybrid rendering.

---

## 2. What are pages and routing?

**Q: File-based routing in Next.js**

A: Pages automatically become routes based on file structure.

```
pages/
├── index.js           → /
├── about.js           → /about
├── contact.js         → /contact
├── blog/
│   ├── index.js       → /blog
│   └── [slug].js      → /blog/post-name
├── api/
│   ├── users.js       → /api/users
│   └── users/
│       └── [id].js    → /api/users/1

app/ (App Router - Next.js 13+)
├── layout.js
├── page.js            → /
├── about/
│   └── page.js        → /about
├── blog/
│   ├── layout.js
│   ├── page.js        → /blog
│   └── [slug]/
│       └── page.js    → /blog/post-name
```

**Dynamic routes**:

```javascript
// pages/blog/[slug].js
import { useRouter } from 'next/router';

export default function BlogPost() {
    const router = useRouter();
    const { slug } = router.query;

    return <h1>Blog: {slug}</h1>;
}

// pages/posts/[id]/comments/[commentId].js
export default function Comment() {
    const router = useRouter();
    const { id, commentId } = router.query;

    return <h1>Post {id}, Comment {commentId}</h1>;
}
```

**Catch-all routes**:

```javascript
// pages/docs/[...params].js
// Matches /docs, /docs/a, /docs/a/b, etc.

export default function Docs() {
  const router = useRouter();
  const { params } = router.query; // ['a', 'b', 'c']

  return <h1>Docs: {params?.join("/")}</h1>;
}
```

**Optional catch-all**:

```javascript
// pages/[[...params]].js
// Matches all routes including root

export default function AllRoutes() {
  const router = useRouter();
  const { params } = router.query; // undefined at root

  return <h1>Route</h1>;
}
```

---

## 3. What is SSR (Server-Side Rendering)?

**Q: getServerSideProps**

A: Render page on server on each request.

```javascript
// pages/posts/[id].js
export default function Post({ post }) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

// Runs on server, before rendering
export async function getServerSideProps(context) {
  const { id } = context.params;

  const res = await fetch(`https://api.example.com/posts/${id}`);
  const post = await res.json();

  if (!post) {
    return {
      notFound: true, // 404 page
    };
  }

  return {
    props: {
      post,
    },
    revalidate: 60, // ISR: revalidate after 60 seconds
  };
}
```

**Context object**:

```javascript
export async function getServerSideProps(context) {
  const {
    params, // Route parameters
    query, // Query string
    req, // Request object
    res, // Response object
    resolvedUrl, // Full URL
  } = context;

  return { props: {} };
}
```

**Redirect**:

```javascript
export async function getServerSideProps() {
  return {
    redirect: {
      destination: "/about",
      permanent: false, // 307 temporary, true for 308 permanent
    },
  };
}
```

---

## 4. What is SSG (Static Site Generation)?

**Q: getStaticProps and getStaticPaths**

A: Pre-build static pages at build time.

```javascript
// pages/posts/[id].js
export default function Post({ post }) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

// Runs at build time
export async function getStaticProps(context) {
  const { id } = context.params;

  const res = await fetch(`https://api.example.com/posts/${id}`);
  const post = await res.json();

  return {
    props: { post },
    revalidate: 60, // ISR: regenerate after 60 seconds
  };
}

// Define which routes to pre-build
export async function getStaticPaths() {
  const res = await fetch("https://api.example.com/posts");
  const posts = await res.json();

  const paths = posts.map((post) => ({
    params: { id: post.id.toString() },
  }));

  return {
    paths,
    fallback: "blocking", // 'blocking' or true or false
  };
}
```

**fallback options**:

- `false` — Return 404 for unknown routes
- `true` — Show loading state, then regenerate
- `'blocking'` — Wait for page to generate (like SSR)

**Incremental Static Regeneration (ISR)**:

```javascript
export async function getStaticProps() {
  return {
    props: { data },
    revalidate: 3600, // Regenerate every hour
  };
}
```

---

## 5. What is the App Router?

**Q: Next.js 13+ App Router**

A: New routing system with layouts and server components.

```
app/
├── layout.js          // Root layout
├── page.js            // Home page
├── about/
│   ├── layout.js      // About layout (extends root)
│   └── page.js        // /about
├── blog/
│   ├── layout.js
│   ├── page.js        // /blog
│   ├── [slug]/
│   │   └── page.js    // /blog/[slug]
│   └── error.js       // Error boundary
├── api/
│   └── users/
│       └── route.js   // /api/users
└── not-found.js       // 404 page
```

**Server components** (default):

```javascript
// app/page.js
export default async function Home() {
  const res = await fetch("https://api.example.com/data");
  const data = await res.json();

  return <div>{data.title}</div>;
}
```

**Client components**:

```javascript
"use client"; // Mark as client component

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Layouts** (shared UI):

```javascript
// app/layout.js
export const metadata = {
    title: 'My Site',
    description: 'Description'
};

export default function RootLayout({ children }) {
    return (
        <html>
            <body>
                <header>Header</header>
                {children}
                <footer>Footer</footer>
            </body>
        </html>
    );
}

// app/blog/layout.js
export default function BlogLayout({ children }) {
    return (
        <div>
            <aside>Blog Sidebar</aside>
            <main>{children}</main>
        </div>
    );
}
```

---

## 6. What are API routes?

**Q: Create backend endpoints**

A: Write API endpoints as JavaScript functions.

```javascript
// pages/api/users.js
export default function handler(req, res) {
    if (req.method === 'GET') {
        res.status(200).json({ users: [] });
    } else if (req.method === 'POST') {
        const { name, email } = req.body;
        res.status(201).json({ id: 1, name, email });
    } else {
        res.status(405).end();  // Method not allowed
    }
}

// pages/api/users/[id].js
export default function handler(req, res) {
    const { id } = req.query;

    if (req.method === 'GET') {
        res.status(200).json({ id, name: 'John' });
    } else if (req.method === 'PUT') {
        res.status(200).json({ id, ...req.body });
    } else if (req.method === 'DELETE') {
        res.status(204).end();
    }
}

// App Router: app/api/users/route.js
export async function GET(request) {
    return Response.json({ users: [] });
}

export async function POST(request) {
    const body = await request.json();
    return Response.json({ success: true }, { status: 201 });
}
```

**Middleware** (for auth, logging):

```javascript
// middleware.js (in root directory)
import { NextResponse } from "next/server";

export function middleware(request) {
  const token = request.headers.get("authorization");

  if (!token && request.nextUrl.pathname.startsWith("/api/protected")) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/api/:path*"],
};
```

---

## 7. What are data fetching patterns?

**Q: How to fetch data in Next.js**

A: Different patterns for SSR, SSG, and client-side.

**Server-side rendering (SSR)**:

```javascript
// pages/posts/[id].js
export default function Post({ post }) {
  return <h1>{post.title}</h1>;
}

export async function getServerSideProps({ params }) {
  const res = await fetch(`/api/posts/${params.id}`);
  const post = await res.json();

  return { props: { post } };
}
```

**Static generation (SSG)**:

```javascript
export default function Post({ post }) {
  return <h1>{post.title}</h1>;
}

export async function getStaticProps({ params }) {
  const res = await fetch(`/api/posts/${params.id}`);
  const post = await res.json();

  return {
    props: { post },
    revalidate: 3600,
  };
}

export async function getStaticPaths() {
  const res = await fetch("/api/posts");
  const posts = await res.json();

  return {
    paths: posts.map((p) => ({ params: { id: p.id } })),
    fallback: "blocking",
  };
}
```

**Client-side data fetching**:

```javascript
"use client";

import { useEffect, useState } from "react";

export default function Posts() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("/api/posts")
      .then((res) => res.json())
      .then((data) => {
        setPosts(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading...</p>;
  return posts.map((p) => <h1 key={p.id}>{p.title}</h1>);
}
```

**SWR (data fetching library)**:

```javascript
"use client";

import useSWR from "swr";

export default function Posts() {
  const { data, error, isLoading } = useSWR("/api/posts");

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return data.map((p) => <h1 key={p.id}>{p.title}</h1>);
}
```

---

## 8. What is image optimization?

**Q: Next.js Image component**

A: Automatic image optimization.

```javascript
import Image from 'next/image';

// Static import
import myImage from '@/public/images/my-image.png';

export default function Home() {
    return (
        <Image
            src={myImage}
            alt="Description"
            width={800}
            height={600}
        />
    );
}

// Dynamic image
export default function Home() {
    return (
        <Image
            src="https://example.com/image.png"
            alt="Description"
            width={800}
            height={600}
            priority           // Load eagerly
            placeholder="blur"  // Show blur while loading
            blurDataURL="..."   // Custom blur
        />
    );
}

// Responsive
export default function Home() {
    return (
        <Image
            src="/image.png"
            alt="Description"
            fill              // Fill parent container
            objectFit="cover"
            sizes="(max-width: 768px) 100vw, 50vw"
        />
    );
}
```

**Benefits**:

- Automatic format conversion (WebP)
- Responsive images
- Lazy loading by default
- Prevents layout shift
- On-demand generation and caching

---

## 9. What is styling?

**Q: Style components in Next.js**

A: Multiple styling options.

**CSS Modules**:

```javascript
// styles/Home.module.css
.container {
    max-width: 1200px;
    margin: 0 auto;
}

.title {
    font-size: 32px;
    font-weight: bold;
}

// pages/index.js
import styles from '@/styles/Home.module.css';

export default function Home() {
    return (
        <div className={styles.container}>
            <h1 className={styles.title}>Hello</h1>
        </div>
    );
}
```

**Tailwind CSS**:

```javascript
// tailwind.config.js
module.exports = {
  content: ["./pages/**/*.js", "./components/**/*.js"],
  theme: {},
  plugins: [],
};

// pages/index.js
export default function Home() {
  return (
    <div className="max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold">Hello</h1>
    </div>
  );
}
```

**CSS-in-JS (styled-components)**:

```javascript
"use client";

import styled from "styled-components";

const Container = styled.div`
  max-width: 1200px;
  margin: 0 auto;
`;

const Title = styled.h1`
  font-size: 32px;
  font-weight: bold;
  color: ${(props) => props.color || "black"};
`;

export default function Home() {
  return (
    <Container>
      <Title color="blue">Hello</Title>
    </Container>
  );
}
```

**Global styles**:

```javascript
// app/globals.css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto;
}

// app/layout.js
import './globals.css';

export default function RootLayout({ children }) {
    return <html><body>{children}</body></html>;
}
```

---

## 10. What is middleware?

**Q: Middleware in Next.js**

A: Run code before request is processed.

```javascript
// middleware.ts (or .js)
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Check authentication
  const token = request.cookies.get("token");

  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  // Add header
  const response = NextResponse.next();
  response.headers.set("X-Custom-Header", "value");

  return response;
}

export const config = {
  matcher: ["/dashboard/:path*", "/api/:path*"],
};

// app/middleware.ts (App Router)
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Logging
  console.log(`${request.method} ${request.nextUrl.pathname}`);

  return NextResponse.next();
}

export const config = {
  matcher: ["/((?!_next/static|_next/image|favicon.ico).*)"],
};
```

---

## 11. What are dynamic imports?

**Q: Code splitting and lazy loading**

A: Load components on demand.

```javascript
import dynamic from 'next/dynamic';

// Lazy load component
const HeavyComponent = dynamic(() => import('@/components/Heavy'), {
    loading: () => <p>Loading...</p>,
    ssr: false  // Don't render on server
});

export default function Home() {
    return (
        <div>
            <HeavyComponent />
        </div>
    );
}

// Named export
const { Foo } = dynamic(() =>
    import('@/components/Foo').then(mod => ({ Foo: mod.Foo }))
);

export default function Home() {
    return <Foo />;
}
```

---

## 12. What is environment variables?

**Q: Configure app with .env**

A:

```bash
# .env.local
API_URL=http://localhost:3000
PUBLIC_ENV_VAR=visible_to_browser
SECRET_KEY=only_on_server

# .env.production
API_URL=https://api.example.com
```

```javascript
// On server (pages, API routes, getStaticProps, getServerSideProps)
const apiUrl = process.env.API_URL;
const secret = process.env.SECRET_KEY;

// On client (exposed)
const publicVar = process.env.NEXT_PUBLIC_ENV_VAR;

// App Router (server component)
export default async function Home() {
  const data = await fetch(process.env.API_URL + "/data");
  return <div>{JSON.stringify(data)}</div>;
}

// next.config.js
module.exports = {
  env: {
    CUSTOM_VAR: process.env.CUSTOM_VAR,
  },
};
```

---

## 13. What is performance optimization?

**Q: Improve performance**

A:

**Code splitting**:

```javascript
// Automatic: Each page is automatically code split

// Manual: Dynamic imports
import dynamic from "next/dynamic";
const Component = dynamic(() => import("@/components/Heavy"));
```

**Font optimization**:

```javascript
// app/layout.js
import { Roboto } from "next/font/google";

const roboto = Roboto({ weight: "400" });

export default function RootLayout({ children }) {
  return (
    <html className={roboto.className}>
      <body>{children}</body>
    </html>
  );
}
```

**Script optimization**:

```javascript
import Script from "next/script";

export default function Home() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        strategy="afterInteractive" // Load after interactive
      />
      <Script
        src="https://analytics.js"
        strategy="lazyOnload" // Load when idle
      />
    </>
  );
}
```

**Image optimization** — Already covered.

**Bundle analysis**:

```bash
npm install --save-dev @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
    enabled: process.env.ANALYZE === 'true'
});

module.exports = withBundleAnalyzer({});

# Analyze
ANALYZE=true npm run build
```

---

## 14. What is internationalization (i18n)?

**Q: Multi-language support**

A:

**Using next-i18next**:

```bash
npm install next-i18next i18next
```

```javascript
// next-i18next.config.js
const path = require('path');

module.exports = {
    i18n: {
        defaultLocale: 'en',
        locales: ['en', 'es', 'fr']
    },
    localePath: path.resolve('./public/locales')
};

// next.config.js
const { i18n } = require('./next-i18next.config');

module.exports = {
    i18n
};

// pages/_app.js
import { appWithTranslation } from 'next-i18next';

function MyApp({ Component, pageProps }) {
    return <Component {...pageProps} />;
}

export default appWithTranslation(MyApp);

// pages/index.js
import { useTranslation } from 'next-i18next';
import { serverSideTranslations } from 'next-i18next/serverSideTranslations';

export default function Home() {
    const { t } = useTranslation('common');

    return <h1>{t('hello')}</h1>;
}

export async function getStaticProps({ locale }) {
    return {
        props: {
            ...(await serverSideTranslations(locale, ['common']))
        }
    };
}
```

---

## 15. What is authentication?

**Q: Implement login/auth**

A:

**NextAuth.js**:

```bash
npm install next-auth
```

```javascript
// pages/api/auth/[...nextauth].js
import NextAuth from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import GitHubProvider from 'next-auth/providers/github';

export default NextAuth({
    providers: [
        GitHubProvider({
            clientId: process.env.GITHUB_ID,
            clientSecret: process.env.GITHUB_SECRET
        }),
        CredentialsProvider({
            async authorize(credentials) {
                const user = await verifyCredentials(credentials);
                if (user) return user;
                return null;
            }
        })
    ],
    pages: {
        signIn: '/auth/signin',
        error: '/auth/error'
    },
    callbacks: {
        async jwt({ token, user }) {
            if (user) token.role = user.role;
            return token;
        },
        async session({ session, token }) {
            session.user.role = token.role;
            return session;
        }
    }
});

// pages/protected.js
import { useSession } from 'next-auth/react';
import { useRouter } from 'next/router';

export default function Protected() {
    const { data: session, status } = useSession();
    const router = useRouter();

    if (status === 'loading') return <p>Loading...</p>;
    if (!session) {
        router.push('/auth/signin');
        return null;
    }

    return <h1>Welcome {session.user.name}</h1>;
}

// Login component
'use client'

import { signIn } from 'next-auth/react';

export default function Login() {
    return (
        <>
            <button onClick={() => signIn('github')}>
                Sign in with GitHub
            </button>
            <button onClick={() => signIn('credentials')}>
                Sign in with Credentials
            </button>
        </>
    );
}
```

---

## 16. What is deployment?

**Q: Deploy Next.js app**

A: Multiple hosting options.

**Vercel (easiest)**:

```bash
npm install -g vercel
vercel
```

```javascript
// vercel.json
{
    "buildCommand": "npm run build",
    "devCommand": "npm run dev",
    "installCommand": "npm install"
}
```

**Docker**:

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

**Docker Compose**:

```yaml
version: "3"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://user:pass@db:5432/dbname
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
```

**AWS/Google Cloud/Azure**: Deploy containers or use serverless.

---

## 17. What is testing?

**Q: Test Next.js apps**

A:

```javascript
// jest.config.js
const nextJest = require("next/jest");

const createJestConfig = nextJest({
  dir: "./",
});

const customJestConfig = {
  moduleNameMapper: {
    "^@/(.*)$": "<rootDir>/$1",
  },
  testEnvironment: "jest-environment-jsdom",
};

module.exports = createJestConfig(customJestConfig);

// __tests__/page.test.js
import { render, screen } from "@testing-library/react";
import Home from "@/pages/index";

describe("Home page", () => {
  it("renders heading", () => {
    render(<Home />);
    const heading = screen.getByRole("heading", { name: /welcome/i });
    expect(heading).toBeInTheDocument();
  });
});

// E2E testing with Playwright
import { test, expect } from "@playwright/test";

test("homepage has title", async ({ page }) => {
  await page.goto("http://localhost:3000");
  await expect(page).toHaveTitle(/My Site/);
});

test("can navigate to about", async ({ page }) => {
  await page.goto("http://localhost:3000");
  await page.click('a[href="/about"]');
  await expect(page).toHaveURL(/about/);
});
```

---

## 18. What is SEO and metadata?

**Q: Optimize for search engines**

A:

**Next.js 13+ (App Router)**:

```javascript
// app/layout.js
export const metadata = {
  title: "My Site",
  description: "Description of my site",
  openGraph: {
    title: "My Site",
    description: "Description",
    url: "https://mysite.com",
    siteName: "My Site",
    images: [{ url: "https://mysite.com/og.png" }],
    type: "website",
  },
};

// app/blog/[slug]/page.js
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [{ url: post.image }],
    },
  };
}
```

**Pages Router (older)**:

```javascript
import Head from "next/head";

export default function Home() {
  return (
    <>
      <Head>
        <title>My Site</title>
        <meta name="description" content="Description" />
        <meta property="og:title" content="My Site" />
        <meta property="og:description" content="Description" />
      </Head>
      <h1>Welcome</h1>
    </>
  );
}
```

**Sitemap**:

```javascript
// pages/sitemap.xml.js
export default function Sitemap() {}

export async function getServerSideProps({ res }) {
  const posts = await getPosts();

  const xml = `<?xml version="1.0" encoding="UTF-8"?>
        <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
            <url>
                <loc>https://mysite.com</loc>
            </url>
            ${posts
              .map(
                (p) => `
                <url>
                    <loc>https://mysite.com/blog/${p.slug}</loc>
                </url>
            `
              )
              .join("")}
        </urlset>`;

  res.setHeader("Content-Type", "text/xml");
  res.write(xml);
  res.end();
}
```

---

## 19. What is common patterns?

**Q: Best practices**

A:

**Folder structure**:

```
project/
├── pages/
├── app/
├── components/
│   ├── common/
│   ├── layouts/
│   └── features/
├── lib/
│   ├── api.js
│   ├── db.js
│   └── utils.js
├── styles/
├── public/
├── __tests__/
├── .env.local
└── next.config.js
```

**Reusable components**:

```javascript
// components/Button.js
export default function Button({ children, variant = 'primary', ...props }) {
    const className = `btn btn-${variant}`;
    return <button className={className} {...props}>{children}</button>;
}

// components/Layout.js
export default function Layout({ children }) {
    return (
        <>
            <Header />
            <main>{children}</main>
            <Footer />
        </>
    );
}

// pages/index.js
import Layout from '@/components/Layout';

export default function Home() {
    return (
        <Layout>
            <h1>Home</h1>
        </Layout>
    );
}
```

**API helper**:

```javascript
// lib/api.js
export async function fetchData(endpoint) {
  const res = await fetch(`${process.env.API_URL}${endpoint}`);
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}

export async function postData(endpoint, data) {
  const res = await fetch(`${process.env.API_URL}${endpoint}`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}
```

---

## 20. What is real-world example?

**Q: Complete blog application**

A:

```javascript
// lib/posts.js
import fs from 'fs/promises';
import path from 'path';

const postsDir = path.join(process.cwd(), 'posts');

export async function getPosts() {
    const files = await fs.readdir(postsDir);
    const posts = await Promise.all(
        files
            .filter(f => f.endsWith('.md'))
            .map(async file => {
                const content = await fs.readFile(path.join(postsDir, file), 'utf8');
                const [metadata, body] = content.split('---');
                const slug = file.replace('.md', '');

                return {
                    slug,
                    ...parseMetadata(metadata),
                    body
                };
            })
    );

    return posts.sort((a, b) => new Date(b.date) - new Date(a.date));
}

export async function getPost(slug) {
    const post = await getPosts();
    return posts.find(p => p.slug === slug);
}

function parseMetadata(metadata) {
    const lines = metadata.trim().split('\n');
    const obj = {};
    lines.forEach(line => {
        const [key, ...value] = line.split(':');
        obj[key.trim()] = value.join(':').trim();
    });
    return obj;
}

// pages/blog/index.js
import Link from 'next/link';
import { getPosts } from '@/lib/posts';

export default function Blog({ posts }) {
    return (
        <div>
            <h1>Blog</h1>
            <ul>
                {posts.map(post => (
                    <li key={post.slug}>
                        <Link href={`/blog/${post.slug}`}>
                            {post.title}
                        </Link>
                        <p>{post.excerpt}</p>
                        <small>{post.date}</small>
                    </li>
                ))}
            </ul>
        </div>
    );
}

export async function getStaticProps() {
    const posts = await getPosts();
    return {
        props: { posts },
        revalidate: 3600
    };
}

// pages/blog/[slug].js
import { getPosts, getPost } from '@/lib/posts';
import markdownToHtml from '@/lib/markdown';

export default function BlogPost({ post, html }) {
    return (
        <article>
            <h1>{post.title}</h1>
            <p>{post.date}</p>
            <div dangerouslySetInnerHTML={{ __html: html }} />
        </article>
    );
}

export async function getStaticProps({ params }) {
    const post = await getPost(params.slug);
    const html = await markdownToHtml(post.body);

    return {
        props: { post, html },
        revalidate: 3600
    };
}

export async function getStaticPaths() {
    const posts = await getPosts();

    return {
        paths: posts.map(p => ({ params: { slug: p.slug } })),
        fallback: 'blocking'
    };
}
```

---

## Next.js Interview Tips

1. **Routing** — File-based routing system
2. **SSR vs SSG** — Know when to use each
3. **App Router vs Pages Router** — Understand both
4. **Data fetching** — getStaticProps, getServerSideProps
5. **Image optimization** — Next.js Image component
6. **API routes** — Backend with /api
7. **Middleware** — Request handling
8. **Deployment** — Vercel, Docker, AWS
9. **Performance** — Code splitting, lazy loading
10. **SEO** — Metadata, Open Graph
11. **Authentication** — NextAuth.js
12. **Testing** — Jest, React Testing Library, Playwright
13. **Environment variables** — .env configuration
14. **Styling** — CSS Modules, Tailwind, styled-components
15. **Production code** — Real project experience
