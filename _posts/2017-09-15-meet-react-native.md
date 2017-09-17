---
layout: single
title:  "Meet React Native"
date:   2017-09-15 20:31:00 +0700
permalink: '/post/react/meet-react-native'
categories: 
  - react
excerpt: This post tell about preparation to build application with **React Native**.
---

{% include toc title="Contents" %}

I've just heard about [**React Native**](https://facebook.github.io/react-native/) from my friend about 3 days ago. 
I was exciting -- that we can just write our code in Javascript to create
a native mobile apps that works both for **Android** and **IOS**. 

This is also my first experience to work with [**React**](https://facebook.github.io/react/), I never know about [**JSX syntax**](https://facebook.github.io/react/docs/introducing-jsx.html) before this. I was impressed that HTML syntax and Javascript can live together -- don't laugh! :)

Let me share you bit about my background. I'm familiar with [**Vuejs**](https://vuejs.org/). I love it -- it helps me a lot to finish my projects successfuly, 
thus I created [**VueTyped**](https://vue-typed.github.io/vue-typed/) and [**VueTypedUI**](https://vue-typed.github.io/vue-typed-ui/) to help me develop apps with **Vuejs** using [**Typescript**](http://www.typescriptlang.org/).

Today I'd like to share you my first experience developing application using **React Native**.
I've created a simple login application to satisfy my couriousity.


## Development Environment

First thing first is to setup development environment. I'm using [**Visual Studio Code**](https://code.visualstudio.com/) as my editor 
running on Windows 10 64 bit.

### Yarn

This is optional, but recommended to make download npm packages faster. Install it from the [official website](https://yarnpkg.com/en/).


### ESLint

Must have! Code must be clean and follow the standard rule.

Install [ESLint](https://eslint.org/) globally:

{% highlight bash %}
npm i -g eslint
{% endhighlight %}

Install [**VS Code ESLint extension**](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) and enable it.


### REACT_EDITOR

When using Android Emulator to run the application, everytime something went wrong
a red box is displayed on emulator screen containing error stacks
so we can track where is the problem came from.

You'll need to set `REACT_EDITOR` environment variable to be able to click on any stack frame 
from the red box in emulator to jump to source file.

```
REACT_EDITOR = <editor you are using>
```

You can set it via ~/.bashrc or ~/.zshrc depending on which shell you use.
Another approach is to use `cross-env`. 

First, download the package:

```bash
yarn add cross-env --save-dev
```

And modify `start` script in `package.json` to:

```json
"start": "cross-env REACT_EDITOR=code react-native-scripts start"
```

This will open **vscode** when you click any stack frame on the Red box.


## First Run

### Install the CLI

```bash
npm install -g create-react-native-app
```

### Create new project

```bash
create-react-native-app AwesomeProject
```

### Setup Visual Studio Code Environment

Go to `AwesomeProject` folder and open **Visual Studio Code**.


#### ESLint

I'm using **rallycoding** ruleset. This validation rules package is famous among **React Native** developers -- so I decided to join the club without asking many questions :).

I installed this package locally as development package.

```bash
yarn add eslint-config-rallycoding --dev
```

Now configure **ESLint** in `.eslintrc` file like this:

```json
{
  "extends": "rallycoding",
  "rules": {
    "semi": 0,
    "arrow-body-style": "off",
    "import/no-extraneous-dependencies": ["error", { "devDependencies": true }]
  },
  "env": {
    "jest": true
  },
  "globals": {
    "fetch": false
  }
}
```

This configuration will make:

- The **rallycoding** ruleset active.
- Remove semicolon in the end of statement is allowed.
- Won't force you to use a body style when creating arrow function.
- Allowing import from `dev-dependencies`. Notice that in `App.test.js` we needs import `react-test-renderer` module which is part of `dev-dependencies`.
- The [`jest`](https://facebook.github.io/jest/) testing environment should works fine.
- You can invoke `fetch` function everywhere since it's a React Native global function.


#### Prettier - JavaScript formatter

I'm using this extention to keep the code always beauty. Just install it from [marketplace](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode).

Configure **Prettier** in `.prettierrc` file:

```json
{
  "semi": false,
  "singleQuote": true
}
```

This configuration will do:
- Remove semicolon in the end of every statement.
- Always use single-quote in the import statement, this will satisfies the standard rule of ESLint.

Alright, now open `App.js` and press `SHIFT`+`ALT`+`F` -- the code should be looks nice now.


#### .vscode/settings

Ensure you turn off `javascript.validate.enable` under `.vscode/settings` otherwise type declaration won't work.

This is my `.vscode/settings` configuration looks like:

```json
{
  "eslint.enable": true,
  "jshint.enable": false,
  "javascript.validate.enable": false,
  "tslint.enable": false
}
```


### Running application in Emulator

Actually we can use our device (Android/IOS phone) to run our application using [**Expo**](https://expo.io/) as described in [**React Native official site**](https://facebook.github.io/react-native/docs/getting-started.html#content).
But for convenience, I prefer to use an emulator.

Make sure that emulator is running. Im using Emulator from [**Android Studio**](https://developer.android.com/studio/index.html).

```bash
cd AwesomeProject
yarn start
```

Press `a` to run application in the Emulator.

![Running in emulator](/assets/images/2017-09-15-meet-react-native/running-emulator.png)

Once *Building Javascript bundle..* done, you'll see text *Open up App.js to start working on your app!...*.

Okay, now open **Visual Studio Code** and try to change text `Open up App.js to start working on your app!` inside `App.js`.
Save it and the emulator is automatically updated with new text. Awesome! The **Hot Reload** is working!

The next step is to start building the application. I wrote another post for this part, please continue reading [here](/post/react/build-login-app-with-react-and-redux).