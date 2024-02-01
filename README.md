In this blog post I'll show you how to setup a Node 4 Max project with TypeScript. The project will live in a Git repository for versioning and will feature `build` and `watch` scripts for hot reloading.

## Requirements

### Install Max MSP

Make sure you have the latest version of Max MSP installed.
At the time of writing, the latest version is 8.6.0, which contains a local instance of NodeJS 20.6 ([Release Notes](https://cycling74.com/releases/max/8.6.0))

To download Max MSP go to [https://cycling74.com/downloads](https://cycling74.com/downloads)

### Install NodeJS

Even though Max MSP bundles NodeJS internally, you still need to run node on your terminal to setup the project. I recommend [installing NVM](https://github.com/nvm-sh/nvm?tab=readme-ov-file#installing-and-updating) to manage your node versions.
Once you have NVM installed, run the following command to install the version we need.

```bash
nvm install 20.6
```

## Create Max Project

To simplify the management of multiple patchers and assets, Max MSP introduces projects. A project can be created by opening the Max MSP app and go to _File > New Project_.

Now you should see a floating panel for your project.
Click on the plus icon on the bottom and create a new pather file called `main.maxpat`;

Once you have created the patcher, your project folder should look similar to this:

```
.
├── n4m-typescript.maxproj
└── patchers
    └── main.maxpat
```

## Setup Node environment

Since this is going to be a Node project, we need to create a `package.json` file, where our scripts and dependencies will be configured.

### Check node version

First, check if your are running the correct Node version:

```bash
node -v
```

The command should print out `v20.6.1`.
If see a different version and you have installed NVM, you can switch to the right version with:

```bash
nvm use 20.6
```

### Create package.json

To create the file, you can run this command inside the project directory.

```bash
npm init -y
```

Now you should have a package.json file that looks like this:

```bash
{
  "name": "n4m-typescript",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

### Set engine version

We want to make sure that the project is running the correct Node version. To do so, we can add an `engines` property to the `package.json` file. 

```diff
+  "engines": {
+     "node": ">=20.6"
+  },
```

This config makes node and npm throw an error if the node version is not compatible with the one specified.

### Setup TypeScript

To make TypeScript work with Node and Max we need to install some dev dependencies:

- `typescript` is the compiler for TypeScript
- `@tsconfig/recommended` contains the recommended TypeScript configuration
- `@types/node` contains the type definitions for Node
- `@types/max-api` contains the type definitions for Max API bindings

Run this command to install the dependencies:

```bash
npm i --save-dev typescript @types/node @tsconfig/recommended @types/max-api
```

After the installation, you should see a new `node_modules` folder and a `package-lock.json` file.

- `node_modules`: folder that contains all the dependencies
- `package-lock.json`: file that contains the exact version of the dependencies installed

Additionally, the dependencies will be added to the `package.json` file like this:

```diff
+  "devDependencies": {
+    "@tsconfig/recommended": "^1.0.3",
+    "@types/max-api": "^2.0.3",
+    "@types/node": "^20.11.15",
+    "typescript": "^5.3.3"
+  }
```

Now, let's create a simple `tsconfig.json` for our project.

```json
{
  "extends": "@tsconfig/recommended/tsconfig.json",
  "compilerOptions": {
    "noEmit": true,
    "module": "ES2015",
    "moduleResolution": "Bundler"
  },
  "include": ["src/**/*"]
}
```

Since we have specified `noEmit`, TypeScript will perform type checking without generating any JavaScript files.
To transform TypeScript into JavaScript we need to use a bundler. In this case we are going to use [ESBuild](https://esbuild.github.io/) because is fast and easy to configure.

Bundling is necessary because Max MSP doesn't support ES Modules very well. ESBuild will concatenate all the JavaScript modules that we need into a single file that we can load in Max.

### Setup Bundling

To install ESBuild, run this command:

```bash
npm i --save-dev esbuild
```
