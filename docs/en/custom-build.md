# Custom Build Guide

[Back to README](./README.md) | [中文](../zh/custom-build.md) | English

If your goal is to **quickly create your own image**, the easiest and most recommended option is **Custom Build** in GitHub Actions.

It now uses an explicit tuple model:

- `base_system`
- `include_docker`
- `output_formats`

So instead of choosing a legacy variant name, you directly describe the image you want.

---

## Quick start in 3 minutes

### Step 1: Open Actions

In your own fork of the repository:

- Click **Actions** in the top navigation bar
- Find **Custom Build** in the left sidebar
- Click **Run workflow**

---

### Step 2: Choose the base tuple

For a first run, start with:

- `base_system=debian`
- `include_docker=false`
- `output_formats=img`

A simple way to think about the common combinations:

- `debian + false`: most general-purpose, recommended for first-time users
- `debian + true`: Debian image with Docker included
- `alpine + false`: lighter image
- `alpine + true`: lighter image with Docker included

And for output formats:

- `img`: the core format used for testing and raw-disk import
- `vmdk`: useful when you specifically need VMDK
- `pve-ova`: useful for PVE import

If you just want your first successful build, use **`debian + false + img`**.

---

### Step 3: Fill in parameters as needed

#### Scenario A: You only want to change network settings

You can enter:

- `lan_server_ip=192.168.50.1`
- `lan_range_start=192.168.50.100`
- `lan_range_end=192.168.50.200`
- `lan_netmask=24`

#### Scenario B: You also want to change passwords

You can additionally enter:

- `root_password=Passw0rd!234`
- `api_username=admin`
- `api_password=Adm1n!234`

#### Other common input

- `landscape_version`
  - The Landscape version to build
  - If left blank, the repository default is used

Current precedence:

**direct inputs > secrets > defaults**

---

### Step 4: Run the workflow

After filling in the options, click:

- **Run workflow**

---

### Step 5: Download the build output

After the workflow finishes:

- Open that workflow run
- Scroll down to **Artifacts**
- Download the artifact you need

The output usually includes:

- the raw image `.img`
- build metadata `build-metadata.txt`
- the resolved configuration `effective-landscape_init.toml`
- and, if requested, `.vmdk` / `.ova`

---

## How to choose a tuple

### What should I pick for my first run?

Use:

- `base_system=debian`
- `include_docker=false`
- `output_formats=img`

### I want Docker

Set:

- `include_docker=true`

### I want a lighter image

Set:

- `base_system=alpine`

### I want to import into PVE

Set:

- `output_formats=img,pve-ova`

That gives you both:

- a raw `.img` for testing and fallback
- an `.ova` for import workflows

---

## What can you do after the build finishes?

After you have successfully run Custom Build once, you can also use:

- **Test Image**

This is useful for:

- re-running validation on an existing artifact
- running readiness or dataplane checks afterward
- testing again with different SSH or API credentials

The retest entry points are now:

- `run_id`
- `artifact_id`

rather than the older `artifact_suffix` naming.

---

## FAQ

### What should I choose for my first run?

Choose:

- `base_system=debian`
- `include_docker=false`
- `output_formats=img`

### Does `pve-ova` replace `.img`?

No.

It is recommended to keep `img` and add `pve-ova` when needed.

### Why does dataplane sometimes not run?

The rule is:

- `include_docker=false` → run dataplane
- `include_docker=true` → readiness only

That rule is based on capability, not on legacy variant names.

---

## One-line recommendation

If your goal is:

> **“I want to create my own image as quickly as possible.”**

Start with:

- `debian + no-docker + img`

Get your first image working first, then decide whether to add Docker, switch to Alpine, or request `vmdk` / `pve-ova`.
