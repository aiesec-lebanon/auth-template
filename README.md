# AIESEC OAuth Template for Next.js

This repository is a Next.js authentication template that allows developers to quickly integrate AIESEC OAuth authentication into any web application.

The template implements the entire login and access management flow, including:

- OAuth login with AIESEC Auth
- Access token handling
- Automatic token refresh
- Route protection with middleware
- User authorization based on roles and office IDs
- Client-side authentication context

Developers can use this template as a starting point for building any AIESEC tool or platform while relying on the built-in authentication system.

The goal is to remove the need to repeatedly implement AIESEC OAuth authentication logic for each new project.

## Features
- OAuth Authorization Code Flow
- Secure cookie-based authentication
- Automatic access token refresh
- Middleware route protection
- Role and office-based authorization
- Ready-to-use React auth context
- GraphQL integration with AIESEC EXPA
- Fully customizable authorization rules

## Tech Stack
- Next.js (App Router)
- TypeScript
- Axios
- TailwindCSS

## Project Structure

```
src
 ├── app
 │   ├── api/auth
 │   │   ├── callback
 │   │   │   └── route.ts
 │   │   ├── logout
 │   │   │   └── route.ts
 │   │   ├── me
 │   │   │   └── route.ts
 │   │   └── refresh
 │   │       └── route.ts
 │   │
 │   ├── context
 │   │   └── auth-context.tsx
 │   │
 │   ├── login
 │   │   └── page.tsx
 │   │
 │   ├── unauthorized
 │   │   └── page.tsx
 │   │
 │   ├── layout.tsx
 │   ├── page.tsx
 │   └── globals.css
 │
 ├── server-utils
 │   ├── user-fetcher.ts
 │   └── user-validation.ts
 │
 ├── types
 │
 └── proxy.ts
 ```

 ## Authentication Flow

The authentication system follows the OAuth Authorization Code Flow.

```
User visits protected page
        │
        ▼
Middleware checks authentication cookie
        │
        ├── No token → redirect to /login
        │
        ├── Token expired → /api/auth/refresh
        │
        ▼
User redirected to AIESEC OAuth login
        │
        ▼
AIESEC Auth redirects to
/api/auth/callback?code=XXXX
        │
        ▼
Callback exchanges code for tokens
        │
        ▼
Cookies stored
        │
        ▼
User redirected to application
```

## Running the Project
1. ### Clone the Repository
```bash
git clone <repo-url>
cd <repo-name>
```

2. ### Install Dependencies
```bash
npm install
```

3. ### Create `.env`
Create a file called:
```bash
.env
```
And add the following environment variables:
```env
NEXT_PUBLIC_AIESEC_GRAPHQL_API=https://gis-api.aiesec.org/graphql
NEXT_PUBLIC_CLIENT_ID=
NEXT_PUBLIC_AUTH_URL=https://auth.aiesec.org/oauth
NEXT_PUBLIC_REDIRECT_URI=http://localhost:3000/api/auth/callback

CLIENT_SECRET=
```

4. ### Run the Development Server
```bash
npm run dev
```

## Obtaining OAuth Credentials

To obtain the required credentials:

- Client ID
- Client Secret

Follow the official guide:

Guide: [Creating Developer Application Guide](https://docs.google.com/presentation/d/11hqdefu18fO7j1dIwm9pgCgwnncGYIs8o9Ee26cDkUs/edit?usp=drive_link)

After registering your application, you will receive:

- Client ID
- Client Secret

Add them to `.env`

## Staging Environment (Optional)

If you want to test your application using AIESEC staging services, different endpoints must be used.

Refer to the staging guide:

Guide: [Staging Environment Guide](https://docs.google.com/presentation/d/136ibryj8ZUY5dhGSp3scZ0uTDpZ5FESjaUmVXiusRSY/edit?usp=drive_link)

Example staging endpoints may look like:
```env
AUTH_URL=https://auth-staging.aiesec.org/oauth
GRAPHQL=staging-jruby.aiesec.org/graphql
```

## More on AIESEC Auth

[Read more on AIESEC Auth in this guide.](https://docs.google.com/presentation/d/15Gp1vapG4askZ8oYkcc1Bgk6Wf26id4NE8Oj2Y8Xs5E/edit?usp=sharing)

## Important Customization Points

This template is designed to be customizable depending on the platform you are building.

Below are the most important places to modify.

## Authorization Rules

File:

```bash
src/server-utils/user-validation.ts
```

Here you can define which AIESEC members are allowed to access your platform.
```ts
const ALLOWED_AIESEC_OFFICE_ID = ["1626"];

const ALLOWED_ROLES = [
    "MCP",
    "MCVP"
];
```

### ALLOWED_AIESEC_OFFICE_ID

Defines which AIESEC offices can access the platform.

Example:

```
1626 → AIESEC International
```

You can find office IDs here: [AIESEC Office ID List](https://docs.google.com/spreadsheets/d/1A8Epd__j7OKFO85A0JAJUZcoCuvKee3DHpn0cDafx6E/edit)

If left empty:
```ts
[]
```
All offices will be allowed.

### ALLOWED_ROLES

Defines which AIESEC roles can access the platform.

Example:
```ts
"MCP"
"MCVP"
"LCP"
"LCVP"
```
If empty:
```ts
[]
```
All roles will be allowed.

## Public Routes (No Authentication)

File:
```bash
src/proxy.ts
```
The middleware protects all routes except those listed in the matcher:
```ts
"/((?!login|api/auth|unauthorized|_next/static|_next/image|aiesec_man.png).*)"
```
You may need to modify this if your application has **additional public routes**.

Example:
```ts
"/((?!login|signup|public|api/auth|unauthorized).*)"
```

## User Data Fetching

File:
```bash
src/server-utils/user-fetcher.ts
```
This file fetches user data from the AIESEC GraphQL API.

You can customize the query to retrieve additional user fields if needed.

Example fields already included:

- full name
- profile photo
- office
- role

## React Authentication Context

File:
```bash
src/app/context/auth-context.tsx
```
This provides authentication state to the frontend.

Available values:
```tsx
user
loading
logout()
```
Example usage:
```tsx
const { user, logout } = useAuth();
```

### Middleware Authentication Guard

File:
```bash
src/proxy.ts
```
This middleware protects application routes by checking:
```ts
aiesec_token
refresh_token
token_expires_at
```
If the access token is expired:
```bash
/api/auth/refresh
```
is triggered automatically to obtain a new token.

## API Routes

| Route                | Purpose                        |
| -------------------- | ------------------------------ |
| `/api/auth/callback` | Handles OAuth login response   |
| `/api/auth/me`       | Returns authenticated user     |
| `/api/auth/logout`   | Clears session cookies         |
| `/api/auth/refresh`  | Refreshes expired access token |

## Login Page

File:
```bash
src/app/login/page.tsx
```
This page automatically redirects the user to:
```bash
https://auth.aiesec.org/oauth/authorize
```

## Unauthorized Page

File:
```bash
src/app/unauthorized/page.tsx
```
Displayed when a user is authenticated but does not meet authorization requirements.

## Extending the Template

This template only provides authentication and access control.

You are free to build:
- dashboards
- internal tools
- analytics platforms
- automation tools
- AIESEC platforms

on top of this authentication layer.

## Summary

This template provides a complete AIESEC authentication foundation for Next.js applications.

It handles:
- login
- session management
- authorization
- token refresh
- route protection

so developers can focus on building their actual application features.