# Welcome to animate

> This project was created using [`npx create-roku-app`](https://github.com/haystacknews/create-roku-app)

## Before you start

1. Install the project dependencies if you haven't done that yet.

1. Open the `bsconfig.json` file and enter the password for your Roku device.

1. Optionally you can hardcode your Roku device's IP in the `host` field. If you do so make sure to remove the `host` entry from the `.vscode/launch.json` settings.

## Launching your app

> This project assumes that you will be using VSCode as your code editor.

1. Go to the `Run and Debug` panel.

1. Select the option `Launch animate (dev)`

## NPM Commands available

- `build`: Builds your project with [`brighterscript`](https://github.com/rokucommunity/brighterscript). Includes source maps.

- `build:prod`: Builds your project without source maps.

- `lint`: Lints your source files with [`@rokucommunity/bslint`](https://github.com/rokucommunity/bslint)

- `lint:fix`: Lints your source files and applies automatic fixes.

- `format`: Formats your source files with [`brighterscript-formatter`](https://github.com/rokucommunity/brighterscript-formatter)

- `format:fix`: Formats your source files and applies automatic fixes.
