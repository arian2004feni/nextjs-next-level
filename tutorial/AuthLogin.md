# Building a Login Form with Next.js, TypeScript, shadcn/ui, Server Actions, Cookies, and `useActionState`

In this tutorial, we'll build a complete login flow using:

* Next.js App Router
* TypeScript
* shadcn/ui
* React `useActionState`
* Next.js Server Actions
* Cookies for storing authentication tokens
* Server-side redirects
* Client-side navigation
* Form validation and pending states

The final flow will look like this:

```text
Login Form
    ↓
useActionState()
    ↓
Server Action
    ↓
Validate credentials
    ↓
Create authentication token
    ↓
Set token in HTTP-only cookie
    ↓
Redirect to dashboard
```

---

## 1. Create the Next.js Project

Start with a new Next.js project:

```bash
npx create-next-app@latest next-login
```

When prompted, select:

```text
TypeScript        Yes
ESLint            Yes
Tailwind CSS      Yes
src/ directory    Yes
App Router        Yes
Turbopack         Yes
Import alias      @/*
```

Then enter the project:

```bash
cd next-login
```

Start the development server:

```bash
npm run dev
```

Your application should now be available at:

```text
http://localhost:3000
```

---

# 2. Install shadcn/ui

Initialize shadcn:

```bash
npx shadcn@latest init
```

Add the components we'll need:

```bash
npx shadcn@latest add button input label card
```

You should now have components such as:

```text
src/components/ui/button.tsx
src/components/ui/input.tsx
src/components/ui/label.tsx
src/components/ui/card.tsx
```

---

# 3. Create the Login Page

Create:

```text
src/app/login/page.tsx
```

Start with a basic page:

```tsx
import { LoginForm } from "@/components/login-form";

export default function LoginPage() {
  return (
    <main className="flex min-h-screen items-center justify-center">
      <LoginForm />
    </main>
  );
}
```

We'll create the actual form next.

---

# 4. Create the Login Form

Create:

```text
src/components/login-form.tsx
```

The login form will eventually submit to a Server Action.

Here's the basic structure:

```tsx
"use client";

import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

export function LoginForm() {
  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle>Login</CardTitle>
        <CardDescription>
          Enter your credentials to access your account.
        </CardDescription>
      </CardHeader>

      <CardContent>
        <form className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="email">Email</Label>

            <Input
              id="email"
              name="email"
              type="email"
              placeholder="you@example.com"
              required
            />
          </div>

          <div className="space-y-2">
            <Label htmlFor="password">Password</Label>

            <Input
              id="password"
              name="password"
              type="password"
              placeholder="••••••••"
              required
            />
          </div>

          <Button type="submit" className="w-full">
            Login
          </Button>
        </form>
      </CardContent>
    </Card>
  );
}
```

At this point we have a working UI, but submitting the form doesn't do anything useful.

Now we'll connect it to a Server Action.

---

# 5. Understanding Server Actions

A Server Action is an asynchronous server-side function that can be invoked directly from a form.

For authentication, this is particularly useful because we can:

1. Receive the form data on the server.
2. Validate the credentials.
3. Authenticate the user.
4. Create an authentication token.
5. Set the token in a cookie.
6. Redirect the user.

Create:

```text
src/app/login/actions.ts
```

Add:

```tsx
"use server";

export async function loginAction(
  previousState: unknown,
  formData: FormData
) {
  const email = formData.get("email");
  const password = formData.get("password");

  console.log(email);
  console.log(password);
}
```

The `"use server"` directive tells Next.js that this module contains Server Actions.

---

# 6. Add `useActionState`

React provides `useActionState` for managing the state returned from an action.

Import it into the login form:

```tsx
import { useActionState } from "react";
```

Then import the Server Action:

```tsx
import { loginAction } from "@/app/login/actions";
```

Create the state:

```tsx
const [state, formAction, isPending] = useActionState(
  loginAction,
  null
);
```

The three returned values are:

```text
state
formAction
isPending
```

### `state`

Contains the value returned by the Server Action.

### `formAction`

This function is passed to the form's `action` attribute.

### `isPending`

Becomes `true` while the Server Action is executing.

---

# 7. Connect the Form to the Server Action

Our component now becomes:

