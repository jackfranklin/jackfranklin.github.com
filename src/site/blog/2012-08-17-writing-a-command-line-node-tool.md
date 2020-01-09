---
permalink: /blog/writing-a-command-line-node-tool

title: "Writing a Command Line Node Tool"
deprecated_by: "/blog/2015/03/node-command-line-tool/"
date: 2012-08-17
---

Today we are going to combine a few different tools and create a simple Node package that will allow a user to search a directory for files. In this tutorial we will use Grunt to do a lot of the work for us, see how to to make a Node script executable on the command line, and finally see how we publish it to the Node Package Manager (npm) so anyone can install it.

The pre-requisites to this are:

* You have NodeJS installed (and preferably 0.10.32 up, this is **not** tested on Node < 0.10.32)
* Have the Node Package Manager (npm) installed.
* Have Grunt-init and and Grunt-cli installed, or if not, run `npm install -g grunt-init` and `npm install -g grunt-cli` (or sudo npm install -g grunt-cli ). Some basic familiarity is good too, [I've written an introduction to it](http://javascriptplayground.com/blog/2012/04/grunt-js-command-line-tutorial) previously. If you've never used it, go read that and then return.

So the first thing to do is create a new project. Create a directory for it and change to the directory you created.

* Install the current version of Grunt local to your project

  npm install grunt --save

This will mark grunt your project's package.json devDependencies section.

* Add the node grunt-init template

  git clone https://github.com/gruntjs/grunt-init-node.git ~/.grunt-init/node

(The current version on grunt-init doesn't come with any base templates. Additional information is avaliable at [Project Scaffolding](http://gruntjs.com/project-scaffolding)

* Use grunt-init to create a new node project

  grunt-init node

This will take us through set up to set up our new project. It will ask you some questions. Feel free to deviate, but here's how I answered them:

    [?] Project name (playground-nodecmd) filesearch
    [?] Description (The best project ever.) Awesome file search.
    [?] Version (0.1.0)
    [?] Project git repository (git://github.com/JackFranklin/filesearch.git)
    [?] Project homepage (https://github.com/JackFranklin/filesearch)
    [?] Project issues tracker (https://github.com/JackFranklin/filesearch/issues)
    [?] Licenses (MIT)
    [?] Author name (Jack Franklin)
    [?] Author email (jack@jackfranklin.net)
    [?] Author url (none)
    [?] What versions of node does it run on? (>= 0.8.0) 0.10.32
    [?] Main module/entry point (lib/filesearch)
    [?] Npm test command (grunt nodeunit)
    [?] Will this project be tested with Travis CI? (Y/n) n
    [?] Do you need to make any changes to the above before continuing? (y/N) n

You will see Grunt has got us started:

```
Writing .gitignore...OK
Writing .jshintrc...OK
Writing Gruntfile.js...OK
Writing README.md...OK
Writing lib/filesearch.js...OK
Writing test/filesearch_test.js...OK
Writing LICENSE-MIT...OK
Writing package.json...OK

Initialized from template "node".
You should now install project dependencies with npm install. After that, you
may execute project tasks with grunt. For more information about installing
and configuring Grunt, please see the Getting Started guide:

http://gruntjs.com/getting-started

Done, without errors.
```

We wont actually be writing tests for this package as it's very simple. To search for files in a directory, we're just going to execute the shell command:

    ls -a | grep somefile

In the future I will write on creating more complex modules and testing them, but for this we'll focus on implementation.

Load up `package.json` in your editor. It should look like this:

    {
      "name": "filesearch",
      "description": "Awesome file search.",
      "version": "0.1.0",
      "homepage": "https://github.com/JackFranklin/filesearch",
      "author": {
        "name": "Jack Franklin",
        "email": "jack@jackfranklin.net"
      },
      "repository": {
        "type": "git",
        "url": "git://github.com/JackFranklin/filesearch.git"
      },
      "bugs": {
        "url": "https://github.com/JackFranklin/filesearch/issues"
      },
      "licenses": [
        {
          "type": "MIT",
          "url": "https://github.com/JackFranklin/filesearch/blob/master/LICENSE-MIT"
        }
      ],
      "main": "lib/filesearch",
      "engines": {
        "node": "0.10.32"
      },
      "scripts": {
        "test": "grunt nodeunit"
      },
      "devDependencies": {
        "grunt-contrib-jshint": "~0.6.4",
        "grunt-contrib-nodeunit": "~0.2.0",
        "grunt-contrib-watch": "~0.5.3",
        "grunt": "~0.4.5"
      },
      "keywords": []
    }

We need to add some properties to that. After the last property, as shown below:

    "Keywords": []
    ```
     //Add here this here
     ,"preferGlobal": "true",
      "bin": {
        "filesearch" : "lib/filesearch.js"
      }
     ```
    }

The first line denotes that our package should be installed globally if possible. If the user installs it locally, they will see a message about how it should be done globally. The second object, `bin`, denotes files that should be executable on the command line, and how we should reference them. Here we are saying that when we hit `filesearch` in the command line, it should run `lib/filesearch.js`.

To make this happen, load up `lib/filesearch.js` in your editor, and add this line at the very top:

    #! /usr/bin/env node

This says how the script should be executed, in this case through Node.

Add an additional line to the end of `lib/filesearch.js`:

    console.log("Success");

Once that is done, we can run `npm link` to install our package locally so we can test it. Run `npm link` and then you should have access to the `filesearch` command. Of course, right now it only logs success to the console. To confirm it is working run `filesearch Grunt` and look for the output `success`.

Now, delete the rest of the code from `lib/filesearch`, which is:

    'use strict';

    exports.awesome = function() {
      return 'awesome';
    };

    console.log("Success");

`exports` is a way of exporting methods and variables from your script, that can be used in others. Say if this script was one other developers could use, `exports` is the object that will be returned when a developer includes our module through `var x = require("ourpackage");`. Because ours is a command line tool that's little use, so there's no need to include it. Now, lets implement this. I am envisaging that the use of this module is like so:

    filesearch filename

So the parameter passed in is what we need to search for. All the arguments are stored in the array `process.argv`. To inspect them, add this line:

    console.log(process.argv);

And then run `filesearch grunt` and check the result:
filesearch grunt
[ 'node', '/usr/local/bin/filesearch', 'grunt' ]
You can see that the first two arguments refer to how the script is executed and where the executable is. Hence, the actual arguments passed in start at the second index. Therefore we can get at the user supplied arguments by slicing the array at index 2:

    var userArgs = process.argv.slice(2);

And then get our argument as the first argument of `userArgs`:

    var searchParam = userArgs[0];

Rather than do the implementation step by step, as it's only six lines, I'll show you and then explain:

    var userArgs = process.argv.slice(2);
    var searchParam = userArgs[0];

    var exec = require('child_process').exec;
    var child = exec('ls -a | grep ' + searchParam, function(err, stdout, stderr) {
        if (err) throw err;
        console.log(stdout);
    });

The first two lines get the search parameter, as I explained above.

Next up we use Node's [Child Process](http://nodejs.org/api/child_process.html) library, more specifically the [exec module](http://nodejs.org/api/child_process.html#child_process_child_process_exec_command_options_callback), which runs a shell command and buffers the output. The command we need to run is:

    ls -a | grep {searchParam}

For those unfamiliar with the shell, `ls -a` means list all files, and `grep something` searches for the term "something". By piping the result of `ls -a` through to `grep something`, it searches everything `ls -a` returned for `something`.

So once we have the `exec` variable, we can execute it. It takes two parameters, the string to execute and a callback. Exec is asynchronous, like most of Node in general, so any code to run after we have the result must go in the callback. Within the callback, all we do is throw an error if it exists, and if it doesn't just log the output.

The pattern of callback functions taking the error as the first parameter is very common within Node. You will often see:

    function(err, stdout, stderr) {
    	if(err) throw err;
    	//rest of code
    }

As a pattern. Now we've done that, running `filesearch Grunt` within our project directory should get you what we want:

    filesearch Grunt
    Gruntfile.js

Of course, this module in practice is useless, but hopefully it has demonstrated how to go about making simple command line tools in Node. If you'd like a more complex example, my [Nodefetch](https://github.com/jackfranklin/nodefetch) tool might make interesting reading.

To publish this as an npm module, you need to do three things. Firstly, authenticate yourself with npm, or signup with npm. To do this, run `npm adduser`.

Secondly, you should make sure your project is a Git repository, and:

* Add `node_modules/` to your `.gitignore` file, to make sure only your module code is pushed, and not the modules you use. These are dealt with when the user installs your module.
* Make sure your repository has a valid `package.json` (running `npm link` will verify this, if it works without error, you're fine).
* Push your repository to Github (or elsewhere) and make sure in your `package.json`, the `repository` object looks like so:

      		"repository": {
      		   "type": "git",
      		   "url": "git://github.com/JackFranklin/filesearch.git"
      		 }

Then it's easy, just run `npm publish`, and you're done. It really is as easy as that. Users can then install your module through `npm install modulename`.

I hope this tutorial has been useful, and if you have any questions please leave a comment, or feel free to tweet or email me.
