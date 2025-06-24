

# 🔐 Login Authentication System - Next.js

A fully functional login authentication system built with **Next.js App Router**, **React Hook Form**, **Zod**, and **JS-Cookie**. This README guides  developers through setup, structure, and usage.

---

## 📦 Required Packages

First, install all the necessary packages:

```bash
npm install js-cookie lucide-react sonner 
```

```bash
npx shadcn@latest add form input label 
```

```bash
npm i --save-dev @types/js-cookie
```

> If using Tailwind CSS and Next.js 13+/14 (App Router), make sure your project is already set up with Tailwind and app directory.

---

### `app/(auth)/login/page.tsx`

> **Path:** `app/(auth)/login/page.tsx`

```tsx
import dynamic from "next/dynamic";
import Image from "next/image";
import { Suspense } from "react";

const LoginForm = dynamic(() => import("./_components/login-form"));

export default function LoginPage() {
  return (
    <div className="flex min-h-screen">
      {/* Left side - Image */}
      <div className="hidden lg:w-3/5 md:w-1/2 bg-gray-900 lg:block relative">
        <Image
          src="https://files.edgestore.dev/t7diwg54d3s82m9n/wellnessmclear/_public/login.jpg"
          alt="Team meeting"
          fill
          className="object-cover"
        />
      </div>

      {/* Right side - Login form */}
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

### `app/(auth)/login/_components/login-form.tsx`

> **Path:** `app/(auth)/login/_components/login-form.tsx`

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

  if (!isMounted) return;

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
                      className="border-primary border-[1px]  min-h-[45px]"
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
                      className="pr-10 border-primary border-[1px]  min-h-[45px]"
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

### `src/action/auth/login.ts`

> **Path:** `src/action/auth/login.ts`

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

### `src/schemas/auth/index.ts`

> **Path:** `src/schemas/auth/index.ts`

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

## ✅ Final Step

After setting up the files:

* Run `npm run dev` or `yarn dev`
* Navigate to `http://localhost:3000/login`

---

Let me know if you'd like me to generate:

* Signup system
* Protected routes with middleware
* JWT/session integration

I'm ready when you are!
