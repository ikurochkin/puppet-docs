---
layout: default
title: "Upgrading Puppet 3.8.x agents to 6.x"
---

[Hiera]: /hiera/
[puppet_agent]: https://forge.puppet.com/puppetlabs/puppet_agent
[moved]: ./whered_it_go.html
[facter]: /facter/
[Puppet Platform]: ./puppet_platform.md

Although there are a lot of changes to Puppet agent configuration from Puppet 3.8 to Puppet 4, the process of upgrading agents to Puppet 5 can be automated in a way that server upgrades can't.

### Decide how to upgrade your nodes

We provide a module called [`puppet_agent`][puppet_agent] to simplify upgrades from Puppet 3 to 4.

> *Note:* This module has not yet been tested for upgrades to Puppet 5, but the process should be the same.

If you're running Puppet on Windows or [any supported Linux operating system](./system_requirements.html#platforms-with-packages), this module can automatically upgrade Puppet, and all of their dependencies on agents.

If you're running Puppet on other operating systems, you can't upgrade them with the module. You can either upgrade your agents manually or automate the process yourself.

The next steps explain both methods.

### Upgrade with the `puppet_agent` module

> **Note**: This module works on Windows and supported Linux distributions. If your agents run any other operating systems, skip to ["Upgrade Manually or Build Your Own Automation"](#upgrade-manually-or-build-your-own-automation).

The `puppet_agent` module does the following things for you:

-   Enables the [Puppet Platform][] repository, if applicable.
-   Installs the latest version of the `puppet-agent` package, which replaces the installed versions of Puppet, [Facter][], [Hiera][].
-   Copies Puppet's SSL files to their new location.
-   Copies your old `puppet.conf` to its [new location][moved], and cleans out old settings that we either removed in Puppet 4 or needed to revert to their default values.
-   Ensures the Puppet service is running.

1.  Install the module on Puppet servers.

    -   If you manage your Puppet code manually, install it by running `puppet module install puppetlabs/puppet_agent --environment <ENVIRONMENT>`
    -   If you manage your code with [r10k]({{pe}}/r10k.html), add the module and its dependencies to your Puppetfile.
    -   If you manage your code some other way, install `puppet_agent` as you would any other module.

2.  Assign the `puppet_agent` class to nodes.

    However you classify nodes --- whether in the [main mainfest](./dirs_manifest.html), with an [external node classifier](./nodes_external.html), [Hiera][], or some other method --- classify your agents with `puppet_agent`.

    You can also [configure the module](https://forge.puppet.com/puppetlabs/puppet_agent/readme#usage) to control which services start or to force a different architecture on Windows.

    Carefully control and monitor the rollout. Assign the class in a development or test environment to ensure it works as expected on systems similar to your production environment. Roll it out to your live agents in phases, and monitor the upgraded agents for issues.

3.  Post-upgrade clean-up

    After you've upgraded your entire deployment, do the [post-upgrade clean-up tasks](./upgrade_major_post.html).

### Upgrade manually or build your own automation

To upgrade agents without using the `puppet_agent` module, you can either install the upgrades manually or design your own upgrade automation.

1.  Install the new version of Puppet.

Find your operating system in the sidebar navigation to the left and follow the Puppet agent installation instructions.

2.  Move SSL files (\*nix only).

    On \*nix systems, we moved the default [`confdir`](./dirs_confdir.html) to `/etc/puppetlabs/puppet` in Puppet 4. Since the default [`ssldir`](./dirs_ssldir.html) is `$confdir/ssl`, its location changes during the upgrade.

    In Puppet 3, the default `ssldir` is `/etc/puppet/ssl`; some systems might also use  `/var/lib/puppet/ssl`.

    Locate your [`ssldir`](./dirs_ssldir.html) in `/etc/puppet/puppet.conf` and move that directory's contents to `/etc/puppetlabs/puppet/ssl` without changing the files' permissions. For example, run `sudo cp -rp /var/lib/puppet/ssl /etc/puppetlabs/puppet/ssl`.

3.  Reconcile `puppet.conf`

    On \*nix systems, we moved [`puppet.conf`](./config_file_main.html) from `/etc/puppet/puppet.conf` to `/etc/puppetlabs/puppet/puppet.conf`. Either edit the new `puppet.conf` file or copy your old version. (We didn't change `puppet.conf`'s location on Windows.)

    Examine the new `puppet.conf` regardless of your operating system and confirm that:

    -   It includes any necessary modifications.
    -   It excludes any settings that were [removed in Puppet 4.0](/puppet/3.8/deprecated_settings.html). Notably, if you set `stringify_facts=false` [before upgrading](./upgrade_major_pre.html), remove this setting.
    -   All [important settings](./config_important_settings.html#settings-for-puppet-master-servers) are correctly configured for your site.

4.  Start service or update cron job.

    We also moved Puppet binaries to `/opt/puppetlabs/bin` in Puppet 4. If you run Puppet as a service, configure it to launch at boot using `/opt/puppetlabs/bin/puppet resource`:

    `/opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true`

    On Windows, use the same command but omit the `/opt/puppetlabs/bin/` prefix.

    If you use a cron job to periodically run `puppet agent -t` on your \*nix systems, edit the job and update the `puppet` binary's path to `/opt/puppetlabs/bin/puppet`.


5.  After you've upgraded your entire deployment, do the [post-upgrade clean-up tasks](./upgrade_major_post.html).