```tsx
"use client";

import { useActionState } from "react";

import { loginAction } from "@/app/login/actions";

import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

export function LoginForm() {
  const [state, formAction, isPending] = useActionState(
    loginAction,
    null
  );

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle>Login</CardTitle>

        <CardDescription>
          Enter your credentials to access your account.
        </CardDescription>
      </CardHeader>

      <CardContent>
        <form action={formAction} className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="email">Email</Label>

            <Input
              id="email"
              name="email"
              type="email"
              placeholder="you@example.com"
              required
            />
          </div>

          <div className="space-y-2">
            <Label htmlFor="password">Password</Label>

            <Input
              id="password"
              name="password"
              type="password"
              placeholder="••••••••"
              required
            />
          </div>

          <Button
            type="submit"
            className="w-full"
            disabled={isPending}
          >
            {isPending ? "Logging in..." : "Login"}
          </Button>

          {state?.error && (
            <p className="text-sm text-red-500">
              {state.error}
            </p>
          )}
        </form>
      </CardContent>
    </Card>
  );
}
```

Notice the important part:

```tsx
<form action={formAction}>
```

We aren't using:

```tsx
onSubmit={...}
```

Instead, React/Next.js invokes the Server Action when the form is submitted.

---

# 8. Define a Proper Action State

Rather than using `unknown`, let's define a TypeScript type.

Create:

```text
src/types/auth.ts
```

```tsx
export type LoginState = {
  error?: string;
  success?: boolean;
};
```

Now update the Server Action:

```tsx
"use server";

import type { LoginState } from "@/types/auth";

export async function loginAction(
  previousState: LoginState | null,
  formData: FormData
): Promise<LoginState> {
  const email = formData.get("email");
  const password = formData.get("password");

  if (!email || !password) {
    return {
      error: "Email and password are required.",
    };
  }

  return {
    success: true,
  };
}
```

And the form:

```tsx
const [state, formAction, isPending] = useActionState<
  LoginState | null,
  FormData
>(loginAction, null);
```

---

# 9. Validate the Form on the Server

Client-side validation is useful for user experience, but authentication validation must happen on the server.

Don't trust:

```tsx
required
```

alone.

A malicious client can bypass browser validation entirely.

Update the action:

```tsx
"use server";

import type { LoginState } from "@/types/auth";

export async function loginAction(
  previousState: LoginState | null,
  formData: FormData
): Promise<LoginState> {
  const email = formData.get("email");
  const password = formData.get("password");

  if (typeof email !== "string") {
    return {
      error: "Invalid email.",
    };
  }

  if (typeof password !== "string") {
    return {
      error: "Invalid password.",
    };
  }

  if (!email || !password) {
    return {
      error: "Email and password are required.",
    };
  }

  // Authentication will happen here.

  return {
    success: true,
  };
}
```

This is important because `FormData.get()` returns:

```text
FormDataEntryValue | null
```

and `FormDataEntryValue` can be a `File`.

Therefore, checking:

```tsx
typeof email === "string"
```

is a good practice.

---

# 10. Authenticate the User

In a real application, you would query your database and verify the password against its stored password hash.

For example:

```tsx
const user = await getUserByEmail(email);

if (!user) {
  return {
    error: "Invalid email or password.",
  };
}

const passwordIsValid = await verifyPassword(
  password,
  user.passwordHash
);

if (!passwordIsValid) {
  return {
    error: "Invalid email or password.",
  };
}
```

Avoid revealing which credential was wrong.

Don't return:

```text
Email doesn't exist.
```

and:

```text
Wrong password.
```

Instead, use a generic message:

```text
Invalid email or password.
```

This helps prevent account enumeration.

---

# 11. Create an Authentication Token

Once the credentials are valid, create a session/token.

Conceptually:

```tsx
const token = await createAuthToken(user.id);
```

For example, your authentication layer might return:

```text
eyJhbGciOiJIUzI1NiIs...
```

The exact token implementation depends on your authentication architecture.

A common architecture is:

```text
User
 ↓
Database
 ↓
Session
 ↓
Session ID / token
 ↓
HTTP-only cookie
```

For many applications, a server-side session cookie is preferable to storing sensitive authentication information in browser-accessible storage.

---

# 12. Set the Token in a Next.js Cookie

Next.js provides the `cookies()` API for working with cookies in Server Actions and Server Components.

Import it:

```tsx
import { cookies } from "next/headers";
```

Then:

```tsx
const cookieStore = await cookies();

cookieStore.set("auth_token", token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === "production",
  sameSite: "lax",
  path: "/",
});
```

The important security option is:

