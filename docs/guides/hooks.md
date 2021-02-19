---
sidebarDepth: 0
---

# Hooks functions

- 👉 [Hook beforeResolve()](#👉-before-resolve-hook)
- 👉 [Hook afterResolve()](#👉-after-resolve-hook)

## 👉 Before resolve hook

In addition to creating custom resolvers, it is also possible to use the `beforeResolve` hook to write custom business logic, before an API query is resolved.

```typescript
import { BeforeResolveProps, AuthModes } from './generated/prisma-appsync/client'

app.beforeResolve(async (props: BeforeResolveProps) => {
    // custom business logic here
})
```

The hook function exposes a certain number of properties that you can inspect via the `BeforeResolveProps` type:

```typescript
export type BeforeResolveProps = { 
    action: AuthAction
    args: RequestProps
    authIdentity: AuthIdentityProps
    fields: string[]
    prisma: PrismaClient
    requestSetPaths: any
    subject: string
}
```

The `prisma` parameter is giving direct access to the Prisma Client instance, allowing to build custom database queries directly.

Here is an example of what the various parameters (excl. `prisma`) might contain:

```json
{
    "action": "list",
    "args": {
        "select": {
            "title": true,
            "content": true,
            "comments": {
                "select": {
                    "content": true
                }
            }
        },
        "where": {
            "author": {
                "username": {
                    "equals": "maoosi"
                }
            }
        }
    },
    "authIdentity": {
        "authorization": "API_KEY"
    },
    "fields": [
        "title",
        "content",
        "comments"
    ],
    "requestSetPaths": [
        "list/post/title",
        "list/post/content",
        "list/post/comments/content"
    ],
    "subject": "Post",
}
```

Returning `false` from the hook would cancel the API request operation. Not returning anything or returning `true` would allow to continue the API request operation.

As an example, we can use the hook to only allow queries done using the AppSync API_KEY authorization mode:

```typescript
import { BeforeResolveProps, AuthModes } from './generated/prisma-appsync/client'

app.beforeResolve(async ({ authIdentity }: BeforeResolveProps) => {
    // only authorize queries using the AppSync API_KEY authorization
    return authIdentity.authorization === AuthModes.API_KEY
})
```

See full list of options, methods and types in the [Reference](/reference) section.

## 👉 After resolve hook

The `afterResolve` hook function work very similarly to `beforeResolve`, except that it run AFTER the API query is resolved.

```typescript
app.afterResolve(async (props: AfterResolveProps) => {
    // custom business logic here
})
```

The hook function exposes the same properties as the `beforeResolve` function, with the addition of a `result` parameter:

```typescript{8}
export type AfterResolveProps = {
    action: AuthAction
    args: RequestProps
    authIdentity: AuthIdentityProps
    fields: string[]
    prisma: PrismaClient
    requestSetPaths: any
    result: any
    subject: string
}
```

Here is an example of what the `result` parameter might contain:

```json
"result": [
    {
        "title": "My first post",
        "content": "Hello world!",
        "comments": []
    },
    {
        "title": "My second post",
        "content": "Hello people!",
        "comments": []
    }
]
```

See full list of options, methods and types in the [Reference](/reference) section.
