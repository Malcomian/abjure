# Abjure

This package is a command line interface utility that can transpile a source directory into a build directory but with some special features in regards to javascript files. It will look for special comment lines and either "comment in" or "comment out" those sections based on some simple patterns. The main idea is that this feature allows for special imports between javascript files that wouldn't normally be able to import or export their code. This allows the developer a type of on-demand special intellisense.

## Setup

This project can be installed globally as a CLI utility using `npm install -g abjure`.

The easiest way to use this package is to register it as a build script in your `package.json` file. For example, if you want to target a `src` folder and transpile it to a `build` folder, use the following script:

```json
{
  "scripts": {
    "build": "abjure build \"./src\" \"./build\""
  }
}
```

Then, running `npm run build` will transpile everything in the `src` folder into the `build` folder in the root of your project.

## Usage

For inserting and removing multiple lines, start with `/*+++*/` for inserting code and `/*---*/` for removing code. End these sections with `/*...*/`. Everything between these lines will either be inserted (whereby double slash comments will be removed from the start of the line) or removed (whereby double slash comments will be added to the start of the line). Note that these lines only need to start with these patterns (ignoring any leading space or tab characters), which can be helpful for extra comments.

For inserting and removing single lines, start with `// /*++*/` for inserting code and `/*--*/` for removing code. These sections don't require any ending pattern.

For inserting and removing inline code, start with `/*+*/ /*` for inserting code and `/*-*/` for removing code. End these sections with `/*.*/`. These patterns can be inserted anywhere in a line of code.

**Example:** look at the following example source code:

```javascript
// multiline insertion
/*+++*/
// console.log('this will be inserted at build!')
/*...*/

// multiline removal
/*---*/
console.log('this will be removed at build!')
/*...*/

// single line insertion
// /*++*/ console.log('this will be inserted at build!')

// single line removal
/*--*/ console.log('this will be removed at build!')

// inline insertion and removal
console.log(/*+*/ /* 'this will be inserted at build!' /*.*/ /*-*/ 'this will be removed at build!' /*.*/)
```

When the project builds, the code will turn into the following:

```javascript
// multiline insertion
/*+++*/
console.log('this will be inserted at build!')
/*...*/

// multiline removal
/*---*/
// console.log('this will be removed at build!')
/*...*/

// single line insertion
/*++*/ console.log('this will be inserted at build!')

// single line removal
// /*--*/ console.log('this will be removed at build!')

// inline insertion and removal
console.log(/*+*/ 'this will be inserted at build!' /*.*/ /*-*/ /* 'this will be removed at build!' /*.*/)
```

As a side note, I thought it'd be worthwhile to mention that - for convenience - these patterns are meant to be typed using only one hand on a number pad.

## Known Issues

One issue with the single line and inline methods is that they will usually behave poorly with anything but the default vscode code formatter for javascript files, as most code formatters will push code that appears after a block quote, which is anything starting with a `/*` and ending with a `*/`, to a new line.
