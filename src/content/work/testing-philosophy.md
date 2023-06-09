---
title: Testing Philosophy
publishDate: 2023-06-09 00:00:00
img: /assets/aristotle.webp
img_alt: Aristotle
description: |
  The attitude I take towards testing and writing code to make tests easy
tags:
  - Testing
  - Principles
  - Programming
  - Base ideas
---

# Testing Philosophy

I want to do a clever joke about classical philosophy and testing here, but I am not clever. So... just think I was funny and therefore I was.

Tests should be easy, tests should be fast, tests should be reliable and tests should make everything else easier.

## Why are tests sometimes hard?

Tests sometimes become difficult to write when we have code that is too interdependent and tightly coupled.

This means sometimes we change something in one file, and a spec file 400 miles away explodes. And its impossible to immediately tell why, so then you have to go and open the damn thing, find out whats wrong, fix it, then try and remember what you were trying to do in the first place and then rinse and repeat.

This is annoying and slows down development, splits focus and, I find, often leads me to just trying to get the tests to pass and not giving the code enough thought

## How do we make tests easier?

I used to be very much a card-carrying TDD evangelist. More recently I have mellowed on that, especially when it comes to writing visual components. However, I do think the principles that drive TDD are worth remembering, whether or not we are actually test driving the code. Also, everyone should know how to test drive, as its a fantastically useful tool to have in your arsenal.

If we stick to a few principles we should be able to operate with context that we know exactly what will change when we change something. Which means we can spend more of our brain-power on the actual problem and not really have to think about the tests.

### Inject Dependencies

**This is the important one**

Injecting dependencies is a silly and complicated name for what is a simple principle. Just make functions take everything they need as arguments, return some data. Do not rely on globals, do not directly use 3rd party libraries, do not directly use some other function we've written, don't make calls to external APIs.

Well... I hear you ask... I need to do those things, thats the point.

And yes, but your function doesn't need to know about it. This explanation has got away from me, here are some examples that should help.

#### Example 1 - avoid mocking env

```js
import { env } from "./env";

const appendCheeseToAnObject = (anObject) => ({
  ...anObject,
  cheese: env.CHEESE,
});
```

So this is attaching a cheese attribute from the process environment to the object.

This means our test will look something like this

```js
jest.mock("./env.ts", () => ({
  env: {
    CHEESE: "cheddar",
  },
}));

test("it appends the cheese to the object", () => {
  const result = appendCheeseToAnObject({ beans: "heinz" });

  expect(result).toEqual({ beans: "heinz", cheese: "cheddar" });
});
```

At the moment this seems fine, but imagine if there 500 lines between the mock and the test, upon reading this file you might be quite confused as to where cheddar has come from.

So instead we can keep our functions entirely pure and write this:

```js
const appendCheeseToAnObject = (cheese, anObject) => ({
  ...anObject,
  cheese: cheese,
});

test("it appends the cheese to the object", () => {
  const result = appendCheeseToAnObject("cheddar", { beans: "heinz" });

  expect(result).toEqual({ beans: "heinz", cheese: "cheddar" });
});
```

And then we've not had to mock anything out, our function is just data-in --> data-out and our tests are super clear.

But... that doesn't seem all that powerful, so what if we have a more complicated function

#### Example 2 - don't mock (or nock) requests

So we have a more complicated function that builds a little object and then posts it to an endpoint, with some headers.

```js
export const registerUser = async (emailAddress: string) => {
  const authServiceKey = await decryptSecret(env.AUTH_SERVICE_API_KEY);

  const { data, status } = await axios.post(
    `${env.AUTH_SERVICE_URL}/v2/magic-links/register`,
    {
      emailAddress: emailAddress,
    },
    {
      headers: {
        Authorization: authServiceKey,
      },
    }
  );

  return data;
};
```

So, thats a bit more true to life, what would the tests look like for that?

