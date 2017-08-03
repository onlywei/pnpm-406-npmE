# Reproducible pnpm issue with NPM Enterprise

When tying to install `pnpm` with an `.npmrc` file pointing to an NPM Enterprise instance, we're
running an HTTP 406 error. Below are steps to reproduce this issue after checking out this repo
while pointing to our internal NPM Enterprise instance (set in the `.npmrc` file):

1. If you have a `node_modules` directory, delete it
  ```
  rm -rf node_modules/
  ```
2. If you have a `~/.pnpm-store` directory, delete it
  ```
  rm -rf ~/.pnpm-store
  ```
3. Install `pnpm` using npm *version 3.10.10* or lower
  ```
  npm install --save-dev pnpm
  ```
4. Run the local `pnpm install` so that `pnpm` can install itself
  ```
  ./node_modules/.bin/pnpm install
  ```

Expected result: pnpm installs successfully

Actual result:
```
└── pnpm@1.9.0

WARN Moving pnpm that was installed by a different package manager to "node_modules/.ignored
ERROR fetch failed with status code 406
```

After lots of investigating, we've determined that the following details matter a lot to get this
bug to be reproducible:
1. There must be an `.npmrc` file that points to an NPM Enterprise instance.  
   Removing it so that we point to the public repo seems to not surface the issue

2. You must be using a 3.x.x version of `npm`  
   Installing `pnpm` using `npm` version 5.x.x does not surface the issue.

3. There has to already be a `shrinkwrap.yaml` file when you run `pnpm install`.  
   If there isn't a shrinkwrap file to begin with, this issue does not surface. Confusingly, when
   the issue does surface, deleting the `shrinkwrap.yaml` file does not surface the issue, and when
   you compare the _new_ `shrinkwrap.yaml` to the one that was deleted, there are *no changes*.

We did some debugging on the URL that `pnpm` is trying to install from.

When it installs successfully, it's pulling from:
```
https://npm-registry.compass.com/p/pnpm/_attachments/pnpm-1.9.0.tgz
```

When it fails, it's pulling from:
```
https://npm-registry.compass.com/pnpm/-/pnpm-1.9.0.tgz
```

Let us know if you need any further help reproducing this issue.

Thanks!  
The Compass Frontend Foundations Team
