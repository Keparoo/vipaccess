### VipAccess

This virtual environment includes a utility called [python-vipaccess](https://github.com/dlenski/python-vipaccess)



## Usage

### Provisioning a new VIP Access credential
This is used to create a new VIP Access token. It connects to https://services.vip.symantec.com/prov and requests a new token, then deobfuscates it, and checks whether it is properly decoded and working correctly, via a second request to https://vip.symantec.com/otpCheck.

By default it stores the new token in the file `.vipaccess` in your home directory (in a format similar to `stoken`), but it can store to another file instead, or instead just print out the "token secret" string with instructions about how to use it.
```bash
# First activate the virtual environment
source .venv/bin/activate

usage: vipaccess provision [-h] [-p | -o DOTFILE] [-t TOKEN_MODEL]

optional arguments:
  -h, --help            show this help message and exit
  -p, --print           Print the new credential, but don't save it to a file
  -o DOTFILE, --dotfile DOTFILE
                        File in which to store the new credential (default
                        ~/.vipaccess)
  -i ISSUER, --issuer ISSUER
                        Specify the issuer name to use (default: Symantec)
  -t TOKEN_MODEL, --token-model TOKEN_MODEL
                        VIP Access token model. Often SYMC/VSMT ("mobile"
                        token, default) or SYDC/VSST ("desktop" token). Some
                        clients only accept one or the other. Other more
                        obscure token types also exist:
                        https://support.symantec.com/en_US/article.TECH239895.html
```

Here is an example of the output from `vipaccess provision -p`:

```bash
Generating request...
Fetching provisioning response from Symantec server...
Getting token from response...
Decrypting token...
Checking token against Symantec server...
Credential created successfully:
	otpauth://totp/VIP%20Access:SYMC12345678?secret=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA&issuer=Symantec&algorithm=SHA1&digits=6
This credential expires on this date: 2019-01-15T12:00:00.000Z

You will need the ID to register this credential: SYMC12345678

You can use oathtool to generate the same OTP codes
as would be produced by the official VIP Access apps:

    oathtool    -b --totp AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  # output one code
    oathtool -v -b --totp AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  # ... with extra information
```

Here is the format of the .vipaccess token file output from vipaccess provision [-o ~/.vipaccess]. (This file is created with read/write permissions only for the current user.)

```bash
version 1
secret AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
id SYMC12345678
expiry 2019-01-15T12:00:00.000Z
```

### Display a QR code to register your credential with mobile TOTP apps
Once you generate a token with `vipaccess provision`, use `vipaccess uri` to show the `otpauth://` URI and [qrencode](https://fukuchi.org/works/qrencode/manual/index.html) to display that URI as a QR code:

```bash
$ qrencode -t UTF8 'otpauth://totp/VIP%20Access:SYMCXXXX?secret=YYYY&issuer=Symantec&algorithm=SHA1&digits=6'
```

Scan the code into your TOTP generating app, like [FreeOTP](https://en.wikipedia.org/wiki/Initiative_For_Open_Authentication) or [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2).

### Generating access codes using an existing credential

The `vipaccess [show]` option will also do this for you: by default it generates codes based on the credential in `~/.vipaccess`, but you can specify an alternative credential file or specify the OATH "token secret" on the command line.

```bash
usage: vipaccess show [-h] [-s SECRET | -f DOTFILE]

optional arguments:
  -h, --help            show this help message and exit
  -s SECRET, --secret SECRET
                        Specify the token secret on the command line (base32
                        encoded)
  -f DOTFILE, --dotfile DOTFILE
                        File in which the credential is stored (default
                        ~/.vipaccess
```

As alluded to above, you can use other standard [OATH](https://en.wikipedia.org/wiki/Initiative_For_Open_Authentication-based tools to generate the 6-digit codes identical to what Symantec's official apps produce.