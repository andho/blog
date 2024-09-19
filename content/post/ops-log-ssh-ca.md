+++
date = "2021-04-06"
slug = "ops-log-starting-ssh-ca-project"
status = 'publish'
title = "Ops Log: Starting SSH CA project"
Categories = ["Linux Admin"]
Tags = ["ssh", "operations", "linux", "ops-log"]
+++

The following text is an ops log entry. A brain dump.

Some research was done before but nothing was recorded. The first google search
pointed to a project called https://github.com/cloudtools/ssh-cert-authority.
This project automates the process of signing and allows delegation of signing
requests to multiple signers, set number of signatures required and have
separate environments for different CAs. All their features are great and the
setup and usage of the tool is easy, and it can also be setup to use Google
Cloud or AWS KMS.

But for the time being we can go with a manual signing. As the setup for
ssh-cert-authority begins with manual setup of an SSH CA it would be easy to
convert to signing with ssh-cert-authority later.

Next thing to look at is how to setup “Principals” so that it is not user based
but role based. For example access to database servers can be based on the
principal “dba-prod” and “dba-dev”. Both the server’s and the user’s cert has to
have this principal. Looking at https://cottonlinux.com/ssh-certificates/ it’s
quite simple to set this up.

So at this point I think I have the information needed to finish up the setup
and take it for a test run.

First thing to setup the CA. Ideally we will have the root private key protected
by sharding or HSM but for now let’s get a simple setup ready.

## Generating the CA key pair.

Generating the CA key pair is the same as generating any SSH key pair, but it’s
imperative that this key pair is well protected and to make sure that it is not
lost. Let’s call this key pair “CA key pair”.

{{< highlight bash >}}
ssh-keygen -f <key-pair-name>
{{< /highlight >}}

You could generate the key pair into an encrypted storage to make sure that it
cannot be extracted from the hard disk.

Now that the CA key pair is generated, let’s first setup a server that will
trust the CA.

Copy the CA public key and paste into the chosen remote server into a file
called **/etc/ssh/ca_keys**. This file can have multiple keys, one on each line. For
now, the file will have just the one public key. Afterwards restart the server.

## Test out a client cert
Now that the server trusts the CA, let’s test it out. We’ll have to generate a
certificate that has a principal that matches the username of the user we want
to log into. Let’s add the principal ‘user’ and test logging into the ‘user’
user. Later we will create a new certificate with principals based on the usage
instead of usernames, but for now we are just testing the trust.

{{< highlight bash >}}
ssh-keygen -V +1h -s acme_cert_authority -I andho -n user ~/.ssh/id_ed25519.pub
{{< /highlight >}}

This will generate a file called ~/.ssh/id_ed25519-cert.pub. Let’s look at this
cert in a human readable form.

{{< highlight bash >}}
ssh-keygen -L -f ~/.ssh/id_ed25519-cert.pub
{{< /highlight >}}

We can see the Key ID and the principal we specified in the signing command. I
should be able to login to the server with the cert now. To test this I removed
my public key `~/.ssh/id_ed25519.pub` from the authorized_keys file on the
server so that the authentication will solely depend on the CA.

## Verifying the host using CA

Next step is to sign the servers host key so that trusting the server is also
based on the CA and not left to the wit of the user.

First copy the public key you want to use to the CA and sign it.

{{< highlight bash >}}
scp user@server.example.com:~/ssh_host_rsa_key.pub ./
ssh-keygen -s acme_cert_authority -I “server.example.com host key” \
-n “server.example.com,server,10.0.0.1” -V always:forever \
-h ssh_host_rsa_key.pub
{{< /highlight >}}

Then copy the signed cert back to the server and place in in the same location
as the host key pair.

{{< highlight bash >}}
scp ./ssh_host_rsa_key-cert.pub user@server.example.com:~/
ssh user@server.example.com
mv ./ssh_host_rsa_key-cert.pub /etc/ssh/
{{< /highlight >}}

Then add the HostCertificate to the sshd_config.

{{< highlight bash >}}
echo “HostCertificate /etc/ssh/ssh_host_rsa_key-cert.pub” \
  >> /etc/ssh/sshd_config
systemctl restart sshd
{{< /highlight >}}

To test this, first I removed the existing host keys from known_hosts file.

{{< highlight bash >}}
ssh-keygen -R server.example.com
{{< /highlight >}}

Then add the CA public key as @cert-authority.

{{< highlight bash >}}
echo “@cert-authority *.example.com $(cat acme_cert_authority.pub)” \
>> ~/.ssh/known_hosts
{{< /highlight >}}

And now I’m able to login to the server without verifying the individual host
file as long as the server has a host certificate signed by the CA.

## Log the cert owner

Let’s also make it so that the auth log will show the key ID as well, because
multiple people are using the same user and this will help us to trace exactly
who logged in at a specific time. This is done by setting the
`LogLevel VERBOSE` in sshd_config file.

## Role based access using principals

Now let’s setup the principals based on roles instead of users. I created a
directory that will contain a file for each user in the server. In this file,
the principals that are allowed to log in to that server as that user is listed.

{{< highlight bash >}}
mkdir /etc/ssh/ca_principals
echo “syseng” > /etc/ssh/ca_principals/user
{{< /highlight >}}

Now I sign my public key again. The previous public key has expired anyway, but
this time I’m giving an expiry of 1 year.

{{< highlight bash >}}
ssh-keygen -V -1m:+365d -s acme_cert_authority -I andho \
-n syseng ~/.ssh/id_id25519.pub
{{< /highlight >}}

And that’s it. Now I’m able to login to the server with principal syseng.

What’s left to do is to do the host certificate, user ca key and principal setup
on all the servers, figure out which principals to assign to the different users
and finally get all the users public keys and sign them with the correct
principals and give them the certificats.

## Links

* https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-creating_ssh_ca_certificate_signing-keys
* https://smallstep.com/blog/use-ssh-certificates/
* https://engineering.fb.com/2016/09/12/security/scalable-and-secure-access-with-ssh/
* https://github.com/vedetta-com/vedetta/blob/master/src/usr/local/share/doc/vedetta/OpenSSH_Principals.md
* https://dmuth.medium.com/ssh-at-scale-cas-and-principals-b27edca3a5d
* https://cottonlinux.com/ssh-certificates/
* https://www.lorier.net/docs/ssh-ca.html
