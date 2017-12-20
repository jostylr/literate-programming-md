# Core

This is the core package that runs a minimal version of literate-programming.
It basically stitches the code together and returns out an object of the
compiled code blocks, with substitutions all done. It has the facilities of
directives and commands built into it, but it has only a minimal set of them. 

This is a good warmup module. The command client can be run with the flag
--core to only run this. It should also be completely safe to run as there
will be no ability to run arbitrary code. 

This should export a function that takes in an optional object to
generate a transpiling function that itself takes in text and an optional function for
requesting new text (give a name, comes back). This allows for the load
directive to be there. It outputs an object with the code blocks compiled, 
a file array created by the save command, and an options object that can be
imported back into the calling transpiler (import directive). 

This is an async function and so it initially returns a promise.



    /*jslint evil:true, node:true*/

    const commonmark = require('commonmark');
    const Promuse = require('bluebird');
   
    module.exports = function makeTranspiler (options = {} ) {
        _"merge options"
        
        async function parser (text, loader) {
            const fragments = new Map();
            _"parser"

            return fragments;
        }
        

        _"stitch"



        async function transpiler (text, loader) {
            
            const fragments = await parser(text);
            const {blocks, files} = await stitch(fragments, loader);
            
            return { blocks, files, options };
        }

        return transpiler; 
    };


* [index.js](# "save:") This is the main entry point of the module.
* [commands](commands.md "load:")
* [directives](directives.md "load:")
* [subcommands](subcommands.md "load:")



Decided to start using Map object. Probably not a great idea. To serialize,
etc., use https://github.com/JSON8/JSON8  

So why Maps? They strike me as appropriate for dynamically generated hashes
where data is the primary part. Prevents accidental name collisions which
could happen with block headers, such as `toString`

The options object can be modified in the parser or stitch function, but it
need not be. The transpiler returns it so that it is possible to modify the
options object after another transpiler called was called (an import function
calling another lit pro file, for example). 

## Stitch

At this point, we should have a number of fragments and are ready to transpile
them. Directives have been executed and we can start doing substitutions. We
iterate on the fragments and this is the place where each iteration is fed. 

Each block comes out as an array with the first entry being the value and the
second entry is a source map of sorts. It contains the starting index value of each
piece of inserted fragment, the source in the original text of the start of
the substitution request (or actual position of text if in fragment), and the
name of the block if applicable.

    
    async function stitch (fragments, loader) {
        const blocks = new Map();
        const files = new Map();

        async function fragpile (value, key) {
            _"fragpile"
        }

        fragments.forEach(function (frag, name) {
           blocks.set(name) = fragpile(frag, name);
        });

        await Promise.props(blocks);

        //do some file stuff
        await Promise.props(files);

        return {blocks, files}

    }


## Parser


## Options

This defines and extends (from the passed in options) the options available.
This includes the commands, directives, subcommands, and special symbols. 

Don't use destructuring because we want to augment commands, directives, and
subcommand. 

    _"merge | sub TYPE, commands, DEFAULT, _'Default Commands | objblocks commands, com | compile' "
    _"merge | sub TYPE, directives, DEFAULT, _'Default directives' "
    _"merge | sub TYPE, subcommands, DEFAULT, _'Default subcommands' "
    
    if (options.commands) {
        _"merge"
    }

    const directives = {
    }

    if (options.directives) {

    }

    const loader = options.loader || _"loader";

    const quotes = options.quotes || ['"', "'", "`"];
    const underscore = options.underscore || "
    const pipe = options.pipe || "|";
    const colon = options.colon || ":";



### Merge

This does the merging. It is generically written so we can apply to commands,
directives, and subcommands. 

    const TYPE = { DEFAULT };

    if (options.TYPE) {
        for (let name in options.TYPE) {
            TYPE[name] = options.TYPE[name];
        }
    }


### Default Commands

    sub, trim, split, join, wrap, log 




### Default Directives

    save, load, pipe


### Default Subcommands

    echo


### Loader

This is a default function for loading. Generally this really needs to be
replaced, but in the absence of it, it will log the request and return the
promise that evaluates in one second an empty text item. 

In general, the loader is responsible for loading text from
other resources, usually in an asynchronous fashion. This is why the timeout,
to simulate the async nature. 


    function logLoader (request) {
        console.log("Request for " + request + " will not be fulfilled.  Returning empty string. ");
        return Promise.delay(1000, "");
    }


## Object Blocks

This takes in a string of names and returns something that should have the
blocks replaced with blocks to evaluate. If a filename is provided then that
is used. 

    function (input, args) {
        let fname = args[0] || '';
        let bp = (_'function boilerplate')(input, fname);
        
        const ret = input.
            split(",").
            map(a => a.trim()).
            map(a => a + ':' + bp(a)).
            join(",");

        console.log(ret);

       return ret;

    }

[objblocks](# "define:")


### Function boilerplate

This creates a basic structure boilerplate function. The creation function
takes in the fname and produces a function that takes in the input and creates
the boilerplate plus a call to the block storage. Basically, it creates
something like 
`trim --->  trim : function commandsTrim (input) {_"commands:trim"}`

    function (file) {
        switch (file) {
        case "commands" : 
        return _"commands::boilerplate"
        case "directives" :
        return _"directives::boilerplate"
        case "subcommands" :
        return _"subcommands::boilerplate"
        
    }


