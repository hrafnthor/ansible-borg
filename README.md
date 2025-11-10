# Ansible Borg
An Ansible role for installing Borg backup [link](https://borgbackup.readthedocs.io)

## Notes

This role offers up three different ways of installing `borg`:

1. `distribution`: Uses the distribution package manager to install a prepackaged version.
2. `pipx`: Uses a pre-exising (see below) version of `pipx` to install `borg` on the global level.
3. `standalone`: Downloads a pre-built artifact matching the operating system and architecture definitions given. See details about requirements in the role variable section below.

When choosing to install `borg` via installation method `pipx` this role does not install `pipx` onto the system automatically. For that I can recommend another of my Ansible roles [ansible-pipx](https://github.com/hrafnthor/ansible-pipx).

### Requirements


This role requires the `ansible.utils` collection be installed from Ansible-Galaxy via:

```shell
ansible-galaxy collection install ansible.utils
```

It then also requires the `jsonschema` Python package be installed. It can be installed via pip:

```shell
pip3 install jsonschema
```

### Role variables

```yaml
borg:
  remove: [bool]                Removes the currently installed version
  force: [bool]                 Will attempt to force install the requested version.
  install: [string]             [required] Indicates the method by which Borg should be installed. Can be 'distribution', 'pipx' or 'standalone'. See discussion on differences below.
  version: [string]             The package version to install. Needs to match the version used for the given installation method. If omitted, 'latest' will be used.
  checksum: [string]            The version artifact checksum. Only used if installation method is 'standalone'.
  pipx_path: [string]           The path to a pipx executable. Only required if the pipx can not be executed as a Python module.
  dependencies: [string array]  List of dependent packages require for pipx installation. Will be installed via distribution package manager.
  add_to_path: [bool]           Indicates that the downloaded artifact should be added to the global path. Defaults to true.
  os: [string]                  Indicates the operating system that a Borg artifact should be downloaded for. Can take values 'linux', 'macos-13', 'macos-14' and 'freebsd-14'. Required if installing via 'standalone'.
  architecture: [string]        The system architecture for which a Borg artifact should be downloaded for. Can take values 'x86_64' or 'arm64'. Required if installing via 'standalone'.
  glibc: [int]                  The integer version of glibc for which a Borg artifact should be downloaded for. Version codes have their dot notation removed (i.e for version 2.31 pass in 231). Required when installing via 'standalone' and for os == 'linux'.
```

For information on dependency requirements when installing for `pipx` please see Borg's [own documentation](https://borgbackup.readthedocs.io/en/stable/installation.html).

### Defaults

The following are standalone variables, and their default values, that can be changed.

`hth_borg_default_package_name`:

The name of the package to install. Applies only in case of 'pipx' and 'standalone' installs.

Default value is `borgbackup`.

`hth_borg_default_executable_name`:

The name of the Borg executable that is added to the path. Applies only in case of 'standalone' installs when `borg.add_to_path` is not false.

Defaults to `borg`.

`hth_borg_default_destination_path`:

The destination to where standalone versions will be stored. Applies only in case of 'standalone' installs.

Defaults to `/opt/borg`.

`hth_borg_default_destination_owner`:

The owner of the destination path. Applies only in case of 'standalone' installs.

Defaults to `root`.

`hth_borg_default_destination_group`:

The group owner of the destination path. Applies only in case of 'standalone' installs.

Defaults to `root`.

`hth_borg_default_destination_mode`:

The chmod of the destination path. Applies only in case of 'standalone' installs.

Defaults to `0755`.

`hth_borg_default_symlink_path`:

The location to where the version executable should be symlinked to, when adding to the path. Applies only in case of 'standalone' installs.

Defaults to `/usr/bin`.

`hth_borg_default_symlink_owner`:

The owner of the symlink path. Applies only in case of 'standalone' installs.

Defaults to `root`.

`hth_borg_default_symlink_group`:

The group owner of the symlink path. Applies only in case of 'standalone' installs.

Defaults to `root`.

`hth_borg_default_symlink_mode`:

The chmod of the symlink path. Applies only in case of 'standalone' installs.

Defaults to `0755`.

`hth_borg_default_os_architecture_map`

The mapping of supported operating systems and architectures. Applies only in case of 'standalone' installs.

Defaults to

```
  linux: ["x86_64", "arm64"]
  macos-13: ["x86_64"]
  macos-14: ["arm64"]
  freebsd-14: ["x86_64"]
```

#### Example

The following definition installs the latest distribution version:

```yaml
borg:
  install: distribution
  version: latest
```

The following definition installs the standalone version 1.4.2:

```yaml
borg:
  install: standalone
  version: "1.4.2"
  os: linux
  architecture: x86_64
  glibc: 231
  checksum: "sha256:8a100a084d5cf0ce17f888e203ab487abfb4859a722fa6f4b4a9ad3c201753ec"
```

The following definition installs the same version via pipx on a Debian based system:

```yaml
borg:
  install: pipx
  version: "1.4.2"
  pipx_path: /usr/bin/pipx
  dependencies:
    - python3
    - python3-dev
    - python3-pip
    - python3-virtualenv
    - python3-pkgconfig
    - libacl1-dev
    - libacl1
    - libssl-dev
    - liblz4-dev
    - libzstd-dev
    - libxxhash-dev
    - build-essential
    - pkg-config
    - libfuse-dev
    - libfuse3-dev
    - fuse3
```

### Setup

Before the role can be used it needs to be added to the machine running the playbook, and as of writing this, this role is not hosted on Ansible-Galaxy only on Github.

1. Create a `requirements.yml` file in the root directory of the playbook being worked on.

2. Add the following definition inside the `requirements.yml` file:

```yml
- name: hth-borg
  src: https://github.com/hrafnthor/ansible-borg.git
```

3. Install the requirements by executing

```shell
ansible-galaxy install -r .requirements.yml
```

This will allow any playbook run from this machine to use the role `hth-borg`

#### Example Playbook

```yaml
- hosts: all
    vars:
      borg:
        install: pipx
        version: "1.4.2"
        pipx_path: /usr/bin/pipx
        dependencies:
          - python3
          - python3-dev
          - python3-pip
          - python3-virtualenv
          - python3-pkgconfig
          - libacl1-dev
          - libacl1
          - libssl-dev
          - liblz4-dev
          - libzstd-dev
          - libxxhash-dev
          - build-essential
          - pkg-config
          - libfuse-dev
          - libfuse3-dev
          - fuse3
  roles:
     - hth-borg
```

### License

```
Copyright 2025 Hrafn Thorvaldsson

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
