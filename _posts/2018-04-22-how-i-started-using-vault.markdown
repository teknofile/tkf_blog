---
layout: post
title: "How I Started Using Vault"
date: 2018-04-22 00:00:00 -0700
categories: development vault home
---

<p>Vault. It's the "new" thing (it's actually not that new).</p>
<p>Secrets management. I'm sure you already know what it is if you're here. What I did to get using it:</p>
<p>First, I had to find a host that would run it well enough. I knew I wanted to make my life simple and run it in a docker image. I have a relatively small home network setup, a few subnets, that will need access. I tried to run it on an old RaspberryPi model B+ (one of the original 32bit ones) that I had laying around. Docker on the RPi sucks.</p>
<p>Next, I thought about just running it on an old MacMini 2010 (Dual-Core 4GB Ram). Yeah - no dice there either as Docker CE won't run on that Mac without virtualization extensions in the CPU. Which I thought was weird, since I use it to run several VirtualBox hosts.</p>
<p>Fine - I'll run it on one of the generic servers I have.</p>
<p>For my (at least initial) use cases, I'm not going to get fancy with a highly available backend. One of these days, I may actually look into it with an S3 backend too.</p>
<p> </p>
<p><em><strong>But not today.</strong></em></p>
<p> </p>
<p>Next, on the host where we are running the vault container, create a directory to store the vault files. This will be local configuration of vault, the secrets file that will be stored on disk, and other stuff.</p>
<p>mkdir -p /srv/vault</p>
<p>Then I created a local.json file I <a href="https://www.melvinvivas.com/secrets-management-using-docker-hashicorp-vault/">stole from the interwebs</a> and tweaked it a little bit. Eventually, I'll get around to configuring TLS. But not today. I (mostly) trust my home network.</p>
<pre class="lang:default decode:true" title="local.json configuration for vault">{
    "listener": [{
            "tcp": {
                        "address": "127.0.0.1:8200",
                                    "tls_disable" : 1
                                            }
                                                }],
                                                    "storage" :{
                                                            "file" : {
                                                                        "path" : "/vault/data"
                                                                                }
                                                                                    }
                                                                                        "max_lease_ttl": "10h",
                                                                                            "default_lease_ttl": "10h",
                                                                                            }</pre>
                                                                                            <p> </p>
                                                                                            <p>Then we run the docker, binding it local filesystem into the docker instnace</p>
                                                                                            <pre class="lang:default decode:true ">docker run -d -v /srv/vault:/vault --name=tkf_vault --cap-add=IPC_LOCK vault server</pre>
                                                                                            <p> </p>
                                                                                            <p>Once the docker has been instantiated, go ahead and connected to it and get a shell:</p>
                                                                                            <pre class="lang:default decode:true ">docker exec -it tkf_vault /bin/sh</pre>
                                                                                            <p> </p>
                                                                                            <p>Let's make sure the /vault/data directory is owned by the correct users:</p>
                                                                                            <pre class="lang:default decode:true ">chown -R vault.vault /vault/data</pre>
                                                                                            <p> </p>
                                                                                            <p>And then we go ahead an initialize the vault database:</p>
                                                                                            <pre class="lang:default decode:true" title="vault operator init">/vault # vault operator init
                                                                                            Unseal Key 1: ***********************************************
                                                                                            Unseal Key 2: ***********************************************
                                                                                            Unseal Key 3: ***********************************************
                                                                                            Unseal Key 4: ***********************************************
                                                                                            Unseal Key 5: ***********************************************

                                                                                            Initial Root Token: ***********************************************

                                                                                            Vault initialized with 5 key shares and a key threshold of 3. Please securely
                                                                                            distribute the key shares printed above. When the Vault is re-sealed,
                                                                                            restarted, or stopped, you must supply at least 3 of these keys to unseal it
                                                                                            before it can start servicing requests.

                                                                                            Vault does not store the generated master key. Without at least 3 key to
                                                                                            reconstruct the master key, Vault will remain permanently sealed!

                                                                                            It is possible to generate new unseal keys, provided you have a quorum of
                                                                                            existing unseal keys shares. See "vault rekey" for more information.


                                                                                            /vault # vault status
                                                                                            Key Value
                                                                                            --- -----
                                                                                            Seal Type shamir
                                                                                            Sealed true
                                                                                            Total Shares 5
                                                                                            Threshold 3
                                                                                            Unseal Progress 0/3
                                                                                            Unseal Nonce n/a
                                                                                            Version 0.10.0
                                                                                            HA Enabled true</pre>
                                                                                            <p> </p>
                                                                                            <p>Please store your keys appropriately. If you trust your family, give one to your partner/spouse. Give one to each kid. Etc. :-)</p>
                                                                                            <p> </p>
                                                                                            <p>Go ahead and unseal the vault using 3 of the 5 tokens created from the initialize above.</p>
                                                                                            <p>For me, I want to store my AWS ACCESS_KEY/SECRET_KEY in vault:</p>
                                                                                            <pre class="lang:default decode:true">vault write secret/aws access_key=ABCDEFGHIJ0123456789 secret_key=ABCDEFGHIJ012345678901234567890123456789
                                                                                            Success! Data written to: secret/aws

                                                                                            </pre>
                                                                                            <p> </p>
                                                                                            <p>Let's see if we can re-read them from the vault:</p>
                                                                                            <pre class="lang:default decode:true">/vault # vault read secret/aws
                                                                                            Key Value
                                                                                            --- -----
                                                                                            refresh_interval 768h
                                                                                            access_key ABCDEFGHIJ0123456789
                                                                                            secretkey ABCDEFGHIJ012345678901234567890123456789</pre>
                                                                                            <p>Final thoughts -- Since we just about done there are some final things you should check on (<em>not a complete list</em>):</p>
                                                                                            <ul>
                                                                                            <li>Check the file permissions on the host OS</li>
                                                                                            <li>How do we make sure vault is always running</li>
                                                                                            <li>Do we keep the vault unsealed for our automated systems accessing it</li>
                                                                                            <li>Create authentication tokens
                                                                                            <ul>
                                                                                            <li>Don't have everything 'login' to vault as root. Grant restrictive permissions to what needs access to what.</li>
                                                                                            </ul>
                                                                                            </li>
                                                                                            <li>You'll probably want to enable port 8200 available to more than just the loopback address
                                                                                            <ul>
                                                                                            <li>Probably will want to enable TLS before doing so</li>
                                                                                            </ul>
                                                                                            </li>
                                                                                            <li>You can unseal the vault via curl:
                                                                                            <ul>
                                                                                            <li>curl -H "Content-Type: application/json" -X PUT http://${VAULT_IP}:8200/v1/sys/unseal -d '{"key": "${VAULT_UNSEAL_KEY}"}'</li>
                                                                                            </ul>
                                                                                            </li>
                                                                                            </ul>
                                                                                            <p> </p>
