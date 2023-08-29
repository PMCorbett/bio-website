---
title: Reverse TDD
publishDate: 2023-08-29 00:00:00
img: /assets/reverse2.webp
img_alt: UNO Reverse
description: |
  How to refactor something that doesn't have good tests
tags:
  - Testing
  - Principles
  - Programming
  - Techniques
---

# What is Reverese TDD

Firstly, TDD is Test Driven Development, its the process of writing tests first and code second. Its an extremly useful way of writing code as it ensures that you write code that is easy to test, code that is largely readable and code that doesn't do unexpected things.

Reverse TDD is trying to recreate the advantages of TDD when the code is already written, but the tests are missing, or rubbish.

## The method

1. Have the code that you want to test & refactor
2. Stop looking at the code you want to refactor
3. Write a failing test
4. Write the smallest amount of code to satisfy the test
5. Write another failing test
6. Write the smallest amount of code to satisfy the test

... repeat

## The method - continued

Obviously this is just the method for doing TDD and I am being facetious. The method is really:

1. Have the code that you want to test & refactor
2. Stop looking at the code you want to refactor
3. Write a failing test
4. Copy & Paste the smallest amount of code (from old to new) to satisfy the test
5. Write another failing test
6. Copy & Paste the smallest amount of code (from old to new) to satisfy the test

... repeat

## Whats important

Its much easier to do this if you are not the person that originally wrote the code, or if you have a terrible memory. You should try and forget about the previous implementation as much as possible and just write the implementation you wish existed. I call this "writing your dream-code".

This will allow you to separate the implementation detail from the business logic, so you can create a nice implmentation, without having to rewrite the business logic.

## Example

The tests kickstarts again

So say you have a large react provider that has lots of internal logic that is complicated, and no direct tests.

```tsx
const AuthProvider: React.FC<AuthProviderProps> = ({ children, ...config }) => {
  const [token, setToken] = useState<string>();

  useEffect(() => {
    if (!token) {
      const storedToken = localStorage.getItem(config.tokenName);

      if (storedToken) {
        setToken(storedToken);
      }
    }
  }, []);

  useEffect(() => {
    localStorage.setItem(config.tokenName, token || "");
  }, [token]);

  const getRefreshedToken = async () => {
    const refreshToken = window.localStorage.getItem(config.refreshTokenName);

    if (refreshToken) {
      const response = await requestRefreshedToken({
        url: config.d2cCognitoAdapterUrl,
        tenant: config.tenant,
        refreshToken,
      });

      setToken(response.jwt);
      window.localStorage.setItem(
        config.refreshTokenName,
        response.refresh_token
      );

      return response.jwt;
    }

    return "";
  };

  const clear = () => {
    setToken(undefined);
  };

  const signOut = () => {
    clear();
    window.localStorage.removeItem(config.tokenName);
    window.localStorage.removeItem(config.refreshTokenName);
  };

  return (
    <Context.Provider
      value={{
        token,
        isVerifiedUser: !!token,
        clear,
        getRefreshedToken,
        signOut,
      }}
    >
      {children}
    </Context.Provider>
  );
};
```

So looking at this example, you have to make some decisions about how you want to refactor this. The major decision, is should you write util helpers or hooks? I think for this example we'll refactor this to use hooks.

So theres two responsibilities in this component currently, access tokens and refresh tokens. So lets say we want two hooks. For the example, I'll start with just the access token hook.

### Step 1: Identify the dream interface

Looking at tokens, it appears that we have them in state, and we read & write them to local storage. We also need a way of clearing them out.

So my dream interface is probably something like

```js
const { token, setToken, clearToken } = useStoredToken(config);
```

### Step 2: Write a failing test

I'm going to use `renderHook` from react testing library. Sometimes I like to write a whole host of failing tests, and then knock them off one by one, other times i write them a test at a time. I think thats a personal preference, and no judgement from me on how you choose to do it.

```ts
describe("useStoredToken", () => {
  describe("when there is a token in local storage", () => {
    test("it is set from the get-go", () => {
      localStorage.setItem(config.tokenName, "TEST_TOKEN");

      const { result } = renderHook(() => useStoredToken());

      expect(result.token).toEq("TEST_TOKEN");

      localStorage.removeItem(config.tokenName);
    });
  });

  describe("when there is no token in local storage", () => {
    test("it is undefined", () => {});
  });

  describe("when the token is updated", () => {
    test("it updates local storage", () => {});
  });

  describe("when the token is cleared", () => {
    test("it removes the item from local storage", () => {});
  });
});
```

### Step 3: Satisfy that code by either writing or copy pasting code

```ts
const useStoredToken = (config) => {
  const [token, setToken] = useState<string>();

  useEffect(() => {
    if (!token) {
      const storedToken = localStorage.getItem(config.tokenName);

      if (storedToken) {
        setToken(storedToken);
      }
    }
  }, []);

  return { token };
};
```

### Step 4: Any possible refactors?

Its pretty clear here that we are rehashing useState's initial state ability, and we can use that directly like this:

