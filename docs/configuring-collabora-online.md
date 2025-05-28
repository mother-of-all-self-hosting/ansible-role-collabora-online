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

### Set a password for the admin interface

You also need to set a password for the admin interface by adding following configuration to your `vars.yml` file. Make sure to replace `YOUR_ADMIN_INTERFACE_PASSWORD_HERE` with your own value. Note that **only alphanumeric characters are accepted**.

```yaml
collabora_online_environment_variable_password: YOUR_ADMIN_INTERFACE_PASSWORD_HERE
```

The default username is set to `admin`. It can be changed by specifying another one with `collabora_online_environment_variable_username`.

The admin interface will be available at `example.com/browser/dist/admin/admin.html`.

### Define a WOPI host to connect

To integrate a CODE instance to another software, you can use the WOPI (Web Application Open Platform Interface) protocol (refer to [this page on the official documentation](https://sdk.collaboraonline.com/docs/introduction.html?highlight=wopi) for how the protocol is related to CODE), and this role is configured to apply it as [this documentation at CODE](https://sdk.collaboraonline.com/docs/installation/CODE_Docker_image.html#how-to-configure-docker-image) instructs.

To define an allowed WOPI host, you can add and adjust following configuration to your `vars.yml` file:

```yaml
collabora_online_environment_variable_aliasgroup1: "https://<domain1>:443,https://<your-dot-escaped-aliasname1>|<your-dot-escaped-aliasname2>:443"
```

- `domain1` — the WOPI host, i.e. your preferred File Sync and Share solution that implements the WOPI protocol, for example `share.example.com`
- `your-dot-escaped-aliasname|your-dot-escaped-aliasname2` — the aliasnames with which you can access the same WOPI host (in this case `domain1`). These aliasnames can be specified with regular expressions.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `collabora_online_environment_variables_additional_variables` variable

For a complete list of CODE's config options that you could put in `collabora_online_environment_variables_additional_variables`, see its [environment variables](https://sdk.collaboraonline.com/docs/installation/CODE_Docker_image.html#setting-the-application-configuration-dynamically-via-environment-variables).

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, a CODE instance becomes available at the specified hostname like `https://example.com`.

To use it, you need to integrate it with a File Sync and Share solution that implements the WOPI protocol.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu collabora-online` (or how you/your playbook named the service, e.g. `mash-collabora-online`).
