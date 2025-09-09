# Next.js Cheat Sheet

A comprehensive reference for Next.js - a React framework for building full-stack web applications with server-side rendering, static generation, and more.

---

## Table of Contents
- [Installation and Setup](#installation-and-setup)
- [Project Structure](#project-structure)
- [Pages and Routing](#pages-and-routing)
- [App Router (Next.js 13+)](#app-router-nextjs-13)
- [Data Fetching](#data-fetching)
- [API Routes](#api-routes)
- [Styling](#styling)
- [Images and Optimization](#images-and-optimization)
- [Deployment](#deployment)
- [Performance](#performance)
- [Best Practices](#best-practices)

---

## Installation and Setup

### Create Next.js App
```bash
# Create new Next.js app
npx create-next-app@latest my-app
cd my-app

# With TypeScript
npx create-next-app@latest my-app --typescript

# With App Router (recommended)
npx create-next-app@latest my-app --app

# Development server
npm run dev
# or
yarn dev

# Production build
npm run build
npm start
```

### Manual Installation
```bash
npm install next react react-dom

# Add scripts to package.json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  }
}
```

## Project Structure

### Pages Router Structure
```
project/
├── pages/
│   ├── api/
│   │   └── hello.js
│   ├── _app.js
│   ├── _document.js
│   ├── index.js
│   └── about.js
├── public/
├── styles/
├── components/
├── lib/
├── utils/
└── next.config.js
```

### App Router Structure (Next.js 13+)
```
project/
├── app/
│   ├── globals.css
│   ├── layout.js
│   ├── page.js
│   ├── loading.js
│   ├── error.js
│   ├── not-found.js
│   ├── about/
│   │   └── page.js
│   └── api/
│       └── hello/
│           └── route.js
├── public/
├── components/
└── next.config.js
```

## Pages and Routing

### Pages Router (Legacy)
```jsx
// pages/index.js - Home page
export default function Home() {
  return (
    <div>
      <h1>Welcome to Next.js!</h1>
    </div>
  );
}

// pages/about.js - About page
export default function About() {
  return (
    <div>
      <h1>About Us</h1>
    </div>
  );
}

// pages/blog/[slug].js - Dynamic route
import { useRouter } from 'next/router';

export default function BlogPost() {
  const router = useRouter();
  const { slug } = router.query;

  return <h1>Blog Post: {slug}</h1>;
}

// pages/blog/[...params].js - Catch-all route
export default function BlogParams() {
  const router = useRouter();
  const { params } = router.query;

  return <h1>Params: {params?.join('/')}</h1>;
}
```

### Navigation
```jsx
import Link from 'next/link';
import { useRouter } from 'next/router';

export default function Navigation() {
  const router = useRouter();

  const handleNavigation = () => {
    router.push('/about');
    // router.replace('/about'); // No history entry
    // router.back(); // Go back
  };

  return (
    <nav>
      <Link href="/">
        <a className={router.pathname === '/' ? 'active' : ''}>Home</a>
      </Link>
      
      <Link href="/about">
        <a>About</a>
      </Link>
      
      <Link href="/blog/my-post">
        <a>Blog Post</a>
      </Link>
      
      {/* Programmatic navigation */}
      <button onClick={handleNavigation}>Go to About</button>
    </nav>
  );
}
```

## App Router (Next.js 13+)

### Basic App Router Setup
```jsx
// app/layout.js - Root layout
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <nav>
          <Link href="/">Home</Link>
          <Link href="/about">About</Link>
        </nav>
        <main>{children}</main>
      </body>
    </html>
  );
}

// app/page.js - Home page
export default function HomePage() {
  return <h1>Home Page</h1>;
}

// app/about/page.js - About page
export default function AboutPage() {
  return <h1>About Page</h1>;
}

// app/blog/[slug]/page.js - Dynamic route
export default function BlogPost({ params }) {
  return <h1>Blog: {params.slug}</h1>;
}
```

### App Router Features
```jsx
// app/loading.js - Loading UI
export default function Loading() {
  return <div>Loading...</div>;
}

// app/error.js - Error UI
'use client';

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}

// app/not-found.js - 404 page
export default function NotFound() {
  return <h2>Page Not Found</h2>;
}

// app/dashboard/layout.js - Nested layout
export default function DashboardLayout({ children }) {
  return (
    <div>
      <aside>Dashboard Navigation</aside>
      <main>{children}</main>
    </div>
  );
}
```

## Data Fetching

### Static Site Generation (SSG)
```jsx
// pages/blog/index.js
export default function Blog({ posts }) {
  return (
    <div>
      {posts.map(post => (
        <div key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </div>
      ))}
    </div>
  );
}

// Fetch data at build time
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    props: {
      posts,
    },
    // Revalidate every 60 seconds
    revalidate: 60,
  };
}
```

### Static Generation with Dynamic Routes
```jsx
// pages/blog/[id].js
export default function Post({ post }) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

// Generate static paths
export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  const paths = posts.map(post => ({
    params: { id: post.id.toString() }
  }));

  return {
    paths,
    fallback: false, // 404 for unknown paths
    // fallback: true, // Generate on demand
    // fallback: 'blocking', // SSR for unknown paths
  };
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`);
  const post = await res.json();

  return {
    props: { post },
    revalidate: 1, // ISR - revalidate every second
  };
}
```

### Server-Side Rendering (SSR)
```jsx
// pages/profile.js
export default function Profile({ user }) {
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <p>Email: {user.email}</p>
    </div>
  );
}

// Fetch data on each request
export async function getServerSideProps({ req, res, params, query }) {
  // Access cookies
  const { cookies } = req;
  const token = cookies.auth_token;

  if (!token) {
    return {
      redirect: {
        destination: '/login',
        permanent: false,
      },
    };
  }

  // Fetch user data
  const userRes = await fetch('https://api.example.com/user', {
    headers: {
      Authorization: `Bearer ${token}`,
    },
  });

  if (!userRes.ok) {
    return {
      notFound: true,
    };
  }

  const user = await userRes.json();

  return {
    props: { user },
  };
}
```

### App Router Data Fetching
```jsx
// app/posts/page.js - Server Component
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    // Cache control
    cache: 'force-cache', // Cache indefinitely
    // cache: 'no-store', // Disable caching
    // next: { revalidate: 3600 }, // Revalidate every hour
  });

  if (!res.ok) {
    throw new Error('Failed to fetch posts');
  }

  return res.json();
}

export default async function PostsPage() {
  const posts = await getPosts();

  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

// app/posts/[id]/page.js - Dynamic params
async function getPost(id) {
  const res = await fetch(`https://api.example.com/posts/${id}`);
  return res.json();
}

export default async function PostPage({ params }) {
  const post = await getPost(params.id);

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

## API Routes

### Pages Router API Routes
```javascript
// pages/api/hello.js
export default function handler(req, res) {
  if (req.method === 'GET') {
    res.status(200).json({ message: 'Hello World' });
  } else {
    res.setHeader('Allow', ['GET']);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}

// pages/api/users/[id].js - Dynamic API route
export default async function handler(req, res) {
  const { id } = req.query;
  const { method } = req;

  switch (method) {
    case 'GET':
      try {
        const user = await getUserById(id);
        res.status(200).json(user);
      } catch (error) {
        res.status(404).json({ error: 'User not found' });
      }
      break;

    case 'PUT':
      try {
        const updatedUser = await updateUser(id, req.body);
        res.status(200).json(updatedUser);
      } catch (error) {
        res.status(400).json({ error: 'Failed to update user' });
      }
      break;

    case 'DELETE':
      try {
        await deleteUser(id);
        res.status(204).end();
      } catch (error) {
        res.status(400).json({ error: 'Failed to delete user' });
      }
      break;

    default:
      res.setHeader('Allow', ['GET', 'PUT', 'DELETE']);
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}
```

### App Router API Routes
```javascript
// app/api/hello/route.js
export async function GET(request) {
  return Response.json({ message: 'Hello World' });
}

export async function POST(request) {
  const body = await request.json();
  
  return Response.json(
    { message: 'Created successfully', data: body },
    { status: 201 }
  );
}

// app/api/users/[id]/route.js - Dynamic API route
export async function GET(request, { params }) {
  const { id } = params;
  
  try {
    const user = await getUserById(id);
    return Response.json(user);
  } catch (error) {
    return Response.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }
}

export async function PUT(request, { params }) {
  const { id } = params;
  const body = await request.json();
  
  try {
    const updatedUser = await updateUser(id, body);
    return Response.json(updatedUser);
  } catch (error) {
    return Response.json(
      { error: 'Failed to update user' },
      { status: 400 }
    );
  }
}

export async function DELETE(request, { params }) {
  const { id } = params;
  
  try {
    await deleteUser(id);
    return new Response(null, { status: 204 });
  } catch (error) {
    return Response.json(
      { error: 'Failed to delete user' },
      { status: 400 }
    );
  }
}
```

### Middleware
```javascript
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  // Authentication check
  const token = request.cookies.get('auth-token');
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  // Add custom headers
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'custom-value');
  
  return response;
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

## Styling

### CSS Modules
```css
/* styles/Home.module.css */
.container {
  padding: 0 2rem;
}

.main {
  min-height: 100vh;
  padding: 4rem 0;
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
```

```jsx
// pages/index.js
import styles from '../styles/Home.module.css';

export default function Home() {
  return (
    <div className={styles.container}>
      <main className={styles.main}>
        <h1>Welcome to Next.js!</h1>
      </main>
    </div>
  );
}
```

### Styled JSX
```jsx
export default function StyledComponent() {
  return (
    <div>
      <h1>Styled with JSX</h1>
      <style jsx>{`
        h1 {
          color: blue;
          font-size: 2rem;
        }
        div {
          padding: 1rem;
          background: #f0f0f0;
        }
      `}</style>
    </div>
  );
}
```

### Tailwind CSS
```bash
# Install Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```javascript
// tailwind.config.js
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
    './app/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

```jsx
export default function TailwindComponent() {
  return (
    <div className="container mx-auto px-4">
      <h1 className="text-3xl font-bold text-blue-600">
        Styled with Tailwind
      </h1>
      <p className="mt-4 text-gray-700">
        This is a paragraph with Tailwind classes.
      </p>
    </div>
  );
}
```

## Images and Optimization

### Next.js Image Component
```jsx
import Image from 'next/image';

export default function ImageExample() {
  return (
    <div>
      {/* Static image import */}
      <Image
        src="/hero-image.jpg"
        alt="Hero"
        width={800}
        height={400}
        priority // Load immediately
      />
      
      {/* Dynamic image with placeholder */}
      <Image
        src={`https://api.example.com/images/${imageId}`}
        alt="Dynamic image"
        width={400}
        height={300}
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQ..."
      />
      
      {/* Responsive image */}
      <div style={{ position: 'relative', width: '100%', height: '400px' }}>
        <Image
          src="/responsive-image.jpg"
          alt="Responsive"
          fill
          style={{ objectFit: 'cover' }}
        />
      </div>
    </div>
  );
}
```

### Font Optimization
```jsx
// app/layout.js or pages/_app.js
import { Inter, Roboto_Mono } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
});

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

## Deployment

### Vercel Deployment
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy to Vercel
vercel

# Production deployment
vercel --prod
```

### Docker Deployment
```dockerfile
# Dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --only=production

FROM node:18-alpine AS builder
WORKDIR /app
COPY . .
COPY --from=deps /app/node_modules ./node_modules
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV production

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["npm", "start"]
```

### Static Export
```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
  distDir: 'dist',
  images: {
    unoptimized: true
  }
};

module.exports = nextConfig;
```

```bash
# Build static export
npm run build

# Serve static files
npx serve dist
```

## Performance

### Code Splitting
```jsx
import dynamic from 'next/dynamic';

// Dynamic import with loading
const DynamicComponent = dynamic(() => import('../components/Heavy'), {
  loading: () => <p>Loading...</p>,
  ssr: false, // Disable server-side rendering
});

// Dynamic import with named export
const DynamicNamedComponent = dynamic(
  () => import('../components/Heavy').then(mod => mod.HeavyComponent)
);

export default function Page() {
  return (
    <div>
      <DynamicComponent />
      <DynamicNamedComponent />
    </div>
  );
}
```

### Bundle Analysis
```bash
# Install bundle analyzer
npm install --save-dev @next/bundle-analyzer

# Update next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // Your Next.js config
});

# Analyze bundle
ANALYZE=true npm run build
```

### Performance Monitoring
```jsx
// pages/_app.js
export function reportWebVitals(metric) {
  switch (metric.name) {
    case 'FCP':
      // First Contentful Paint
      break;
    case 'LCP':
      // Largest Contentful Paint
      break;
    case 'CLS':
      // Cumulative Layout Shift
      break;
    case 'FID':
      // First Input Delay
      break;
    case 'TTFB':
      // Time to First Byte
      break;
    default:
      break;
  }
  
  // Send to analytics
  console.log(metric);
}
```

## Best Practices

### Configuration
```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  
  // Environment variables
  env: {
    CUSTOM_KEY: 'my-value',
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-path',
        destination: '/new-path',
        permanent: true,
      },
    ];
  },
  
  // Headers
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          {
            key: 'Access-Control-Allow-Origin',
            value: '*',
          },
        ],
      },
    ];
  },
  
  // Image optimization
  images: {
    domains: ['example.com'],
    formats: ['image/avif', 'image/webp'],
  },
  
  // Webpack customization
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack }) => {
    // Custom webpack config
    return config;
  },
};

