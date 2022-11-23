# Testing for PSWSMan

As testing PSWSMan requires a Windows host to talk to it is hard to test all the various edge cases and scenarios through CI.
This document is designed to walk through the integration test setup to run the full suite of tests against on the current host.
The test environment to create the WinRM server and domain environment is provided by [wsman-environment](https://github.com/jborean93/wsman-environment).

The `wsman-environment` repo will spin up a few virtual machines with the following:

+ A domain controller under `dc.wsman.env`
+ A target WSMan server under `test.wsman.env`
+ A target Windows server for testing things like double hop authentication or running the module on a domain joined Windows host
+ A Linux host that has a proxy server for testing proxy configurations

The target WSMan server has the following features

+ All auth methods enabled; `Basic`, `Certificate`, `Negotiate`, `CredSSP`
+ A local admin account for testing local account access
+ A HTTPS listener signed by a certificate authority that `wsman-environment` provides
+ Multiple WinRM listeners that cover:
    + HTTPS listeners signed by a CA provided by `wsman-environment`
    + Multiple signing algorithms, `sha1`, `sha256`, `sha512`, etc
    + Ports blocked by direct access for proxy verification
+ A Just Enough Administration endpoint

To set up this environment you will need both `Vagrant` and `Ansible` to be installed.
Once installed run the following:

```bash
git clone https://github.com/jborean93/wsman-environment.git
cd wsman-environment
ansible-galaxy collection install -r requirements.yml

vagrant up

ansible-playbook main.yml -vv
```

This may take some time to complete and if the Ansible playbook fails, running it again should clear out any errors.
Once setup a file in the root of the `PSWSMan` repository called `test.settings.json` can be filled out with all the details of the environment to test against.
TODO: Add script that will generate this from wsman-environment path.

On macOS, the CA certificate generated by `wsman-environment` needs to be loaded in the system trust store.
This can be done using the following commands to load the CA cert at the `~/ca.pem` path.

```bash
sudo security authorizationdb read com.apple.trust-settings.admin > rights
sudo security authorizationdb write com.apple.trust-settings.admin allow
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ~/ca.pem
sudo security authorizationdb write com.apple.trust-settings.admin < rights
```

Once the tests are completed the certificate can be removed from the `Keychain Access` application.