```tsx
httpOnly: true
```

This prevents client-side JavaScript from reading the cookie.

That means this won't be able to access it:

```tsx
document.cookie
```

This is desirable for authentication cookies.

---

# 13. Understand the Cookie Options

Our cookie:

```tsx
cookieStore.set("auth_token", token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === "production",
  sameSite: "lax",
  path: "/",
});
```

has several important properties.

### `httpOnly`

```tsx
httpOnly: true
```

JavaScript running in the browser can't read the cookie.

This reduces exposure from client-side JavaScript attacks such as cookie theft through XSS.

### `secure`

```tsx
secure: process.env.NODE_ENV === "production"
```

In production, the browser only sends the cookie over HTTPS.

### `sameSite`

```tsx
sameSite: "lax"
```

Provides protection against many cross-site request scenarios while still allowing normal navigation.

### `path`

```tsx
path: "/"
```

Makes the cookie available throughout the application.

---

# 14. Add an Expiration Time

You can also configure expiration.

For example:

```tsx
cookieStore.set("auth_token", token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === "production",
  sameSite: "lax",
  path: "/",
  maxAge: 60 * 60 * 24 * 7,
});
```

This represents seven days:

```text
60 seconds
× 60 minutes
× 24 hours
× 7 days
```

Your session strategy should determine the appropriate lifetime.

For sensitive applications, shorter sessions and refresh-token/session-rotation strategies may be preferable.

---

# 15. Redirect the User After Login

After setting the authentication cookie, we usually want to redirect the user to the dashboard.

Next.js provides:

```tsx
redirect()
```

Import it:

```tsx
import { redirect } from "next/navigation";
```

Then:

```tsx
redirect("/dashboard");
```

The complete action could look like:

```tsx
"use server";

import { cookies } from "next/headers";
import { redirect } from "next/navigation";

import type { LoginState } from "@/types/auth";

export async function loginAction(
  previousState: LoginState | null,
  formData: FormData
): Promise<LoginState> {
  const email = formData.get("email");
  const password = formData.get("password");

  if (
    typeof email !== "string" ||
    typeof password !== "string"
  ) {
    return {
      error: "Invalid email or password.",
    };
  }

  if (!email || !password) {
    return {
      error: "Email and password are required.",
    };
  }

  // Example authentication.
  //
  // const user = await getUserByEmail(email);
  // const valid = await verifyPassword(
  //   password,
  //   user.passwordHash
  // );

  const authenticated = true;

  if (!authenticated) {
    return {
      error: "Invalid email or password.",
    };
  }

  const token = "your-generated-auth-token";

  const cookieStore = await cookies();

  cookieStore.set("auth_token", token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
    path: "/",
    maxAge: 60 * 60 * 24 * 7,
  });

  redirect("/dashboard");
}
```

In production, obviously replace the placeholder token with your real authentication/session implementation.

---

# 16. An Important Detail About `redirect()`

You may notice that the action is typed as:

```tsx
Promise<LoginState>
```

but:

```tsx
redirect("/dashboard");
```

doesn't return a `LoginState`.

That's okay.

Next.js's `redirect()` interrupts the current request and performs the redirect. You don't need to write:

```tsx
return redirect("/dashboard");
```

In fact, the usual pattern is:

```tsx
redirect("/dashboard");
```

after successful authentication.

The action only returns a `LoginState` when the login fails and the form needs to display an error.

---

# 17. The Final Login Form

Our form can now be:

```tsx
"use client";

import { useActionState } from "react";

import { loginAction } from "@/app/login/actions";
import type { LoginState } from "@/types/auth";

import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

export function LoginForm() {
  const [state, formAction, isPending] = useActionState<
    LoginState | null,
    FormData
  >(loginAction, null);

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle>Login</CardTitle>

        <CardDescription>
          Enter your credentials to access your account.
        </CardDescription>
      </CardHeader>

      <CardContent>
        <form action={formAction} className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="email">
              Email
            </Label>

            <Input
              id="email"
              name="email"
              type="email"
              placeholder="you@example.com"
              autoComplete="email"
              required
            />
          </div>

          <div className="space-y-2">
            <Label htmlFor="password">
              Password
            </Label>

            <Input
              id="password"
              name="password"
              type="password"
              placeholder="••••••••"
              autoComplete="current-password"
              required
            />
          </div>

          {state?.error && (
            <p
              role="alert"
              className="text-sm text-red-500"
            >
              {state.error}
            </p>
          )}

          <Button
            type="submit"
            className="w-full"
            disabled={isPending}
          >
            {isPending
              ? "Logging in..."
              : "Login"}
          </Button>
        </form>
      </CardContent>
    </Card>
  );
}
```

