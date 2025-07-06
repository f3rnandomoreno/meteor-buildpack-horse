# Meteor Buildpack Horse - SWC Edition

[![Horse](https://i.imgur.com/YhIL9zM.jpg)](https://commons.wikimedia.org/wiki/File:Draw-Costa_Rican-2smallest.jpg)

A heroku buildpack for Meteor v1+, designed to be as simple and readable as possible, **now with Meteor 3.3+ SWC support**.

## 🚀 SWC Support (NEW!)

This buildpack has been enhanced to support **Meteor 3.3+** which uses **SWC (Speedy Web Compiler)** instead of Babel for faster builds. The buildpack automatically detects Meteor 3.3+ and handles SWC native bindings correctly during deployment.

### SWC Features:
- ✅ Automatic detection of Meteor 3.3+ with SWC
- ✅ Proper handling of SWC native bindings for Linux x64 deployment
- ✅ Clean dependency installation with optional packages
- ✅ Enhanced error handling for SWC compilation issues
- ✅ Backward compatibility with pre-SWC Meteor versions

## Usage

To use this with your meteor app and heroku:

1. Set up your app to [deploy to heroku with git](https://devcenter.heroku.com/articles/git).
2. Set your buildpack to this SWC-compatible version:

        heroku buildpacks:set https://github.com/f3rnandomoreno/meteor-buildpack-horse

3. Add the MongoLab addon:

        heroku addons:create mongolab

4. Set the `ROOT_URL` environment variable. This is required for bundling and running the app.  Either define it explicitly, or enable the [Dyno Metadata](https://devcenter.heroku.com/articles/dyno-metadata) labs addon to default to `https://<appname>.herokuapp.com`.

        heroku config:set ROOT_URL="https://<appname>.herokuapp.com" # or other URL

Once that's done, you can deploy your app using this build pack any time by pushing to heroku:

    git push heroku master

## Environment

The following are some important environment variables for bundling and running your meteor app on heroku.  Depending on your settings, you may need to override these on heroku.  See [heroku's documentation](https://devcenter.heroku.com/articles/config-vars) for how to set these.

 - `ROOT_URL`: The root URL for your app, needed for bundling as well as running. If you enable the [Dyno Metadata](https://devcenter.heroku.com/articles/dyno-metadata) labs addon and `ROOT_URL` is undefined, it will default to `https://<appname>.herokuapp.com`).
 - `MONGO_URL`: The URL to mongodb. If not defined, it will default the value of `MONGODB_URI`, `MONGOLAB_URI`, or `MONGOHQ_URL` (in order).  If you don't use mongolab as a regular addon (and none of the fallbacks are defined), you'll need to set this.
 - `METEOR_APP_DIR`: The relative path to the root of your meteor app within your git repository (i.e. the path to the directory that contains `.meteor/`). The buildpack will look in the root of your repository and `app/` subdirectory; if you put your app anywhere else (like `src/`), define this variable to tell the buildpack where to look.
 - `BUILDPACK_PRELAUNCH_METEOR`: If your app uses packages that need to compile their assets on first run, you may need meteor to launch prior to bundling.  If this applies for you, define `BUILDPACK_PRELAUNCH_METEOR=1`. [Reference issue](https://github.com/meteor/meteor/issues/2606).
 - `BUILDPACK_VERBOSE`: Set `BUILDPACK_VERBOSE=1` to enable verbose bash debugging during slug compilation. Only takes effect after the environment variables have been loaded.
 - `BUILDPACK_CLEAR_CACHE`: This buildpack stores the meteor installation in the [CACHE_DIR](https://devcenter.heroku.com/articles/buildpack-api#caching) to speed up subsequent builds. Set `BUILDPACK_CLEAR_CACHE=1` to clear this cache on startup and after build is done.
 - `BUILD_OPTIONS`: Set to any additional options you'd like to add to the invocation of `meteor build`, for example `--debug` or `--allow-incompatible-update`.
 - `NODEJS_PARAMS`: additional parameters for running `node` binary. This can be used eg. for [adjusting garbage collector settings](https://devcenter.heroku.com/articles/node-best-practices#avoid-garbage) by putting `--optimize_for_size --max_old_space_size=460 --gc_interval=100` here

## 🛠️ SWC-Specific Configuration

For Meteor 3.3+ apps using SWC, the buildpack automatically handles:

- **Automatic SWC Detection**: Detects Meteor 3.3+ and enables SWC-specific handling
- **Direct Binary Download**: Downloads SWC binaries directly from npm registry to avoid npm installation issues
- **Smart Fallback**: Falls back to traditional npm install if direct download fails
- **Enhanced Memory**: Sets `NODE_OPTIONS="--max-old-space-size=4096"` for SWC compilation
- **Cache Optimization**: Disables Meteor's optimistic caching that can interfere with SWC
- **Production Bundle**: Ensures SWC binaries are available in the final production bundle

### SWC Troubleshooting

If you encounter SWC native binding errors:

1. **Clear buildpack cache**:
   ```bash
   heroku config:set BUILDPACK_CLEAR_CACHE=1
   git commit --allow-empty -m "Clear buildpack cache for SWC"
   git push heroku master
   heroku config:unset BUILDPACK_CLEAR_CACHE
   ```

2. **Enable verbose logging**:
   ```bash
   heroku config:set BUILDPACK_VERBOSE=1
   ```

3. **Check your package.json** includes `@swc/core` in devDependencies:
   ```json
   {
     "devDependencies": {
       "@swc/core": "^1.12.9"
     }
   }
   ```

## Extras

The basic buildpack should function correctly for any normal-ish meteor app,
with or without npm-container.  For extra steps needed for your particular build,
just add shell scripts to the `extra` folder and they will get sourced into the
build.

Extras included in this branch:
 - `mongo_url.sh`: If `MONGO_URL` is empty, set it to the value of `MONGODB_URI`, `MONGOLAB_URI`, or `MONGOHQ_URL` (in order).
 - `root_url.sh`: If `ROOT_URL` is empty and `HEROKU_APP_NAME` is available, set `ROOT_URL` to `https://$HEROKU_APP_NAME.herokuapp.com`

## Where things go

This buildpack creates a directory `.meteor/heroku_build` (`$COMPILE_DIR`)
inside the app checkout, and puts all the binaries and the built app in there.
So it ends up having the usual unixy `bin/`, `lib/`, `share` etc
subdirectories.  Those directories are added to `$PATH` and
`$LD_LIBRARY_PATH` appropriately.

So `$COMPILE_DIR/bin` etc are great places to put any extra binaries or stuff
if you need to in custom extras.

## Version Compatibility

- ✅ **Meteor 3.3+**: Full SWC support with automatic detection
- ✅ **Meteor 1.3 - 3.2**: Standard Babel-based compilation  
- ✅ **Meteor 1.0 - 1.2**: Legacy support

## Using the latest buildpack code

To use this SWC-compatible buildpack, set your buildpack to this repository:

        heroku buildpacks:set https://github.com/f3rnandomoreno/meteor-buildpack-horse

## Tips & Tricks

Please help us add tips and tricks to the [wiki](https://github.com/AdmitHub/meteor-buildpack-horse/wiki) for further help, like usage with Dokku or other environments.

## Changelog

### SWC Edition Changes:
- 🚀 Added automatic Meteor 3.3+ SWC detection
- 🛠️ Enhanced npm install process for SWC native bindings
- 🔧 Added SWC-specific environment variables
- 📋 Improved error logging for SWC compilation issues
- 🧹 Added automatic cache cleaning for SWC compatibility
- ✅ Maintained backward compatibility with pre-SWC versions
- 🎯 **NEW**: Direct binary download from npm registry for reliable SWC installation
- 🔄 **NEW**: Smart fallback mechanism when direct download fails
- 🏭 **NEW**: Production bundle SWC binary verification and installation

## Why horse?

There are a gazillian forks and branches of various buildpacks remixing the
words "heroku", "buildpack", and "meteor", many of which are abandoned or
outdated or broken, and it's really hard to keep them straight.

So this one is the horse one (now with SWC support! 🐎⚡).

README image credit: wikicommons contributor [Arsdelicata](https://commons.wikimedia.org/wiki/User:Arsdelicata)

<a href="https://zenhub.com"><img src="https://raw.githubusercontent.com/ZenHubIO/support/master/zenhub-badge.png"></a>