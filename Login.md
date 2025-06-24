Certainly! Here’s a clean, well-structured, and developer-focused documentation for your **Next.js Login Authentication System**. This version prioritizes clarity, modularity, and best practices for onboarding other developers.

---

# 🔐 Next.js Login Authentication System

A robust and modern authentication system using **Next.js App Router**, **React Hook Form**, **Zod**, and **js-cookie**. This guide covers setup, structure, and usage.

---

## 📦 Installation

Make sure your project uses **Next.js 13+ (App Router)** and **Tailwind CSS**.

Install required packages:

```bash
npm install js-cookie lucide-react sonner
```

Add UI components via shadcn:

```bash
npx shadcn@latest add form input label
```

Install types for js-cookie (for TypeScript):

```bash
npm install --save-dev @types/js-cookie
```

---

## 🗂️ Project Structure Overview

```text
src/
  components/
    ui/
      input.ts
  action/
    auth/
      login.ts
  schemas/
    auth/
      index.ts
app/
  (auth)/
    login/
      page.tsx
      _components/
        login-form.tsx
```

---

## 🏗️ Component & Logic Breakdown

### 1. Input Component (`src/components/ui/input.ts`)

Provides an input field with optional icons for enhanced UI.

```tsx
import * as React from "react";
import { cn } from "@/lib/utils";
import { LucideIcon } from "lucide-react";

export interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  startIcon?: LucideIcon;
  endIcon?: LucideIcon;
}

const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ className, type, startIcon, endIcon, ...props }, ref) => {
    const StartIcon = startIcon;
    const EndIcon = endIcon;

    return (
      <div className="w-full relative">
        {StartIcon && (
          <div className="absolute left-1.5 top-1/2 transform -translate-y-1/2 pl-1">
            <StartIcon size={18} className="text-muted-foreground" />
          </div>
        )}
        <input
          type={type}
          className={cn(
            "flex pl-1 h-10 w-full rounded-md border border-input bg-background py-2 px-4 text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50",
            startIcon ? "pl-8" : "",
            endIcon ? "pr-8" : "",
            className
          )}
          ref={ref}
          {...props}
        />
        {EndIcon && (
          <div className="absolute right-3 top-1/2 transform -translate-y-1/2">
            <EndIcon className="text-muted-foreground" size={18} />
          </div>
        )}
      </div>
    );
  }
);
Input.displayName = "Input";
export { Input };
```

---

### 2. Login Page (`app/(auth)/login/page.tsx`)

Handles page layout and imports the login form.

```tsx
import dynamic from "next/dynamic";
import Image from "next/image";
import { Suspense } from "react";

const LoginForm = dynamic(() => import("./_components/login-form"));

export default function LoginPage() {
  return (
    <div className="flex min-h-screen">
      {/* Left: Illustration */}
      <div className="hidden lg:w-3/5 md:w-1/2 bg-gray-900 lg:block relative">
        <Image
          src="https://files.edgestore.dev/t7diwg54d3s82m9n/wellnessmclear/_public/login.jpg"
          alt="Team meeting"
          fill
          className="object-cover"
        />
      </div>
      {/* Right: Login Form */}
      <div className="flex w-full flex-col items-center justify-center px-4 py-12 lg:w-1/2 relative">
        <div className="mx-auto w-full max-w-md space-y-12">
          <div className="text-center">
            <h1 className="text-2xl font-bold text-gray-900">
              Welcome <span>back</span>
            </h1>
            <p className="mt-2 text-sm text-gray-600">
              Please enter your credentials to continue
            </p>
          </div>
          <Suspense>
            <LoginForm />
          </Suspense>
        </div>
      </div>
    </div>
  );
}
```

---

### 3. Login Form Component (`app/(auth)/login/_components/login-form.tsx`)

Handles form state, validation, and authentication calls.