---

# 18. Handling the Pending State

The important line is:

```tsx
const [state, formAction, isPending] = useActionState(
  loginAction,
  null
);
```

While the Server Action is executing:

```tsx
isPending === true
```

Therefore:

```tsx
<Button disabled={isPending}>
  {isPending ? "Logging in..." : "Login"}
</Button>
```

gives the user immediate feedback.

The flow becomes:

```text
User clicks Login
       ↓
isPending = true
       ↓
Button becomes disabled
       ↓
"Logging in..."
       ↓
Server Action executes
       ↓
Authentication
       ↓
Cookie is created
       ↓
redirect("/dashboard")
```

This prevents users from repeatedly clicking the login button while authentication is processing.

---

# 19. Displaying Server-Side Validation Errors

Suppose authentication fails:

```tsx
return {
  error: "Invalid email or password.",
};
```

`useActionState` receives that return value:

```tsx
state.error
```

and the component renders:

```tsx
{state?.error && (
  <p role="alert">
    {state.error}
  </p>
)}
```

This is one of the main benefits of `useActionState`: the Server Action can return structured state directly to the form.

---

# 20. Create a Protected Dashboard

Create:

```text
src/app/dashboard/page.tsx
```

For example:

```tsx
export default function DashboardPage() {
  return (
    <main className="p-8">
      <h1 className="text-2xl font-bold">
        Dashboard
      </h1>

      <p>
        You are successfully logged in.
      </p>
    </main>
  );
}
```

But there's a problem.

A user can directly visit:

```text
/dashboard
```

without logging in.

We need to protect the route.

---

# 21. Read the Authentication Cookie on the Server

Because the cookie is HTTP-only, server-side code can read it.

For example:

```tsx
import { cookies } from "next/headers";

export default async function DashboardPage() {
  const cookieStore = await cookies();

  const token = cookieStore.get("auth_token");

  if (!token) {
    // User is not authenticated.
  }

  return (
    <main>
      Dashboard
    </main>
  );
}
```

You should not merely check whether a cookie exists.

You need to validate the token/session against your authentication system.

Conceptually:

```tsx
const session = await getSession(token);

if (!session) {
  redirect("/login");
}
```

---

# 22. Server-Side Navigation

This is the preferred approach when authentication is handled on the server.

```tsx
import { cookies } from "next/headers";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const cookieStore = await cookies();

  const token = cookieStore.get("auth_token")?.value;

  if (!token) {
    redirect("/login");
  }

  const session = await getSession(token);

  if (!session) {
    redirect("/login");
  }

  return (
    <main className="p-8">
      <h1 className="text-2xl font-bold">
        Welcome to your dashboard
      </h1>
    </main>
  );
}
```

This happens on the server before the protected page is rendered.

---

# 23. Client-Side Navigation

Next.js also provides client-side navigation through:

```tsx
useRouter()
```

Import:

```tsx
"use client";

import { useRouter } from "next/navigation";
```

Then:

```tsx
const router = useRouter();

router.push("/dashboard");
```

For example:

```tsx
"use client";

import { useRouter } from "next/navigation";

export function GoToDashboardButton() {
  const router = useRouter();

  return (
    <button
      onClick={() => router.push("/dashboard")}
    >
      Go to Dashboard
    </button>
  );
}
```

This is useful when navigation is triggered by a client-side interaction.

---

# 24. Client-Side vs Server-Side Navigation

There is an important distinction.

### Client-side

```tsx
router.push("/dashboard");
```

Useful when:

* A client component controls navigation.
* Navigation occurs after some client-side interaction.
* You need client-side routing behavior.

### Server-side

```tsx
redirect("/dashboard");
```

Useful when:

* Authentication has just happened.
* Authorization is being checked.
* A protected server-rendered route needs to redirect.
* You want the server to make the authentication decision.

For login specifically, server-side redirect is usually cleaner:

```text
Login Form
    ↓
Server Action
    ↓
Validate credentials
    ↓
Set HTTP-only cookie
    ↓
redirect("/dashboard")
```

Rather than:

```text
Login Form
    ↓
Server Action
    ↓
Return success
    ↓
Client receives success
    ↓
router.push("/dashboard")
```

