# Documenation

1. Creating an RSA key and generating a `jwk` object from it
2. Signing rulesets with this key
3. Publishing those rulesets somewhere
4. Getting users to use your update channel

### 1. Creating an RSA key and generating a `jwk` object from it

To create an RSA key, issue the following command (either on your development machine if you are not using an airgapped process, or the airgap if you are):

```shell
openssl genrsa -out ./keys/key.pem 4092
```

Your RSA keypair will now be stored in the file `key.pem`.  To generate a corresping public key, issue this command:

```shell
openssl rsa -in ./keys/key.pem -outform PEM -pubout -out ./keys/public.pem
```

Your public key is now stored in `public.pem`.  If you are using an airgap, copy this public key (with whatever method is safest in your setup, perhaps with the `qr` command) to your development environment.

At this point, you will need to generate a `jwk` object from the public key.  This can be done with the `pem-jwk` node package.  You'll have to download node.js and npm, or just issue this command in docker:

    sudo docker run -it -v $(pwd):/opt --workdir /opt node bash

And you will be booted into a node environment.  Next, run

```shell
npm install -g pem-jwk
```

This will install the `pem-jwk` package globally.  You can now run

```shell
cat ./keys/public.pem | pem-jwk > ./keys/public.jwk.json
```

And you should see a `jwk` object displayed.  Take note of this, you will need it later.

**TLDR**

```bash
openssl genrsa -out ./keys/key.pem 4092
openssl rsa -in ./keys/key.pem -outform PEM -pubout -out ./keys/public.pem
npm install -g pem-jwk
cat ./keys/public.pem | pem-jwk > ./keys/public.jwk.json
```

### 2. Signing rulesets with this key

#### Setup

On your development machine, clone or download the HTTPS Everywhere repository.  Since it's quite large, it will suffice to do a shallow clone:

```shell
git clone https://github.com/Jackymancs4/https-everywhere-rulesets.git
```

Next,

```shell
cd https-everywhere-rulesets
rm rules/*.xml
```

This will remove all the rulesets bundled with the extension itself.  All the rulesets you want to sign for your update channel must be in the `rules` directory before moving to the next step.  Generate an example ruleset or use your own ruleset:

```shell
./make-trivial-rule example.com
```

**TLDR**

```shell
./make-trivial-rule example.com
```

#### Signing

You will need python 3.6 on your system or available via docker for the next step.

```shell
sudo docker run -it -v $(pwd):/opt --workdir /opt python:3.6 bash
```

Next, run

```shell
python3 merge-rulesets.py
```

You should see the following output:

```shell
 * Parsing XML ruleset and constructing JSON library...
 * Writing JSON library to rules/default.rulesets
 * Everything is okay.
```

This prepares the file you are about to sign.  If your do not have an airgap, run the following command:

```shell
./standalone.sh ./keys/key.pem ./rulesets/v1/
```

If you have an airgapped setup, run the following command on your development machine:

```shell
./async-request.sh ./keys/public.pem ./rulesets/v1/
```

This will display a hash for signing, as well as a metahash.  On your airgap machine, run the `async-airgap.sh` script that you had previously copied to it:

```shell
./async-airgap.sh ./keys/key.pem SHA256_HASH
```

typing the hash carefully.  Check the metahash to make sure it is the same as what was displayed on your development machine.  This will output base64-encoded data as well as a QR code representing that data that you can scan, and send that data to your development machine.  Once you have that data from the QR code pasted into the development machine prompt, press Ctrl-D and you should have output indicating that your rulesets have been signed successfully.

**TLDR**

```shell
python3 merge-rulesets.py
./standalone.sh ./keys/key.pem ./rulesets/v1/
```

### 3. Publishing those rulesets somewhere

Once you've signed the rulesets successfully, choose a public URL to make these rulesets accessible.  You may want to use a CDN if you expect a lot of traffic on this endpoint.  Your rulesets as well as their signatures are stored in `./dist` you chose above, you need only to upload them to an endpoint your users can access.

### 4. Getting users to use your update channel

Once you've established an update channel by publishing your rulesets, you'll want to let your users know how to use them.  From step 1 above, you have a `jwk` object.  You may want to also only allow modification of certain URLs, using the `scope` field.  The `update_path_prefix` field will simply be the public URL that you chose in step 3.

If your users are using a custom build of HTTPS Everywhere (such as in a corporate LAN environment), you can take a look at `./keys/jwk.json` to include a new update channel in the same format as the EFF update channel.

In most cases, your users will just be using a standard HTTPS Everywhere build.  In this case, they will have to add your update channel using the UX, as explained below.

## Adding and Deleting Update Channels

In addition to being defined in `update_channels.js`, users can add additional update channels via the extension options.

In Firefox, enter `about:addons` into the URL bar, then click on `Extensions` on the left navbar, then click `Preferences` next to the HTTPS Everywhere extension.

In Chrome, right-click on the HTTPS Everywhere icon and click `Options`.

You will now see the HTTPS Everywhere options page.  Click `Update Channels`.

You will now see a list of update channels, with `EFF (Full)` being the first.  Below, you can add a new update channel.  Once you hit `Update`, the channel will download a new ruleset release (if available) from the channel.

If a new ruleset update is available, after a few seconds you should now see the new ruleset version in the bottom of the extension popup:

You can also delete rulesets from the extension options.  Under `Update Channels`, just click `Delete` for the channel you want to delete.  This will immediately remove the rulesets from this update channel.