```tsx
"use client";

import { zodResolver } from "@hookform/resolvers/zod";
import { Eye, EyeOff, Lock, Mail } from "lucide-react";
import { useEffect, useState, useTransition } from "react";
import { useForm } from "react-hook-form";
import { loginAction } from "@/action/auth/login";
import { Button } from "@/components/ui/button";
import { Checkbox } from "@/components/ui/checkbox";
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormMessage,
} from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import { loginFormSchema, LoginFormValues } from "@/schemas/auth";
import Cookies from "js-cookie";
import Link from "next/link";
import { useRouter, useSearchParams } from "next/navigation";
import { toast } from "sonner";

const rememberedEmail = Cookies.get("rememberMeEmail");
const rememberMePassword = Cookies.get("rememberMePassword");
const isRemembered = !!rememberedEmail && !!rememberMePassword;

export default function LoginForm() {
  const [showPassword, setShowPassword] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  const [pending, startTransition] = useTransition();
  const [isMounted, setMounted] = useState(false);

  const router = useRouter();
  const searchParams = useSearchParams();
  const callback = searchParams.get("callback") || undefined;

  useEffect(() => {
    setMounted(true);
  }, []);

  const form = useForm<LoginFormValues>({
    resolver: zodResolver(loginFormSchema),
    defaultValues: {
      email: rememberedEmail ?? "",
      password: rememberMePassword ?? "",
      rememberMe: isRemembered ?? false,
    },
  });

  async function onSubmit(data: LoginFormValues) {
    startTransition(() => {
      loginAction(data).then((res) => {
        if (!res.success) {
          toast.error(res.message);
          return;
        }
        setIsLoading(true);
        router.push(callback ?? "/");
      });
    });
  }

  const loading = isLoading || pending;
  if (!isMounted) return null;

  return (
    <>
      <Form {...form}>
        <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
          <FormField
            control={form.control}
            name="email"
            render={({ field }) => (
              <FormItem>
                <FormControl>
                  <div className="relative">
                    <Input
                      {...field}
                      placeholder="Enter your email"
                      type="email"
                      className="border-primary border-[1px] min-h-[45px]"
                      disabled={loading}
                      startIcon={Mail}
                    />
                  </div>
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />
          <FormField
            control={form.control}
            name="password"
            render={({ field }) => (
              <FormItem>
                <FormControl>
                  <div className="relative">
                    <Input
                      {...field}
                      placeholder="Enter your Password"
                      type={showPassword ? "text" : "password"}
                      autoComplete="current-password"
                      className="pr-10 border-primary border-[1px] min-h-[45px]"
                      startIcon={Lock}
                      disabled={loading}
                    />
                    <button
                      type="button"
                      onClick={() => setShowPassword(!showPassword)}
                      className="absolute right-3 top-3 text-gray-400"
                      tabIndex={-1}
                    >
                      {showPassword ? (
                        <EyeOff className="h-5 w-5" />
                      ) : (
                        <Eye className="h-5 w-5" />
                      )}
                    </button>
                  </div>
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />
          <div className="flex items-center justify-between">
            <FormField
              control={form.control}
              name="rememberMe"
              render={({ field }) => (
                <div className="flex items-center space-x-2">
                  <Checkbox
                    id="rememberMe"
                    checked={field.value}
                    onCheckedChange={field.onChange}
                    disabled={loading}
                  />
                  <label
                    htmlFor="rememberMe"
                    className="text-sm font-medium text-gray-700"
                  >
                    Remember me
                  </label>
                </div>
              )}
            />
            <Link
              href="/reset-request"
              className="text-sm font-medium text-[#8C311ECC] hover:text-[#8C311ECC]/60"
            >
              Forgot password?
            </Link>
          </div>
          <Button
            type="submit"
            className="w-full min-h-[45px]"
            disabled={loading}
          >
            {pending
              ? "Signing In..."
              : isLoading
              ? "Just a second..."
              : "Sign In"}
          </Button>
        </form>
      </Form>
      <div className="text-center text-sm">
        <span className="text-gray-600">New to our platform?</span>{" "}
        <Link
          href={callback ? `/sign-up?callback=${callback}` : "/signup"}
          className="font-medium text-primary-blue hover:text-primary-blue/80"
        >
          Sign Up Here
        </Link>
      </div>
    </>
  );
}
```

---

### 4. Login Action (`src/action/auth/login.ts`)

Handles form validation and mock authentication.

```ts
"use server";

import { loginFormSchema, LoginFormValues } from "@/schemas/auth";

export async function loginAction(data: LoginFormValues) {
  const parsedData = loginFormSchema.safeParse(data);

  if (!parsedData.success) {
    return {
      success: false,
      message: "Invalid form input. Please check your email and password.",
    };
  }

  const { email, password } = parsedData.data;

  // Replace this logic with your actual authentication system
  if (email !== "user@example.com" || password !== "securePassword123") {
    return {
      success: false,
      message: "Incorrect email or password.",
    };
  }

  return {
    success: true,
    message: "Login successful.",
  };
}
```

---

### 5. Schema Definition (`src/schemas/auth/index.ts`)

Zod-powered schema for type-safe validation.

```ts
import { z } from "zod";

export const loginFormSchema = z.object({
  email: z
    .string()
    .min(1, { message: "Email is required" })
    .email({ message: "Invalid email address" }),
  password: z.string().min(1, { message: "Password is required" }),
  rememberMe: z.boolean().optional(),
});

export type LoginFormValues = z.infer<typeof loginFormSchema>;
```

---

## 🚀 Getting Started

1. **Install dependencies** (see [Installation](#installation)).
2. **Add the provided files** to your Next.js project, mirroring the structure above.
3. **Run the development server:**

   ```bash
   npm run dev
   # or
   yarn dev
   ```

4. **Open** `http://localhost:3000/login` in your browser.

---

## 🛠️ Next Steps

Need more features?

- Signup system
- Protected routes with middleware
- JWT/session integration

Let me know, and I can generate boilerplate for these as well!

---

**Happy coding!** 🚀
