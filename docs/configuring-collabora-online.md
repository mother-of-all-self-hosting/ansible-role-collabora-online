<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Collabora Online Development Edition (CODE)

This is an [Ansible](https://www.ansible.com/) role which installs [Collabora Online Development Edition (CODE)](https://www.collaboraonline.com/code/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

CODE is the development version of [Collabora Online](https://www.collaboraonline.com/), which enables you to edit office documents online with integrations such as [Nextcloud](https://nextcloud.com/office/), [OwnCloud](https://owncloud.com/), and [XWiki](https://xwiki.com/en/Blog/Collabora-Connector-Application/).

See the project's [documentation](https://www.collaboraonline.com/code/) to learn what CODE does and why it might be useful to you.

>[!NOTE]
> While there are several software porivided by Collabora Productivity Ltd such as Collabora Online, Collabora Office, this role is specifically configured to install a CODE instance.

## Prerequisites

As CODE runs on the server and is interacted with a standard browser, there is no need for any client installation. However, please keep in mind that it is necessary to integrate it with another software which functions as a data storage and manages access control for users (see the entry "Do the programs run in a browser and / or as local clients?" on the [FAQs](https://www.collaboraonline.com/faqs/)). **It is not possible to edit office documents only with a CODE instance.**

If you are looking for an Ansible role for Nextcloud, you can check out [this role (ansible-role-nextcloud)](https://github.com/mother-of-all-self-hosting/ansible-role-nextcloud) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team, which can integrate CODE with a Nextcloud instance.

## Adjusting the playbook configuration

To enable CODE with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# collabora-online                                                     #
#                                                                      #
########################################################################

collabora_online_enabled: true

########################################################################
#                                                                      #
# /collabora-online                                                    #
#                                                                      #
########################################################################
```

### Set the hostname

To enable the CODE instance you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
collabora_online_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting CODE under a subpath (by configuring the `collabora_online_path_prefix` variable) does not seem to be possible due to CODE's technical limitations.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `collabora_online_environment_variables_additional_variables` variable

For a complete list of Collabora Online's config options that you could put in `collabora_online_environment_variables_additional_variables`, see its [environment variables](https://docmost.com/docs/self-hosting/environment-variables).

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Collabora Online becomes available at the specified hostname like `https://example.com`.

To get started, go to the URL on a web browser and create a first workspace by inputting required information. For an email address, make sure to input your own email address, not the one specified to `collabora_online_environment_variable_mail_from_address`.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu docmost` (or how you/your playbook named the service, e.g. `mash-docmost`).