---

# 25. Why Server-Side Redirect Is Better for Login

Consider this:

```tsx
const [state, formAction] = useActionState(
  loginAction,
  null
);
```

The action performs:

```tsx
cookieStore.set(...);

redirect("/dashboard");
```

The browser receives the redirect as part of the server response.

This means the authentication decision and navigation happen together.

You don't need:

```tsx
if (state.success) {
  router.push("/dashboard");
}
```

This is especially useful because authentication is fundamentally a server-side concern.

---

# 26. When Should You Use `router.push()`?

Client-side navigation is still useful.

For example, after clicking a "Create Account" button:

```tsx
"use client";

import { useRouter } from "next/navigation";

export function SignupButton() {
  const router = useRouter();

  return (
    <button onClick={() => router.push("/signup")}>
      Create account
    </button>
  );
}
```

Or:

```tsx
router.push("/forgot-password");
```

for a forgot-password flow.

---

# 27. `router.replace()` vs `router.push()`

Next.js also provides:

```tsx
router.replace("/dashboard");
```

The difference is browser history.

### `push`

```tsx
router.push("/dashboard");
```

Adds a new history entry.

The user can press Back and return to the previous page.

### `replace`

```tsx
router.replace("/dashboard");
```

Replaces the current history entry.

This can be useful for flows where you don't want the previous page to remain in the history.

For example, after certain authentication transitions:

```text
/login
  ↓
/dashboard
```

you may prefer:

```tsx
router.replace("/dashboard");
```

if you are performing the navigation on the client.

---

# 28. Login Action With a Realistic Structure

A production-oriented Server Action might look like this:

```tsx
"use server";

import { cookies } from "next/headers";
import { redirect } from "next/navigation";

import type { LoginState } from "@/types/auth";

export async function loginAction(
  previousState: LoginState | null,
  formData: FormData
): Promise<LoginState> {
  const email = formData.get("email");
  const password = formData.get("password");

  if (
    typeof email !== "string" ||
    typeof password !== "string"
  ) {
    return {
      error: "Invalid email or password.",
    };
  }

  if (!email || !password) {
    return {
      error: "Email and password are required.",
    };
  }

  // 1. Find user.
  const user = await getUserByEmail(email);

  if (!user) {
    return {
      error: "Invalid email or password.",
    };
  }

  // 2. Verify password.
  const passwordValid = await verifyPassword(
    password,
    user.passwordHash
  );

  if (!passwordValid) {
    return {
      error: "Invalid email or password.",
    };
  }

  // 3. Create session.
  const token = await createSession(user.id);

  // 4. Store session token.
  const cookieStore = await cookies();

  cookieStore.set("auth_token", token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
    path: "/",
    maxAge: 60 * 60 * 24 * 7,
  });

  // 5. Navigate to protected page.
  redirect("/dashboard");
}
```

The functions:

```tsx
getUserByEmail()
verifyPassword()
createSession()
```

would be implemented by your database and authentication layer.

---

# 29. Recommended Project Structure

A clean project structure could be:

```text
src/
├── app/
│   ├── dashboard/
│   │   └── page.tsx
│   │
│   ├── login/
│   │   ├── actions.ts
│   │   └── page.tsx
│   │
│   └── page.tsx
│
├── components/
│   ├── login-form.tsx
│   └── ui/
│       ├── button.tsx
│       ├── card.tsx
│       ├── input.tsx
│       └── label.tsx
│
├── lib/
│   ├── auth.ts
│   ├── db.ts
│   └── session.ts
│
└── types/
    └── auth.ts
```

The responsibilities are separated:

```text
login/page.tsx
    → page layout

login-form.tsx
    → interactive UI

login/actions.ts
    → Server Action

lib/auth.ts
    → authentication logic

lib/session.ts
    → session/token logic

dashboard/page.tsx
    → protected page
```

---

# 30. Avoid Putting Authentication Logic in the Client

Don't do this:

```tsx
"use client";

const response = await fetch("/api/login", {
  method: "POST",
  body: JSON.stringify({
    email,
    password,
  }),
});
```

followed by:

```tsx
localStorage.setItem("token", token);
```

for a typical web authentication flow.

A better architecture is:

```text
Client
   │
   │ Form submission
   ▼
Server Action
   │
   ├── Validate input
   ├── Query database
   ├── Verify password
   ├── Create session
   └── Set HTTP-only cookie
          │
          ▼
      Redirect
```

