<!--
SPDX-FileCopyrightText: 2018-2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently there is one testing scenario available.

### `default`

Tests a standard Prometheus SSH exporter installation.

This exporter probes SSH servers, so the scenario is built around an SSH server to probe: an OpenSSH daemon joined to the container network the role created, reached by container name over that network's DNS. Probing it exercises the real path — a TCP connection over the bridge, an SSH handshake, an authentication, a session and a command run on the far end.

- the image the role runs ships no configuration of its own and exits immediately without one, which the scenario confirms by running it bare as a negative control — so the exporter answering at all already says the role's configuration file arrived
- the running process refuses a module it was never configured with, and accepts `default`, which comes from the role's own configuration template
- a module the scenario adds through `prometheus_ssh_exporter_configuration_extension_yaml` completes a full SSH session against the target — and reports that same target down when pointed at a port where nothing listens
- the `ssh_output` metric carries the target's own hostname, which the exporter could only have learned by running a command in a session on it
- a second module leaves `output_truncate` unset so that it comes from `--collector.ssh.default-output-truncate`, passed through `prometheus_ssh_exporter_process_extra_arguments_custom`, and the shortened label is what says the role's process arguments reached the running process
- the container's environment and labels are read back off the running container, which is where the `env` and `labels` files the role rendered have to have arrived
- the version the running binary reports matches `prometheus_ssh_exporter_version`

Everything is offline: the only thing probed is a container on the network the role created.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
