# Abjure

This package is a command line interface utility that can transpile a source directory into a build directory but with some special features in regards to javascript files. It will look for special comment lines and either "comment in" or "comment out" those sections based on some simple patterns. The main idea is that this feature allows for special imports between javascript files that wouldn't normally be able to import or export their code. This allows the developer a type of on-demand special intellisense.

## Setup

This project can be installed globally as a CLI utility using `npm install -g abjure`. Alternatively, it can be installed locally, typically as a development dependency, with `npm install --save-dev abjure`. To invoke the locally installed version, use `npx abjure [options] [command]`, where `[options]` and `[command]` would be replaced with some basic parameters.

```text
Options:
  -V, --version            output the version number
  -s, --skip               skip greeting
  -l, --log                enable verbose logging
  -h, --help               display help for command

Commands:
  build <source> <target>  build special comments from source directory to target directory
  help [command]           display help for command
```

The easiest way to use this package is to register it as a build script in your `package.json` file. For example, if you want to target a `src` folder and transpile it to a `build` folder, use the following script:

```json
{
  "scripts": {
    "build": "abjure build \"./src\" \"./build\""
  }
}
```

Then, running `npm run build` will transpile everything in the `src` folder into the `build` folder in the root of your project.

The "--skip" flag will skip the large logo and version/website logging. The "--log" flag will enable verbose logging - any file or folder that is copied, edited, or deleted will be logged.

Using a file watching script in conjunction with this project is highly recommended. The idea is that whenever a file is changed, added, or deleted within your main source folder, your project should rebuild automatically. Here's an example using [Chokidar-CLI](https://www.npmjs.com/package/chokidar-cli):

```json
{
  "scripts": {
    "build": "abjure build \"./src\" \"./build\"",
    "watch": "chokidar \"./src\" -c \"npm run build\""
  }
}
```

Running `npm run watch` with the above script configuration will start the watcher, and whenever a file is changed within the `src` folder, the build script will run.

## Usage

For inserting and removing multiple lines, start with `/*+++*/` for inserting code and `/*---*/` for removing code. End these sections with `/*...*/`. Everything between these lines will either be inserted (whereby double slash comments will be removed from the start of the line) or removed (whereby double slash comments will be added to the start of the line). Note that these lines only need to start with these patterns (ignoring any leading space or tab characters), which can be helpful for extra comments.

For inserting and removing single lines, start with `// /*++*/` for inserting code and `/*--*/` for removing code. These sections don't require any ending pattern.

For inserting and removing inline code, start with `/*+*/ /*` for inserting code and `/*-*/` for removing code. These patterns can be inserted anywhere in a line of code, and will change into `/*+*/` for insertion and `/*-*/ /*` for removal. Ending these parts of a line with `/*.*/` can help clean up lingering block quotes in both the source file or the transpiled result.

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

## Examples

Here are a couple of projects that I designed which make use of Abjure:

* [Morphmaster](https://github.com/Malcomian/Morphmaster): An electron boilerplate application with CLI build utilities based off AngularJS and Bootstrap.
* [Morphmaster-Web](https://github.com/Malcomian/Morphmaster-Web): A web application boilerplate with CLI build utilities based off AngularJS and Bootstrap.

## Known Issues

One issue with the single line and inline methods is that they will usually behave poorly with anything but the default vscode code formatter for javascript files, as most code formatters will push code that appears after a block quote, which is anything starting with `/*` and ending with `*/`, to a new line.
