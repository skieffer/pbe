# How to make a release


To begin with,

* You should be on the `main` branch.
* All the changes you want to make it into the release should be committed.
* You should update the `CHANGELOG.md` with an entry for the release you're about to
  make, and commit it.

Now make a release branch, of the form `release/VERSION`. For example,
if releasing version 3.1,

    $ git checkout -b release/3.1

Edit `package.json`, and remove the final `.1111` component of the version number.
Note: Since the `pbe` project represents an application (not a library),
we're using a simple two-part version number, not a strict three-part semver.
We use a final `1111` to signal a development version. We do this instead of
a `-dev` suffix, because Chromium won't let you install an extension for testing
unless its version number consists entirely of integers, separated by dots.

Do an `npm install` so the `package-lock.json` updates accordingly:

    $ npm install

Build the two releases, one for Mozilla, and one for Chromium-based browsers:

    $ npm run build:moz
    $ npm run build:chr

Note: We do not release _minified_ code. (It is however still _packed_ by webpack.)
As explained [here](https://extensionworkshop.com/documentation/publish/source-code-submission/),
there is little reason to minify a browser extension.

Produce the zip files using
[web-ext](https://extensionworkshop.com/documentation/develop/getting-started-with-web-ext/):

    $ cd dist-mozilla
    $ web-ext build
    $ cd ../dist-chrome
    $ web-ext build
    $ cd ..

Now you can stage everything. Note that you have to force-add the dist directories,
since they are gitignored.

    $ git add .
    $ git add -f dist-mozilla
    $ git add -f dist-chrome

Commit, with a simple message stating the version number. Then add a tag, and push both
the branch and the tag to GitHub. For example,

    $ git commit -m "Version 3.1"
    $ git tag v3.1
    $ git push origin release/3.1
    $ git push origin v3.1

Go back to the `main` branch. Do not delete the `release` branch; it is useful when
submitting the release to the web stores, as well as for testing old versions at a later
time.

    $ git checkout main

Bump the dev version number. For example, if the release was `v3.1`, then go
into `package.json` and change the version to `3.2.1111`. Finally, do a commit:

    $ git add package.json
    $ git commit -m "Bump dev version"

Finished! Now you can proceed to
<https://addons.mozilla.org/> and <https://chrome.google.com/webstore/devconsole>
to submit the releases for distribution. To access the releases, you can checkout
the version tag, e.g. `git checkout v3.1`.
