# Changelog

## [1.0.1] - 2026-11-03

### Changed

- Align the canonical GitHub source and installer `RAW_BASE` with
  `Tuxbull-Open-Source/ansible.installer`.
- Prevent root-sourced `ANSIBLE_LOCAL_TEMP` and `ANSIBLE_LOG_PATH` values from
  leaking into installer commands executed as a selected non-root system user.
- Add a version-rendered default `requirements.txt` baseline and optional
  `ANSIBLE_REQUIREMENTS_FILE` override; keep `ANSIBLE_PIP_PACKAGES` as an
  explicit full-baseline override.
- Add `collections/requirements.yml` for optional common controller collections.
- Ensure a generated userspace profile overrides inherited system-profile
  runtime variables, including paths, virtualenv, configuration, temporary, and
  log settings.
- Update static checks to protect the transferred source location and
  maintainer guidance path.

### Verified

- Fresh Rocky Linux 10 system-scope and user-scope installer validation using
  repository-local payloads, including same-input reruns and runtime checks.

All notable changes to `ansible.installer` are documented here.

This project follows semantic versioning where practical.

## [1.0.0] - 2026-07-07

### Added

- Initial stable release of the uv-based local Ansible control-node installer for Rocky, RHEL, AlmaLinux and compatible systems.
- Shared `SCOPE=system` layout under `/opt/ansible` with:
  - versioned runtimes below `/opt/ansible/apps/<python>_<ansible>`
  - `/opt/ansible/current` active-runtime symlink
  - `/etc/profile.d/ansible.sh` shell integration
  - `/etc/ansible -> /opt/ansible` compatibility symlink when safe
  - `/usr/local/bin/ansible-local-switch`
  - `/usr/local/bin/adoc`
  - argcomplete integration in `/etc/bash_completion.d/ansible`
- Isolated `SCOPE=user` layout below `${ANSIBLE_HOME}` with installer-specific files below `${ANSIBLE_HOME}/apps` and no writes to `/etc`, `/usr/local/bin`, or `/opt`.
- Idempotent userspace shell activation block in the target user's `~/.bashrc`.
- Foreign Ansible installation detection for RPM, pip, legacy profile scripts, unexpected `ansible` binaries and conflicting `/etc/ansible` paths.
- Safe ansible-uv reruns that reuse existing runtimes and preserve existing operational config such as `${ANSIBLE_HOME}/ansible.cfg`.
- Shared system-scope uv-managed Python location via `UV_PYTHON_INSTALL_DIR=/opt/ansible/apps/python` to avoid root-private Python symlink targets.
- Runtime switching through `ansible-local-switch`, including session-only switching via the profile function and persistent default switching with `--permanent`.
- Vendored `adoc` helper for quick `ansible-doc` lookup from the active runtime.
- Default controller-side Python package set through `ANSIBLE_PIP_PACKAGES`:
  - `ansible==${ANSIBLE_VERSION}`
  - `argcomplete`
  - `passlib`
  - `jmespath`
  - `netaddr`
  - `dnspython`
- `ansible-pip-install` interactive alias for installing additional packages into the active uv-managed Ansible runtime.
- `PYTHONDONTWRITEBYTECODE=1` profile/runtime handling plus `__pycache__` cleanup during reruns to preserve the managed permission model.
- Static test coverage for installer paths, docs, payload ownership intent, user/system scope behavior, package defaults and profile integration.
- `AGENTS.md` maintainer guide for future agent and human maintenance work.

### Changed

- Clarified that `ANSIBLE_VERSION` means the Ansible community PyPI package version, while `ANSIBLE_CORE_VERSION` is only a legacy alias input.
- Documented how to install additional Python packages into the active runtime and how to override `ANSIBLE_PIP_PACKAGES` at install time.
- Documented userspace and unattended install patterns with explicit `sudo -n` guards.
- Hardened system-scope uv behavior so later user installs can reuse a traversable system uv binary.
- Hardened permission normalization for the managed `/opt/ansible` tree.

### Security

- System-scope Ansible payloads and managed runtime files are intended to be accessible only to root and the selected Ansible group.
- User-scope installs intentionally avoid system integration paths.
- The installer stops on foreign Ansible installations instead of mixing install methods.
- Public documentation uses placeholder paths and public URLs; no lab inventory or credentials are required.

### Validation

Validated for the v1.0.0 release candidate with:

```bash
pytest tests/test_installer_static.py -q
sh -n ansible/ansible_setup.sh
bash -n ansible/files/etc/profile.d/ansible.sh
bash -n ansible/files/usr/local/bin/ansible-local-switch
bash -n ansible/files/usr/local/bin/adoc
git diff --check
```

Also validated on a fresh Rocky 10.1 lab VM using local repository payloads with `RAW_BASE=file:///tmp/ansible.installer`, including install, rerun, package imports, `ansible localhost -m ping`, `password_hash`, `ansible-local-switch --list`, `adoc copy`, `ansible-pip-install passlib`, and a clean `/opt/ansible` permission scan.