```js
jest.mock("./env.ts", () => ({
  env: {
    AUTH_SERVICE_API_KEY: "apiKeyGoesHere",
    AUTH_SERVICE_URL: "https://not-the-old-value.com",
  },
}));

beforeEach(() => {
  jest.clearAllMocks();
  nock.cleanAll();
});

afterEach(() => {
  jest.clearAllMocks();
  nock.cleanAll();
});

const testEmail = "person@place.com";

const validRegistrationResponse = {
  jwt: "definitelySuperValidToken",
  refresh_token: "refreshToken",
  userIdentifier: "88888888-4444-4444-4444-121212121212",
};

it("should send registration request with correct values", async () => {
  const scope = nock(env.AUTH_SERVICE_URL)
    .post("/v2/magic-links/register", {
      emailAddress: testEmail,
    })
    .reply(200, validRegistrationResponse);

  await registerUser(testEmail);

  expect(scope.isDone()).toBeTruthy();
});
```

So, thats a nightmare. There are million set up and tear down steps. If any part of the function isn't doing what we suspect, the only error we get is false is not truthy. We can do better.

```js
export const registerUser = async (
  emailAddress,
  authServiceKey,
  authServiceUrl,
  post
) => {
  const { data, status } = await post(
    `${authServiceUrl}/v2/magic-links/register`,
    {
      emailAddress: emailAddress,
    },
    {
      headers: {
        Authorization: authServiceKey,
      },
    }
  );

  return data;
};

it("should send registration request with correct values", async () => {
  const testEmail = "person@place.com";
  const postSpy = jest.spy();

  await registerUser(testEmail, "key", "url.com", postSpy);

  expect(postSpy).toHaveBeenCalledWith(
    "url.com/v2/magic-links/register",
    {
      emailAddress: emailAddress,
    },
    {
      headers: {
        Authorization: "key",
      },
    }
  );
});
```

Writing the test in this way means we can see the whole test in one bite, and we get a detailed error message if any part of our request is wrong.

### Write small unitary pieces of code

If you write code in little pieces that adhere to the dependency injection principle, you will find that you naturally tend towards writing small chunks of easily testable code. And then these can simply be combined up into a big function that does lots.

### Be prepared to throw tests away

If you write little tests to test your little unitary chunks, sometimes once you've got them all working together you can have a little scan back and realise you are testing the same thing several times, from different levels, so you might at that point decide to delete some lower level tests. That is ok. They were just tools, they helped you get to this place, and if they are no longer of any use, let go.

### Have short feedback loops

I want to start an indie band called The Short Feedback Loops.

The most important part of testing I think is to write the tests as early as feasable (write them first if possible), and then go through the red-green cycle as often and as quickly as possible. If you find yourself 10 or 15 minutes from the last time you ran the test and you are still writing code, I always feel lost and alone, like a child in the woods, without the protection of my tests.

### Four phase tests

`Setup -> Exercise -> Assert -> Teardown`

or

`before()/beforeAll() -> something in a describe block 400 lines ago -> assert -> after() -> afterAll()`

We spend more time reading tests than writing tests over the whole life of the test file. So tests should be writting to read.

When a person comes to a massive spec days, weeks, or months after it was written, they will not know the exact context of what the original author intended, so it will help if they can see the whole context of each assertion in one eye-zone (I feel like their should be a word for that, but I can't think of it).

So four phase tests look like this:

```js
test("I am an elephant", () => {
  // setup
  setupAMockService();
  const name = "Nellie";

  // exercise
  const result = isElephant(name);

  // assert
  expect(result).toEqual(true);

  // teardown
  tearDownAMockService();
});
```

## Conclusion

Testing is super important because it helps us rely on our code.

When tests stop us from writing our code they are performing the opposite of their intended functions.

Our tests should be readable, reliable and understandable enough that we should be able to know exactly what will change when we make a code change.

The real trick is not working on making our tests better, but by changing the way we write code, to make it more testable.

Not only will it make the tests easier to write, it will, by natural consequence make our code more isolated, more composable and cleaner.
