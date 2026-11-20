# pkg-tc9564

Debian packaging for the Qualcomm TC9564 (QPS615) PCIe Ethernet bridge,
built from [TC9564_Host_Driver](https://github.com/qualcomm-linux/TC9564_Host_Driver).
The `tc9564` source package produces:

- **tc9564-dkms**: sources for the `tc956x_pcie_eth` kernel module, built
  by DKMS for installed kernels (arm64, `CONFIG_ARCH_QCOM`). It uses the
  same physical-function and Qualcomm platform configuration as Yocto.
- **tc9564-firmware**: the TC956X PCIe bridge firmware
  (`/usr/lib/firmware/TC956X_Firmware_PCIeBridge.bin`). The driver loads
  it into the bridge's M3 at probe and again on every resume. The bridge
  is reset across suspend, so `tc9564-dkms` depends on this package.

The installed `README.Debian` covers RB3 Gen2 setup. That includes
activating the staging device-tree overlay and post-install checks.

## Branches

- **qli-ci**: the default branch. It holds this documentation and the
  workflow callers, which run the shared package CI from
  [qli-ci](https://github.com/qualcomm-linux/qli-ci).
- **qcom/ubuntu/resolute**: the Ubuntu 26.04 (Resolute Raccoon) packaging
  branch. Its history is the upstream driver history, then the `firmware/`
  commit, the PR hook workflow and `debian/`.
- **upstream/latest**: the upstream driver history (`qcom` branch of
  TC9564_Host_Driver) plus one commit that adds `firmware/`. Each packaged
  upstream version is tagged `upstream/<version>`, and `gbp` builds the
  orig tarballs from that tag.
- **qli/debian/latest**: still the pkg-template placeholder. It is not
  built for this package yet.

## Firmware

`firmware/Bin/` holds the unmodified published binary and its licence
from [TC956X/TC9564_Firmware](https://github.com/TC956X/TC9564_Firmware)
tag `V6.0.0` (commit `f9b0e1bc0f7c3dfc74ad1a46a87efa56885b9288`):

| File | sha256 |
|------|--------|
| `TC956X_Firmware_PCIeBridge.bin` | `c2d38397da62452ae7defffbf182ed22e186b9e4ffb927f8e2d7cce8957a0230` |
| `LICENSE` (MIT) | `0947ae5d3cffd521ec73941910d29296e0f865564d0a04cd4f01a3dc60f6f148` |

`debian/gbp.conf` declares it as the `firmware` orig component, so it
ships as `tc9564_<version>.orig-firmware.tar.gz` next to the driver
tarball. It is committed to git rather than downloaded at build time,
because package builds run without network access. The firmware is not
expected to change. If it does, replace both files in a new
`upstream/latest` commit, record the new tag, commit and sha256 values
here, and tag a new upstream version.

## Building locally

On Ubuntu 26.04 with `git-buildpackage`, `debhelper`, `dh-dkms` and
`lintian` installed, from a checkout of `qcom/ubuntu/resolute` with the
`upstream/*` tags fetched:

```sh
gbp buildpackage --git-no-pristine-tar --git-builder='dpkg-buildpackage --no-sign --host-arch=arm64'
lintian -EviL +pedantic ../tc9564_*.changes
```

Nothing is compiled at package build time. DKMS compiles the module on
the target against its kernel headers.

## Updating the driver

The `pkg-promote` workflow cannot be used for this repository. It
fast-forwards `upstream/latest` to an upstream tag, which fails because of
the `firmware/` commit. Upstream also packages untagged `qcom` snapshots,
and its release tags (`V_06-00-03`) do not map to a Debian version.

Upstream versions are `<last upstream release>+git<short commit>`, for
example `6.0.3+git2cbd7b9`. To package a new upstream commit:

```sh
git fetch https://github.com/qualcomm-linux/TC9564_Host_Driver.git qcom
git checkout upstream/latest
git merge --signoff <commit>        # firmware/ is carried along
git tag -a upstream/<version> -m "Upstream version <version>"
git checkout -b <my-branch-name> qcom/ubuntu/resolute
git merge --signoff upstream/<version>
dch -v <version>-0qli1~ "New upstream snapshot <commit>."
```

Update `debian/copyright` if the copyright holders changed, and update
the commit recorded in its `Comment:` field. Then build and run Lintian
as above. Push `upstream/latest` and the new tag, and open a PR against
`qcom/ubuntu/resolute`.

## Installing

Install the firmware and DKMS packages together:

```sh
sudo apt install ./tc9564-firmware_*_all.deb ./tc9564-dkms_*_arm64.deb
dkms status
```

If the earlier `qps615-dkms` and `qps615-firmware` test packages are
installed, purge them first. They ship the same files.

## Getting in Contact

* [Report an Issue on GitHub](../../issues)
* [E-mail us](mailto:Maintainers.pkg-tc9564@qualcomm.com) for general questions

## License

The packaging in `debian/` is licensed under the
[BSD-3-Clause-Clear License](https://spdx.org/licenses/BSD-3-Clause-Clear.html).
The driver is licensed under GPL-2.0 and the firmware under MIT. The
packaging branch's `debian/copyright` lists the licence of each file.
The documentation and workflow files on this branch are licensed under
the BSD-3-Clause License; see [LICENSE.txt](LICENSE.txt).