```ts
const useStoredToken = (config) => {
  const [token, setToken] = useState<string>(
    localStorage.getItem(config.tokenName)
  );

  return { token };
};
```

Ooooooh yeahh

### Step 5: Write a failing test

```ts
describe("when there is no token in local storage", () => {
  test("it is undefined", () => {
    const { result } = renderHook(() => useStoredToken());

    expect(result.token).toEq(undefined);
  });
});
```

### Step 6: Satisfy that code

Damn, it already passes, moving on.

### Step 7: Write a failing test

```ts
describe("when the token is updated", () => {
  test("it updates local storage", () => {
    localStorage.setItem(config.tokenName, "TEST_TOKEN");

    const { result } = renderHook(() => useStoredToken());

    act(() => {
      result.setToken("NEW_TOKEN");
    });

    expect(result.token).toEq("NEW_TOKEN");
    expect(localStorage.getItem(config.tokenName)).toEq("NEW_TOKEN");

    localStorage.removeItem(config.tokenName);
  });
});
```

### Step 7: Satisfy

```ts
const useStoredToken = (config) => {
  const [token, setToken] = useState<string>(
    localStorage.getItem(config.tokenName)
  );

  useEffect(() => {
    localStorage.setItem(config.tokenName, token || "");
  }, [token]);

  return { token, setToken };
};
```

### Step 8: Refactor?

Its probably ok there

### Step 9: Write a failing test

```ts
describe("when the token is cleared", () => {
  test("it removes the item from local storage", () => {
    localStorage.setItem(config.tokenName, "TEST_TOKEN");

    const { result } = renderHook(() => useStoredToken());

    act(() => {
      result.clearToken();
    });

    expect(result.token).toEq(undefined);
    expect(localStorage.getItem(config.tokenName)).toEq(undefined);

    localStorage.removeItem(config.tokenName);
  });
});
```

### Step 10: Satisfy

Here we spot in the original code that there is no direct equivalent of clearToken, because of the coupling between access tokens and refresh tokens. So we have to actually write it but its only 3 lines, so we'll manage.

```ts
const useStoredToken = (config) => {
  const [token, setToken] = useState<string>(
    localStorage.getItem(config.tokenName)
  );

  useEffect(() => {
    localStorage.setItem(config.tokenName, token);
  }, [token]);

  const clearToken = () => {
    setToken(undefined);
  };

  return { token, setToken };
};
```

### Step 11: Put it back in the original place

So lets look at our old auth provider again:

```tsx
const AuthProvider: React.FC<AuthProviderProps> = ({ children, ...config }) => {
  const [token, setToken] = useState<string>();

  useEffect(() => {
    if (!token) {
      const storedToken = localStorage.getItem(config.tokenName);

      if (storedToken) {
        setToken(storedToken);
      }
    }
  }, []);

  useEffect(() => {
    localStorage.setItem(config.tokenName, token || "");
  }, [token]);

  const getRefreshedToken = async () => {
    const refreshToken = window.localStorage.getItem(config.refreshTokenName);

    if (refreshToken) {
      const response = await requestRefreshedToken({
        url: config.d2cCognitoAdapterUrl,
        tenant: config.tenant,
        refreshToken,
      });

      setToken(response.jwt);
      window.localStorage.setItem(
        config.refreshTokenName,
        response.refresh_token
      );

      return response.jwt;
    }

    return "";
  };

  const clear = () => {
    setToken(undefined);
  };

  const signOut = () => {
    clear();
    window.localStorage.removeItem(config.tokenName);
    window.localStorage.removeItem(config.refreshTokenName);
  };

  return (
    <Context.Provider
      value={{
        token,
        isVerifiedUser: !!token,
        clear,
        getRefreshedToken,
        signOut,
      }}
    >
      {children}
    </Context.Provider>
  );
};
```

So lets use our new hook and busy with the delete:

```tsx
const AuthProvider: React.FC<AuthProviderProps> = ({ children, ...config }) => {
  const { token, setToken, clearToken } = useStorageToken(config);

  const getRefreshedToken = async () => {
    const refreshToken = window.localStorage.getItem(config.refreshTokenName);

    if (refreshToken) {
      const response = await requestRefreshedToken({
        url: config.d2cCognitoAdapterUrl,
        tenant: config.tenant,
        refreshToken,
      });

      setToken(response.jwt);
      window.localStorage.setItem(
        config.refreshTokenName,
        response.refresh_token
      );

      return response.jwt;
    }

    return "";
  };

  const signOut = () => {
    clearToken();
    window.localStorage.removeItem(config.refreshTokenName);
  };

  return (
    <Context.Provider
      value={{
        token,
        isVerifiedUser: !!token,
        clear: clearToken,
        getRefreshedToken,
        signOut,
      }}
    >
      {children}
    </Context.Provider>
  );
};
```

OK, so thats a bit better. If we did the same for the refresh token parts, this context provider would become entirely trivial, maybe I'll write that bit up later.