module.exports = nextConfig;
```

### Error Handling
```jsx
// pages/_error.js
function Error({ statusCode, hasGetInitialPropsRun, err }) {
  if (!hasGetInitialPropsRun && err) {
    // Report error to logging service
    console.error(err);
  }

  return (
    <p>
      {statusCode
        ? `An error ${statusCode} occurred on server`
        : 'An error occurred on client'}
    </p>
  );
}

Error.getInitialProps = ({ res, err }) => {
  const statusCode = res ? res.statusCode : err ? err.statusCode : 404;
  return { statusCode };
};

export default Error;
```

### SEO Optimization
```jsx
import Head from 'next/head';

export default function SEOPage() {
  return (
    <>
      <Head>
        <title>Page Title - Site Name</title>
        <meta name="description" content="Page description" />
        <meta name="keywords" content="keyword1, keyword2" />
        <meta property="og:title" content="Page Title" />
        <meta property="og:description" content="Page description" />
        <meta property="og:image" content="/og-image.jpg" />
        <meta name="twitter:card" content="summary_large_image" />
        <link rel="canonical" href="https://example.com/page" />
      </Head>
      
      <h1>Page Content</h1>
    </>
  );
}
```

---

## Common Patterns

| Pattern | Use Case | Implementation |
|---------|----------|----------------|
| Layout Pattern | Shared UI across pages | `_app.js` or `layout.js` |
| HOC Pattern | Wrap components with logic | `withAuth(Component)` |
| Custom Hook | Share stateful logic | `useLocalStorage()` |
| Context Pattern | Global state management | `React.createContext()` |

## Environment Variables

```bash
# .env.local
DATABASE_URL=postgresql://...
NEXTAUTH_SECRET=your-secret
NEXT_PUBLIC_API_URL=https://api.example.com
```

```javascript
// Usage
const dbUrl = process.env.DATABASE_URL; // Server-side only
const apiUrl = process.env.NEXT_PUBLIC_API_URL; // Client and server
```

---

## Resources
- [Official Next.js Documentation](https://nextjs.org/docs)
- [Next.js Examples](https://github.com/vercel/next.js/tree/main/examples)
- [Next.js Learn Course](https://nextjs.org/learn)
- [Vercel Platform](https://vercel.com)
- [Next.js Blog](https://nextjs.org/blog)

---
*Originally compiled from various sources. Contributions welcome!*