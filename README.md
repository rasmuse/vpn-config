# OpenVPN configuration

## Setting up a certificate authority (CA)

Install easy-rsa

```
sudo apt-get install easy-rsa
```

Create a separate user for the CA:

```
sudo adduser --home /home/vpnca vpnca
```

Now log in as the `vpnca` user. For example, in a terminal emulator:

```
su vpnca
```

Then, as the `vpnca` user,

```
# Copy easy-rsa to a place where it will not be changed by someone else
cp -r /usr/share/easy-rsa ~

# Protect home dir (and easy-rsa dir)
chmod -R go-rwx ~
```

Now edit the CA identity in `~/easy-rsa/vars`. Specifically, make sure you set values for the following:

```
KEY_COUNTRY
KEY_PROVINCE
KEY_CITY
KEY_ORG
KEY_EMAIL
KEY_OU
```

Also, in `~/easy-rsa/vars` set the `KEY_SIZE` to at least 2048 bits. But computer power is cheap, so I choose 4096 bits at least for the CA.

Now set up the CA as follows:
```
cd ~/easy-rsa
. ./vars
./clean-all
./build-ca
```

The last command `./build-ca` launches an interactive wizard where you get to confirm all your settings. If you are satisified, just press return repeatedly until it finishes.

Now the CA public key `ca.crt` and private key `ca.key` should be found in `~/easy-rsa/keys/`. Keep the `ca.key` in a safe place.

### Signing certificates

There are two methods:

    1. Create key pairs using the CA machine, sign them there, and give/send them over a secure channel to the servers.
    2. Send the CA public key to server machines, make key pairs there and send Certificate Signing Requests (CSRs) to the CA.

        * Note: If this method is used it is of course necessary that the `ca.crt` is not tampered with, so it should in principle be sent over a secure channel and/or checked for integrity upon arrival.

#### To make them on the CA machine

```
cd ~/easy-rsa
./build-key-server [server-name]
```

When certificates are signed, they end up at `keys/[server-name].crt` and `keys/[server-name].key`.

#### To make them on the user's machine

* Get the `ca.crt` file to the user and verify its integrity.
* Install and configure easy-rsa and run `pkitool` with or without the `--server`` option as appropriate:
    ```
    ./pkitool --server [server-name]
    ```
    or
    ```
    ./build-req [client-name]
    ```

    Now put the request file `keys/[name].csr` to the CA `keys` directory and sign it by running `./sign-req [name]` or `./sign-req --server [name]`. 

    Note that the `--server` option is needed if the CSR was created using `pkitool --server`. If you miss this, the certificate will be signed, but it will not pass the security test an OpenVPN client makes when configured with the option `remote-cert-tls server`.