The authentication token isn't exposed to client-side JavaScript.

---

# 31. Why Not `localStorage` for Authentication Tokens?

Avoid:

```tsx
localStorage.setItem(
  "auth_token",
  token
);
```

for sensitive authentication tokens when an HTTP-only cookie-based session is appropriate.

JavaScript can access `localStorage`:

```tsx
localStorage.getItem("auth_token");
```

An HTTP-only cookie cannot be read by JavaScript.

That gives you a stronger boundary:

```text
localStorage
    ↓
JavaScript can read it

HTTP-only cookie
    ↓
JavaScript cannot read it
    ↓
Browser sends it automatically
```

You still need proper XSS protection, CSRF considerations, session management, and secure cookie configuration.

---

# 32. Complete Flow

At this point, our application works like this:

```text
                    Browser
                       │
                       │ POST form
                       ▼
                ┌───────────────┐
                │ Server Action │
                └───────┬───────┘
                        │
                Validate input
                        │
                        ▼
                Find user in DB
                        │
                        ▼
                Verify password
                        │
               ┌────────┴────────┐
               │                 │
            Invalid            Valid
               │                 │
               ▼                 ▼
        return { error }    Create session
                                 │
                                 ▼
                         Set HTTP-only cookie
                                 │
                                 ▼
                       redirect("/dashboard")
                                 │
                                 ▼
                             Dashboard
```

---

# 33. Full Client/Server Example

### `src/types/auth.ts`

```tsx
export type LoginState = {
  error?: string;
};
```

### `src/app/login/actions.ts`

```tsx
"use server";

import { cookies } from "next/headers";
import { redirect } from "next/navigation";

import type { LoginState } from "@/types/auth";

export async function loginAction(
  previousState: LoginState | null,
  formData: FormData
): Promise<LoginState> {
  const email = formData.get("email");
  const password = formData.get("password");

  if (
    typeof email !== "string" ||
    typeof password !== "string"
  ) {
    return {
      error: "Invalid email or password.",
    };
  }

  if (!email || !password) {
    return {
      error: "Email and password are required.",
    };
  }

  // Replace these with your real authentication functions.

  const user = await getUserByEmail(email);

  if (!user) {
    return {
      error: "Invalid email or password.",
    };
  }

  const valid = await verifyPassword(
    password,
    user.passwordHash
  );

  if (!valid) {
    return {
      error: "Invalid email or password.",
    };
  }

  const token = await createSession(user.id);

  const cookieStore = await cookies();

  cookieStore.set("auth_token", token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
    path: "/",
    maxAge: 60 * 60 * 24 * 7,
  });

  redirect("/dashboard");
}
```

### `src/components/login-form.tsx`

```tsx
"use client";

import { useActionState } from "react";

import { loginAction } from "@/app/login/actions";
import type { LoginState } from "@/types/auth";

import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

export function LoginForm() {
  const [state, formAction, isPending] = useActionState<
    LoginState | null,
    FormData
  >(loginAction, null);

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle>Login</CardTitle>

        <CardDescription>
          Enter your credentials to continue.
        </CardDescription>
      </CardHeader>

      <CardContent>
        <form action={formAction} className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="email">
              Email
            </Label>

            <Input
              id="email"
              name="email"
              type="email"
              autoComplete="email"
              placeholder="you@example.com"
              required
            />
          </div>

          <div className="space-y-2">
            <Label htmlFor="password">
              Password
            </Label>

            <Input
              id="password"
              name="password"
              type="password"
              autoComplete="current-password"
              placeholder="••••••••"
              required
            />
          </div>

          {state?.error && (
            <p
              role="alert"
              className="text-sm text-red-500"
            >
              {state.error}
            </p>
          )}

          <Button
            type="submit"
            className="w-full"
            disabled={isPending}
          >
            {isPending
              ? "Logging in..."
              : "Login"}
          </Button>
        </form>
      </CardContent>
    </Card>
  );
}
```

### `src/app/login/page.tsx`

```tsx
import { LoginForm } from "@/components/login-form";

export default function LoginPage() {
  return (
    <main className="flex min-h-screen items-center justify-center px-4">
      <LoginForm />
    </main>
  );
}
```

---

# 34. What Happens During Submission?

Let's trace one login request.

The user enters:

```text
Email:
john@example.com

Password:
********
```

Then clicks:

