Ruboto Core
=============

Ruby on Android.

Installation
-------

    $ gem install ruboto-core

Getting Started
---------------

Before you use Ruboto, you should do the following things:

* Install the JDK if it's not on your system already
* Install [jruby](http://jruby.org/) if you don't already have it. JRuby has a [very easy install process](http://jruby.org/#2), or you can use [rvm](http://rvm.beginrescueend.com/)
* Install [the Android SDK](http://developer.android.com/sdk/index.html)
* Add the sdk's `tools/` directory to your `$PATH`
* Generate an [Emulator](http://developer.android.com/guide/developing/tools/emulator.html) image unless you want to develop using your phone.

General Information
------------------

The Rakefile assumes that you are in the root directory of your app, as do all commands of the `ruboto` command line utility, other than `ruboto gen app`.

The Rakefile requires you to run it through JRuby's rake. 

Command-line Tools
-------

* [Application generator](#application_generator) (like the rails application generator)
* [Class generator](#class_generator) to generate additional Activities, BroadcastReceivers, Services, etc.
* [Packaging task](#packaging_task) to generate an apk file
* [Deployment task](#deployment_task) to deploy a generated package to an emulator or connected device
* [Develop without having to compile to try every change](#update_scripts)


<a name="application_generator"></a>
### Application generator

    $ ruboto gen app --package com.yourdomain.whatever --path path/to/where/you/want/the/app --name NameOfApp --target android-version --activity MainActivityName
Currently any value but `android-8` will not work.

<a name="class_generator"></a>
### Class generator

    $ ruboto gen class ClassName --name YourObjectName
Ex:
    $ ruboto gen class BroadcastReceiver --name AwesomenessReceiver

<a name="packaging_task"></a>
### Packaging task

This will generate an apk file.

    $ rake

To generate an apk and install it to a connected device (or emulator) all in one go, run

    $ rake install

<a name="deployment_task"></a>
### Deployment task

When you're ready to post your app to the Market, you need to do a few things.

First, you'll need to generate a key to sign the app with using `keytool` if you do not already have one. If you're ok with accepting some sane defaults, you can use
    $ ruboto gen key --alias alias_for_your_key
with an optional flag `--keystore /path/to/keystore.keystore`, which defaults to `~/.android/production.keystore`. It will ask for a password for the keystore and one for the key itself. Make sure that you remember those two passwords, as well as the alias for the key. 

Also make sure to keep your key backed up (if you lose it, you won't be able to release updates to your app that can install right over the old versions), but secure.

Once you have your key, use the `rake publish` task to generate a market-ready `.apk` file. You will need the `RUBOTO_KEYSTORE` and `RUBOTO_KEY_ALIAS` environment variables set to the path to the keystore and the alias for the key, respectively. So either run
    $ RUBOTO_KEYSTORE=~/.android/production.keystore RUBOTO_KEY_ALIAS=foo rake publish
or set those environment variables in your `~/.bashrc` or similar file and just run
    $ rake publish
Now get that `.apk` to the market!

<a name="update_scripts"></a>
### Updating Your Scripts on a Device

With traditional Android development, you have to recompile your app and reinstall it on your test device/emulator every time you make a change. That's slow and annoying.

Luckily, with Ruboto, most of your changes are in the scripts, not in the compiles Java files. So if your changes are Ruby-only, you can just run

    $ rake update_scripts

to have it copy the current version of your scripts to your device.

Sorry if this takes away your excuse to have sword fights:

![XKCD Code's Compiling](http://imgs.xkcd.com/comics/compiling.png)

Caveats:

This only works if your changes are all Ruby. If you have Java changes (which would generally just mean generating new classes) or changes to the xml, you will need to recompile your script.

Also, you need root access to your device for this to work, as it needs to write to directories that are read-only otherwise. The easiest solution is to test on an emulator, but you can also root your phone.

### Updating Ruboto's Files

Not implemented, yet.


Scripts
-------

The main thing Ruboto offers you is the ability to write Ruby scripts to define the behavior of Activites, BroadcastReceievers, and Services. (Eventually it'll be every class. It's setup such that adding in more classes should be trivial.)

Here's how it works:

First of all, your scripts are found in `assets/scripts/` and the script name is the same as the name of your class, only under_scored instead of CamelCased. Android classes have all of these methods that get called in certain situations. `Activity.onDestroy()` gets called when the activity gets killed, for example. Save weird cases (like the "launching" methods that need to setup JRuby), to script the method onFooBar, you call the Ruby method handle_foo_bar on the Android object. In your scripts, they are defined as `$class_name`. That was really abstract, so here's an example. 

You generate an app with the option `--activity FooActivity`, which means that ruboto will generate a FooActivity for you. So you open `assets/scripts/foo_activity.rb` in your favorite text editor. If you want an activity that does nothing but Log when it gets launched and when it gets destroyed (in the onCreate and onPause methods). You want your script to look like this:

    require 'ruboto.rb' #scripts will not work without doing this
    $activity.handle_create do |bundle|
      Log.v 'MYAPPNAME', 'onCreate got called!'
      handle_pause do
        Log.v 'MYAPPNAME', 'onPause got called!'
      end
    end

If you prefer, you can also do this. It's equivalent:

    require 'ruboto.rb' #scripts will not work without doing this
    $activity.handle_create do |bundle|
      Log.v 'MYAPPNAME', 'onCreate got called!'
    end
    $activity.handle_pause do
      Log.v 'MYAPPNAME', 'onPause got called!'
    end

Each class has only one method that you can nest other calls inside of (ie. what is happening in that first example that removes the need for the second `$activity.`). For Activities and Services, it is `handle_create`, and for BroadcastReceivers, it is `handle_receive`. The general rule is that it corresponds to the first method in the class's lifecycle. But you should never really have to think about it because generating a class generates a sample script that calls that method.

The arguments passed to the block you give `handle_` methods are the same as the arguments that the java methods take. Consult the Android documentation.

Activities also have some special methods defined to make things easier. The easiest way to get an idea of what they are is looking over the [demo scripts](http://github.com/ruboto/ruboto-irb/tree/master/assets/demo-scripts/). You can also read the [ruboto.rb file](http://github.com/ruboto/ruboto-core/blob/master/assets/assets/scripts/ruboto.rb) where everything is defined.

Contributing
------------

Want to contribute? Great! Meet us in #ruboto on irc.freenode.net, fork the project and start coding!

"But I don't understand it well enough to contribute by forking the project!" That's fine. Equally helpful:

* Use Ruboto and tell us how it could be better.
* As you gain wisdom, contribute it to [the wiki](http://github.com/ruboto/ruboto-core/wiki/)
* When you gain enough wisdom, reconsider whether you could fork the project.

Getting Help
------------

* You'll need to be pretty familiar with the Android API. The [Developer Guide](http://developer.android.com/guide/index.html) and [Reference](http://developer.android.com/reference/packages.html) are very useful. 
* There is further documentation at the [wiki](http://github.com/ruboto/ruboto-core/wiki)
* If you have bugs or feature requests, [open an issue on GitHub](http://github.com/ruboto/ruboto-core/issues)
* You can ask questions in #ruboto on irc.freenode.net and on the [mailing list](http://groups.google.com/groups/ruboto)
* There are some sample scripts (just Activities) [here](http://github.com/ruboto/ruboto-irb/tree/master/assets/demo-scripts/)

Tips & Tricks
-------------

### Emulators

If you're doing a lot of Android development, you'll probably find yourself typing `emulator -avd name_of_emulator` a lot to open emulators. It can be convenient to alias these to shorter commands.

For example, in your `~/.bashrc`, `~/.zshrc`, or similar file, you might put
    alias eclair="emulator -avd eclair"
    alias froyo="emulator -avd froyo"
If you have an "eclair" emulator that runs Android 2.1 and a "froyo" one that runs Android 2.2.


Alternatives
------------

If Ruboto's performance is a problem for you, or you want something that gives you total access to the android API (as Ruboto does not yet do), check out [Mirah](http://mirah.org/) and [Garrett](http://github.com/technomancy/Garrett). 

Mirah, formerly known as Duby, is a language with Ruby-like syntax that compiles to java files. This means that it adds no big runtime dependencies and has essentially the same performance as writing Java code because it essentially generates the same Java code that you would write. This makes it extremely well-suited for mobile devices where performance is a much bigger consideration. 

Garrett is a "playground for Mirah exploration on Android."


Domo Arigato
-----

Thanks go to:

* Charles Nutter, a member of the JRuby core team, for mentoring this RSoC project and starting the Ruboto project in the first place with an [irb](http://github.com/ruboto/ruboto-irb)
* All of Ruby Summer of Code's [sponsors](http://rubysoc.org/sponsors)
* [Engine Yard](http://engineyard.com/) in particular for sponsoring RSoC and heavily sponsoring JRuby, which is obviously critical to the project.
* All [contributors](http://github.com/ruboto/ruboto-core/contributors) and [contributors to the ruboto-irb project](http://github.com/ruboto/ruboto-irb/contributors), as much of this code was taken from ruboto-irb.
