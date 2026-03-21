---
name: page-scaffold
description: >
  Create new Next.js pages following the project's Pages Router conventions, including public-facing
  pages with SEO, admin-protected pages with auth, and dynamic routes. Use this skill whenever
  the user wants to add a new route, create a new page, add a public page, create an admin page,
  add a landing page, or set up a new URL endpoint. Trigger on: "new page", "add route",
  "create page", "landing page", "add a page at /...", or when the user describes any new
  URL that needs to exist in the application.
---

# Page Scaffold

Create new Next.js pages using the project's Pages Router conventions. The project uses the `pages/` directory (not App Router) with specific patterns for public pages, admin pages, and dynamic routes.

## Page Types

### 1. Public Page (visitor-facing)

For pages accessible without authentication. Uses `PresentationLayout` (header + footer) and `NextSeo` for SEO.

```jsx
import { NextSeo } from 'next-seo';
import { PresentationLayout } from '@components';

const Page = ({ data }) => {
  return (
    <PresentationLayout>
      <NextSeo
        title="Page Title — Site Name"
        description="Page description for search engines"
        openGraph={{
          title: 'Page Title',
          description: 'Page description',
          // images: [{ url: '/images/og-image.jpg', width: 1200, height: 630 }],
        }}
      />

      {/* Page content */}
      <section className="py-16">
        <div className="container mx-auto px-4">
          <h1 className="text-4xl font-bold text-gray-900">Page Title</h1>
        </div>
      </section>
    </PresentationLayout>
  );
};

// Optional: server-side data fetching
export async function getServerSideProps() {
  try {
    // const data = await publicApi.getData();
    return { props: { /* data */ } };
  } catch {
    return { props: {} };
  }
}

export default Page;
```

### 2. Admin Page (protected)

For pages requiring authentication. Uses `Layout` (admin sidebar + header) with `checkAuth`/`withAuth`.

```jsx
import { checkAuth, withAuth } from '@auth';
import { Layout } from '@components';

const Page = () => {
  return (
    <Layout title="Page Title">
      {/* Admin page content */}
    </Layout>
  );
};

export async function getServerSideProps(context) {
  return await checkAuth(context);
}

export default withAuth(Page);
```

### 3. Dynamic Route Page

For pages with URL parameters. Use `[param].js` file naming.

**Public dynamic route** (e.g., `/blog/[slug].js`):
```jsx
import { NextSeo } from 'next-seo';
import { PresentationLayout } from '@components';

const Page = ({ item }) => (
  <PresentationLayout>
    <NextSeo title={item?.title} description={item?.description} />
    {/* Render item */}
  </PresentationLayout>
);

export async function getServerSideProps({ params }) {
  try {
    const item = await publicApi.getBySlug(params.slug);
    if (!item) return { notFound: true };
    return { props: { item } };
  } catch {
    return { notFound: true };
  }
}

export default Page;
```

**Admin dynamic route** (e.g., `/admin/resources/edit/[id].js`):
```jsx
import { checkAuth, withAuth } from '@auth';
import { Button, Layout } from '@components';
import { EditResourceForm } from '@components/Forms';
import { useQuery } from '@hooks';
import { useRouter } from 'next/router';

const Page = () => {
  const router = useRouter();
  const { id } = router.query;
  const { data: resource, status } = useQuery(`admin/resources/${id}`);

  return (
    <Layout title="Edit Resource">
      {status === 'loading' && <div>Loading...</div>}
      {status === 'error' && <div>Error loading resource.</div>}
      {status === 'success' && (
        <>
          <Button onClick={() => router.back()} className="text-yellow-600 hover:text-yellow-700 text-sm font-medium mb-4">
            ← Back
          </Button>
          <EditResourceForm resource={resource} />
        </>
      )}
    </Layout>
  );
};

export async function getServerSideProps(context) {
  return await checkAuth(context);
}

export default withAuth(Page);
```

## File Naming Conventions

| Pattern | File Path | URL |
|---------|-----------|-----|
| Static page | `pages/about.js` | `/about` |
| Index in folder | `pages/blog/index.js` | `/blog` |
| Dynamic param | `pages/blog/[slug].js` | `/blog/my-post` |
| Nested dynamic | `pages/admin/articles/edit/[id].js` | `/admin/articles/edit/abc123` |

- File names use **kebab-case** (e.g., `forgot-password.js`)
- Use `index.js` inside directories for clean URLs (e.g., `pages/admin/articles/add/index.js` → `/admin/articles/add`)

## Layout Components

| Layout | Use For | Provides |
|--------|---------|----------|
| `PresentationLayout` | Public/visitor pages | Header + Footer + mobile menu |
| `Layout` | Admin pages | Admin sidebar + header + title prop |

Import both from `@components`.

## SEO Setup

Use `next-seo` package for per-page SEO:

```jsx
import { NextSeo } from 'next-seo';

<NextSeo
  title="Page Title"
  description="Meta description"
  canonical="https://example.com/page"
  openGraph={{
    title: 'OG Title',
    description: 'OG Description',
    images: [{ url: '/images/og.jpg', width: 1200, height: 630, alt: 'Image description' }],
  }}
/>
```

The project also has a custom `OpenGraph` component in `@components` for more control.

## Auth Flow

The authentication flow for admin pages uses two layers:

1. **Server-side** (`getServerSideProps`): `checkAuth(context)` validates the JWT token from cookies. Redirects to `/login` if invalid.
2. **Client-side** (`withAuth` HOC): Monitors token expiry, triggers refresh on window focus, redirects on auth failure.

Always use both together for admin pages.

## Key Import Map

| What | Import From |
|------|------------|
| PresentationLayout, Layout, Button | `@components` |
| checkAuth, withAuth | `@auth` |
| useQuery, useInfiniteQuery | `@hooks` |
| NextSeo | `next-seo` |
| useRouter | `next/router` |