```text
Login
```

The client calls:

```tsx
formAction
```

React changes:

```tsx
isPending
```

to:

```text
true
```

The button changes to:

```text
Logging in...
```

The Server Action receives:

```tsx
formData.get("email")
formData.get("password")
```

The server validates the credentials.

If invalid:

```tsx
return {
  error: "Invalid email or password."
};
```

The form receives that state.

If valid:

```tsx
const token = await createSession(user.id);
```

Then:

```tsx
cookieStore.set("auth_token", token, {
  httpOnly: true,
  secure: true,
  sameSite: "lax",
  path: "/",
});
```

Finally:

```tsx
redirect("/dashboard");
```

The browser navigates to:

```text
/dashboard
```

---

# 35. Server Action vs API Route

You may wonder whether you need an API endpoint such as:

```text
POST /api/login
```

Not necessarily.

For a form handled entirely within your Next.js application, a Server Action can provide a simpler architecture:

```text
Form
 ↓
Server Action
 ↓
Database
 ↓
Cookie
 ↓
Redirect
```

Instead of:

```text
Form
 ↓
fetch()
 ↓
/api/login
 ↓
Database
 ↓
JSON response
 ↓
Client code
 ↓
router.push()
```

Server Actions are particularly convenient for server-side form mutations.

---

# 36. Security Checklist

For a production login implementation, don't stop at the basic example.

### Passwords

Never store plaintext passwords.

Use a modern password-hashing algorithm such as:

```text
Argon2id
```

or an appropriately configured bcrypt implementation.

### Authentication cookies

Use:

```tsx
httpOnly: true
```

and in production:

```tsx
secure: true
```

Use an appropriate:

```tsx
sameSite
```

policy.

### Sessions

Prefer short-lived/rotating sessions where appropriate.

Provide a way to invalidate sessions on logout.

### Login errors

Use:

```text
Invalid email or password.
```

rather than revealing which credential failed.

### Rate limiting

Protect login endpoints/actions against brute-force attacks.

### HTTPS

Production authentication should use HTTPS.

### CSRF

Evaluate CSRF protections appropriate to your authentication architecture and cookie configuration.

### Authorization

Authentication answers:

```text
"Who are you?"
```

Authorization answers:

```text
"Are you allowed to do this?"
```

Don't assume that having a valid session means the user can access every resource.

---

# 37. The Key Concepts to Remember

The entire tutorial can be reduced to five important ideas.

## 1. `useActionState`

Use it to connect a form to a Server Action and manage returned state:

```tsx
const [state, formAction, isPending] =
  useActionState(loginAction, null);
```

---

## 2. Server Action

Use a Server Action for the authentication mutation:

```tsx
"use server";

export async function loginAction(
  previousState,
  formData
) {
  // authenticate
}
```

---

## 3. HTTP-only Cookie

After authentication:

```tsx
const cookieStore = await cookies();

cookieStore.set("auth_token", token, {
  httpOnly: true,
  secure: true,
  sameSite: "lax",
  path: "/",
});
```

---

## 4. Server Redirect

After successful authentication:

```tsx
redirect("/dashboard");
```

This is generally the cleanest way to navigate after a login Server Action.

---

## 5. Client Navigation

When navigation genuinely needs to be controlled by a Client Component:

```tsx
const router = useRouter();

router.push("/dashboard");
```

or:

```tsx
router.replace("/dashboard");
```

---

# Final Architecture

A good mental model for this implementation is:

```text
┌──────────────────────────────┐
│       LoginForm              │
│      Client Component        │
│                              │
│  useActionState(loginAction) │
│                              │
│  isPending → loading UI      │
└──────────────┬───────────────┘
               │
               │ FormData
               ▼
┌──────────────────────────────┐
│       Server Action          │
│                              │
│  Validate input              │
│  Find user                   │
│  Verify password             │
│  Create session              │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│       HTTP-only Cookie       │
│                              │
│  auth_token                  │
│  Secure                      │
│  HttpOnly                    │
│  SameSite                    │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│       Server Redirect        │
│                              │
│     redirect("/dashboard")   │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│         Dashboard            │
│                              │
│  Validate session            │
│  Render protected content    │
└──────────────────────────────┘
```

This architecture keeps the **UI concerns in the Client Component**, the **authentication logic on the server**, the **session credential inside an HTTP-only cookie**, and the **authorization/navigation decisions on the server** wherever possible.
