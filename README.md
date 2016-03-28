## bake - bash [mr]ake

Pure bash build framework.  No libraries, no dependencies (in this framework).  Designed in the spirit of rake and make.  I wished for a self-contained tool that did not require any more boostrapping than running curl or a single scp, so I made this.

The API follows in the spirit of Ruby's amazing and wonderful Rake utility.

## Installation

    test -d $HOME/bin || mkdir $HOME/bin
    curl https://raw.githubusercontent.com/kyleburton/bake/master/bake > $HOME/bin/bake
    chmod 755 $HOME/bin/bake

NOTE: OS X (the Apple Mac) has been regressing it's version of bash.  Newer (as of 2016) versions of OS X have a bash that bake is incompatible with.  You can easily install a modern supported version of bash via [homebrew](http://brew.sh/) and then either [set it as your shell](http://johndjameson.com/blog/updating-your-shell-with-homebrew/) or change the first line of the bake script to point to the bash you installed via homebrew.

## Example Bakefile

    $ mkdir test/lib
    $ cat >> test/lib/mylib.sh
    #!/usr/bin/env bash
    set -eu
    bake_task foo "The foo command just echos it's arguments"
    function foo () {
      echo "foo: args='$@'"
    }
    ^D

    $ cat >> Bakefile
    #!/usr/bin/env bash
    bake_push_libdir $(bake_bakefile_dir)/test/lib
    bake_require mylib
    ^D

Then run bake:

    $ bake
    
    bake task [arg ...]
    
      foo                            The foo command just echos it's arguments
    
    
    $ bake foo this that
    foo: args='this that'



## `Bakefile`

`bake` is controlled by a Bakefile (similarly to make and rake).  This file is just a bash script.  You define functions for your tasks and register them with `bake`.

### `bake_task task-name "task-description"`

This registers a task and it's description so it can be executed and help can be displayed.

### `bake_default_task task-name`

This registers the default task to be run if no task is passed on the command line.

### `bake_push_libdir path`

Pushes a file system path onto the front of `BAKEPATH`.

### `bake_add_libdir path`

Pushes a file system path onto the end of `BAKEPATH`.

### `bake_require libname`

Searches `BAKEPATH` for the library and sources it, loading the file (executing its contents).  Libraries should (generally) only contain declarations, as any imperative code will be executed when the library is loaded.  Libraries may load other libraries.

## Environment

### BAKEPATH

This is a colon separated list of paths that `bake_require` uses to locate libraries.

## Best Practices for writing Bakefiles

#### Configuration at the top

Factor out configuration and put it at the top of your Bakefile. 

```sh
#!/usr/bin/env bake

CFGFNAME="${CFGFNAME:-config.json}"

bake_task configure_system "Configure the system with $CFGFNAME"
function configure_system () {
  bake_echo_green "Pushing configuration: $CFGFNAME"
  # implement the task here
}
```

#### Use defaults, allow overrides

For configuration and function arguments, give users the ability to supply values and override them.

```sh
#!/usr/bin/env bash

# do this:
CFGFNAME="${CFGFNAME:-config.json}"
# instaed of this:
CFGFNAME="config.json"
```

The former allows users to override the environment variable by either exporting it in their shell, or for a single invocation:

```sh
$ CFGFNAME=my-config.json bake configure_system
```

#### Externalize Complex Configuration

If there is either a lot of configuraiton, or it's complex and can't really be defaulted (thus forcing users to create configuraiton before running your tool).

#### Be Helpful

Use exit codes to indicate success or failure.  If you write a function that detects an error and exits, use 'return 1' so the calling shell knows that your task did not succeed.  This is important for your CI server and any other automation around your Bakefile.

Write task descriptions.  

Use your configuration variables in your task descriptions.  

Default your required task arguments to the empty string, then test if they are zero length and provide a sensible error message (and return an error code).


## License

Copyright (C) 2014-2016 Kyle Burton &lt;kyle.burton@gmail.com&gt;

Distributed under the Eclipse Public License, the same as Clojure.

