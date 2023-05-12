---
title: The Adapter Pattern
publishDate: 2023-05-12 00:00:00
img: /assets/adapter.webp
img_alt: The Adapter Pattern
description: |
  An introduction to the adapter pattern, where it might be used, and how it helps
tags:
  - Adapter Pattern
  - Principles
  - Programming
  - Base ideas
---

# The Adapter Pattern

## Useless Introduction

A quote often misattributed to Charles Darwin is: "It is not the strongest of the species that survives, nor the most intelligent that survives. It is the one that is most adaptable to change."

This is not particularly useful or relevant, as the adapter pattern often has nothing to do with change.

## Potentially Useful Introduction

The adapter pattern is about decoupling. Its about having one thing here, and another thing there and letting that first thing use the second thing without becoming reliant on it. Too many things in that sentence. I'll try again.

The adapter pattern is about separating code into distinct blocks and particularly about separating "our" code from "theirs". Usually you would think about "our" code as being code we are writing, and "their" code being something we don't control like a browser interface. And indeed that is a very useful application for the adapter pattern. However, you can also think of "our" code as being code we are writing now, and "their" code as being code that isn't written yet, is being written by another team, or we don't know enough about to use.

## Write your dream-code

We all need, once in a while, to live out our dreams and fantasies when we code. Often that isn't possible, but with the adapter pattern, you can write whatever you want, hooray!!!

## Time for an example, a real world example:

Once upon a time I had to maintain a Progressive Web App that then got repurposed as a Cordova Phonegap app temporarily. Naturally, 6 years later the Cordova app is still going, such is the nature of temporary things that work.

So... I had an app that had to work in modern browsers, and in Cordova.

I had to write a component that checked whether the user had granted camera permissions and then could request permission if they hadn't. Simple enough in principle, in modern browsers you can use the [Permissions Query API](https://developer.mozilla.org/en-US/docs/Web/API/Permissions/query) and in cordova we were using the [Corodva Diagnostic Plugin](https://github.com/dpa99c/cordova-diagnostic-plugin).

If I use these APIs directly in my component, my whole component has to know things about what environment I'm operating in, and then make the correct decisions accordingly.

However, what i can do instead, is just write the code I want to write, and completely ignore the realities of the world around me.

So I write a class that does exactly what I want it to do:

```js
class CameraPermission {
  hasPermission = () => Promise.resolve(true);
  requestPermission = () => Promise.resolve(true);
}
```

Blimey, that was easy.

Now I have a choice about what I want to do first. I could write my component, sort out all the styles, get it all looking perfect; or I could start making my class operate in the real world. Or I can separate the job between the team and one person can do the component, while another writes the adapters.

### Writing an adapter

When writing an adapter, we have two things to remember.

1. The contract the CameraPermission class defines
2. The contract "their" code defines

So first, lets update the class to use an adapter

```js
class PermissionAdapter {
  isPermissionGranted = () => Promise.resolve(true);
  requestPermissionFromUser = () => Promise.resolve(true);
}

class CameraPermission {
  constructor(adapter: PermissionAdapter) {
    this.adapter = adapter;
  }

  hasPermission = () => this.adapter.isPermissionGranted();
  requestPermission = () => this.adapter.requestPermissionFromUser();
}
```

Ok, so its a little bit more complicated. But what we can see is we are deferring to the adapter all the complicated knowledge, and our CameraPermission class is simple.

### Writing another adapter!

Ok, we've done something thats allowed us to continue writing the rest of our app, but it doesn't do anything, so lets write something useful.

```js
class WebPermissionAdapter {
  isPermissionGranted = () =>
    navigator.permissions.query({ name: "camera" }).then((result) => {
      if (result.state === "granted") {
        return true;
      }

      return false;
    });

  requestPermissionFromUser = () => Promise.resolve(true);
}
```

Now we have a real life adapter!

### Lets write another

```js
class CordovaPermissionAdapter {
  isPermissionGranted = () =>
    new Promise((resolve, reject) => {
      cordova.plugins.diagnostic.isCameraAuthorized(resolve, reject);
    });

  requestPermissionFromUser = () => Promise.resolve(true);
}
```

So now we can make a very simple decision when we instantiate the CameraPermission class, as to which adapter we would like to pass in.

```js
const cameraPermission = new CameraPermission(
  navigator.permission
    ? new WebPermissionApdapter()
    : new CordovaPermissionAdapter()
);
```

## So thats all well and good, but why bother?

In this instance, the two external apis are fairly similar so the adapters are not having to do anything particularly clever. However, the adapters can call any number of services in any number of ways and as long as they stick to the contract defined by our class, they will work, and our component will need 0 knowledge of the apis to continue.

It means we can add or change interfaces as the need arises, and if anything in the apis beyond our control changes, we will know exactly where the change needs to happen.

## Decoupling Front end from Back end to increase development speed

Using the adapter pattern means you can, if you desire, defer implementing parts of the code until a time that suits you. If you have, say, a large multi-stage web form to develop, and a slightly confusing, slightly out of date data source to update. You could write the data layer with an adapter, which would allow you to simply fake the data layer until your web form was complete, and then write the adapter afterwards, or in parallel without any conflicts.

### Increase ease of testing

There is an old adage in testing that you shouldn't mock what you do not own. This is a good principle (often), I'd quote the source on it, but I forget who said it first, and google doesn't exist.

The adapter pattern makes this incredibly easy.

When testing your ui components, often the easiest thing to do is test with a fake adapter, the kind we wrote right at the start

```js
class AlwaysTrueAdapter {
  isPermissionGranted = () => Promise.resolve(true);
  requestPermissionFromUser = () => Promise.resolve(true);
}
```

So now when we want to assert things about our component when the user has granted permission, we can simply compose our component with the AlwaysTrueAdapter, and make our assertions.

And if we want to test whether things are being called in the right way, we can make a spy adapter

```js
const isPermissionGrantedSpy = jest.fn();
const requestPermissionSpy = jest.fn();

class SpyingAdapter {
  isPermissionGranted = isPermissionGrantedSpy;
  requestPermissionFromUser = requestPermissionSpy;
}

test("calls isPermissionGranted", () => {
  setupTest({ adapter: new SpyingAdapter() });

  exerciseTheCodeInSomeWay();

  expect(isPermissionGrantedSpy).toHaveBeenCalled(1, "times");
});
```

## Conclusion

The adapter pattern is excellent. Everyone should use it all the time for all things.

Well, its a useful tool to have in your arsenal and it is extremely good at decoupling UI components from messy implementation details.

## Addendum

You don't necessarily have to have more than one thing to interface with in order to use an adapter. Carrying on with the example used here, it seems that the adapters are just replicating methods from the original class. But, the original class could be doing other things too, it could have some internal state and be more complicated, and then the adapter would be helping with Single Responsiblility Principle as well.
