![SeAT](https://i.imgur.com/aPPOxSK.png)

# Developer Installation

This document describes how to set up a development environment for both core package and third party package development. 
The easiest way to set up a development environment is to use docker. You can use the same image as in a production environment.

For development, plugins and the seat core are treated the same. When working on the core, you install a core package like
any other plugin. This is also how core packages are implemented: Internally they are also just a plugin, only that they 
are considered the core and that other plugins build on top of them.

The SeAT docker image is built in a way to prefer development installations over others. While the docker image always downloads and installs
the core packages, when it finds a local development install of a core package, it uses that over the latest version from packagist.

## Setup Process

First, start with [standard installation](../installation/docker_installation.md) to get a working environment.

The official docker-compose wrapper is shipped with a `packages` directory.
It is mounted readonly, and you can store your development sources in it.

To make things easier, we recommend you keep vendor path convention to split your sources across every single package you want to play with.

### Overrider
In the next step, we need to let laravel, the php framework used by seat, know that our plugin is there.

The image has been designed to look for a file called `override.json` inside `packages` directory.
When it is found, it will be merged together with standard `composer.json` file from `eveseat/seat` package.

It's designed to override both `autoload` and `providers`.
Here is a complete `override.json` structure:

```json linenums="1"
{
  "autoload": {
    "namespace_to_load\\": "packages/sources_path"
  },
  "providers": [
    "FQCN\\Provider"
  ]
}
```

An override can have either autoload, providers or even both property.
Do not forget to escape `\` in order to get a valid json file.

When your container will start, mapping from `autoload` property in your `override.json` file will be merged with `autoload-dev` property from official `composer.json`.

!!! tips "SeAT 4.x"

    1. If you need access the console of any container, access it via `docker exec seat-web sh` where `seat-web` is the name of the target container.
    2. You can execute `artisan` commands from outside of docker with `docker exec seat-web php artisan <command>`

!!! tips "SeAT 5.x"

    1. If you need access the console of any container, access it via `docker exec front sh` where `front` is the name of the target container.
    2. You can execute `artisan` commands from outside of docker with `docker exec front php artisan <command>`

Please note that there is currently no way to install dependencies with the package override. 

When installing already existing packages (e.g. a core package), you can find all required autoloads and service providers in it's `composer.json` file.

## Teach things by example

As an example, let's say I want to make a new feature in web core package, I'll spawn an `eveseat` directory at root `packages` directory, followed by a clone from `eveseat/web` git repository.
Last but not least, I'll create an `override.json` file to inform SeAT there are developer things to load.

1. Create vendor directory into `packages` directory `mkdir packages/eveseat`
2. Cloning core web package into `packages/eveseat/web` directory `git clone https://github.com/eveseat/web.git packages/eveseat/web`
3. Create an `override.json` to use custom web sources

```shell linenums="1"
cat > packages/override.json << EOL
{
  "autoload": {
    "Seat\\Web\\": "packages/eveseat/web/src/"
  }
}
EOL
```

!!! tips

    If you're working with Windows, prefer to store your files in wsl layer rather than Windows directory.
    Both work, however, you'll get better performances!
