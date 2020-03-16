# airsigner

Automates signing OpenPGP public keys when using offline, airgapped computer.

Why not caff and pius?

The idea was to minimize the amount of trusted code that needs to run on the
offline computer. caff is a [2000 lines perl script][CAFF] while pius is an
[entire project][PIUS] full of various options.

[CAFF]: https://salsa.debian.org/stappers/pgp-tools/-/blob/master/caff/caff
[PIUS]: https://github.com/jaymzh/pius/blob/master/libpius/signer.py

air signer on the other hand is just a simple 40 lines Bash script. The savings
come from the fact that air signer uses new GnuPG features (such as `--quick-sign-key`)
and that it doesn't send the e-mails leaving these tasks to other software.

## Online computer:

First, convert KSP list into line format:

    cat ksp-fosdem2020.txt | ./online/convert-ksp-list > offline/converted.txt

Then, edit `converted.txt` and remove lines that you don't want to
sign. Don't edit anything.

Now, we'll need keys of all these people. Make sure you have them in
your keyring (e.g. by importing the KSP keyring with `gpg --import`).

Then run:

    cat offline/converted.txt | ./online/export-keys

That will export all keys as files with `gpg` extension.

Copy `offline` directory and all exported `gpg` files to offline computer.

## Offline computer:

Customize `inner-1.eml` because that's what your recipients will see.

Note: Currently `outer-1.eml` also needs to be modified adjusting `From` header.

**Carefully** read and customize `offline/sign-and-create-emails` as this will
take the converted list, import keys, sign them, export into an encrypted
e-mails and remove signed keys.

Check once again that `converted.txt` contains the list of keys
and identities you want to sign.

Then run it with:

    cat converted.txt | ./sign-and-create-emails

Copy all `eml` files to online computer.

## Online computer:

Check `eml` files for e-mail addresses. If they are missing you need to
add them manually.

Then send e-mails with:

    git send-email --suppress-cc=all --quiet --confirm=never 001_0.eml

This assumes that [you have `git send-email` configured][GSE]. It's also
possible to use any other sending mechanism.

[GSE]: https://git-send-email.io/

