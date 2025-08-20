Perfect! Here's a **TypeScript-based** solution that centralizes permissions in a **React + Redux Toolkit app**, supports **database-driven read/edit access**, and keeps everything type-safe.

---

## 🧱 Overview of What You'll Build:

1. **Permissions model** with `read`/`edit` access.
2. **Redux slice** to hold the current user and their permissions.
3. **Type-safe utility hook** to check permissions (`hasPermission`, `canEdit`, etc.).
4. **(Optional)** Components/HOCs to simplify usage.

---

## 1. 🔐 Define Types and Permission Model

Let's assume your API sends something like:

```json
{
  "user": { "id": "u1", "name": "John Doe" },
  "permissions": [
    { "resource": "user", "access": "read" },
    { "resource": "dashboard", "access": "edit" }
  ]
}
```

### TypeScript types:

```ts
// types/permissions.ts

export type AccessType = 'read' | 'edit';

export interface Permission {
  resource: string;
  access: AccessType;
}

export interface User {
  id: string;
  name: string;
}

export interface AuthState {
  user: User | null;
  permissions: Permission[];
}
```

---

## 2. 🧠 RTK Auth Slice

```ts
// store/authSlice.ts

import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import { AuthState, Permission, User } from '../types/permissions';

const initialState: AuthState = {
  user: null,
  permissions: [],
};

interface SetUserPayload {
  user: User;
  permissions: Permission[];
}

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    setUser(state, action: PayloadAction<SetUserPayload>) {
      state.user = action.payload.user;
      state.permissions = action.payload.permissions;
    },
    logout(state) {
      state.user = null;
      state.permissions = [];
    },
  },
});

export const { setUser, logout } = authSlice.actions;
export default authSlice.reducer;
```

---

## 3. 🧩 usePermissions Hook (Central Access Logic)

```ts
// hooks/usePermissions.ts

import { useSelector } from 'react-redux';
import { RootState } from '../store/store';
import { AccessType } from '../types/permissions';

export function usePermissions() {
  const permissions = useSelector((state: RootState) => state.auth.permissions);

  const hasPermission = (resource: string, access: AccessType): boolean => {
    return permissions.some(
      (perm) =>
        perm.resource === resource &&
        (perm.access === access || (access === 'read' && perm.access === 'edit')) // edit implies read
    );
  };

  const canRead = (resource: string): boolean => {
    return hasPermission(resource, 'read');
  };

  const canEdit = (resource: string): boolean => {
    return hasPermission(resource, 'edit');
  };

  return { hasPermission, canRead, canEdit };
}
```

> ✅ Logic note: `edit` access also implies `read`, hence that logic in `hasPermission`.

---

## 4. 🧪 Example Usage in Components

```tsx
// components/UserPage.tsx
import React from 'react';
import { usePermissions } from '../hooks/usePermissions';

const UserPage: React.FC = () => {
  const { canRead, canEdit } = usePermissions();

  if (!canRead('user')) {
    return <div>You do not have access to view this page.</div>;
  }

  return (
    <div>
      <h1>User Information</h1>
      {canEdit('user') ? (
        <button>Edit User</button>
      ) : (
        <p>Read-only mode</p>
      )}
    </div>
  );
};

export default UserPage;
```

---

## 5. 🧰 Redux Store Setup (Boilerplate)

```ts
// store/store.ts
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './authSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

---

## ✅ Backend-Driven Permissions (Optional Note)

After login or token validation, you should fetch user permissions from the backend:

```ts
// Example API response
{
  user: { id: 'u1', name: 'Admin' },
  permissions: [
    { resource: 'user', access: 'edit' },
    { resource: 'dashboard', access: 'read' },
  ]
}
```

Store it using:

```ts
dispatch(setUser(apiResponse));
```

Now your access control system will adapt dynamically to what the backend defines.

---

## 📦 Bonus: Central `Resources` Enum (optional for safety)

```ts
// constants/resources.ts
export enum Resources {
  USER = 'user',
  DASHBOARD = 'dashboard',
  SETTINGS = 'settings',
}
```

Then use like:

```ts
canEdit(Resources.USER)
```

---

## ✅ Summary

| Goal                   | Solution                     |
| ---------------------- | ---------------------------- |
| Central access logic   | `usePermissions` hook        |
| Backend-driven control | Store `permissions[]` in RTK |
| Type-safety            | With enums + TS types        |
| Edit/read distinction  | Built into `hasPermission()` |

---

Would you like me to generate a working example app (GitHub link or downloadable codebase)?
