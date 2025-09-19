# fpgad-provider-template

This repo provides example snap templates to use as inspiration for writing an FPGAd provider snap.
Two different approaches are provided since there are options. In this case there are 3 differences described
in the following table.

| Problem                 | Approach 1: load-git-dtbo-manually                                                                                       | Approach 2: load-local-bitstream-automatically                                                                 |
|-------------------------|--------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| Firmware source type    | packs firmware files from a git repository into the snap                                                                 | packs firmware files from a local project subdirectory into the snap                                           |
| Firmware load mechanism | uses a .dtbo device tree overlay file to trigger the firmware load to the FPGA device, and changes device tree structure | Directly writes the bitstream path to the FPGA manager which then loads to the device - no device tree changes |
| How to run              | The loading of the dtbo must be initiated manually by running <provider-snap>.<provider-app> in cmdline                  | The application is automatically run once during boot - requires connection hooks                              |



# anatomy of snapcraft.yaml files

If running on target device:

```shell
snapcraft
sudo snap install ..._arm64.snap
sudo snap connect <provider-snap>:fpgad-dbus fpgad:daemon-dbus
```

NB: the `fpgad:daemon-dbus` is external to this repo so may be subject to change.
Check [fpgad's snapcraft.yaml](https://github.com/canonical/fpgad/blob/main/snap/snapcraft.yaml) for changes if this
command fails.

## snapcraft.yaml explained

The `plugs` entry here allows the connection to be made between this snap and the fpgad daemon

```yaml
plugs:
  fgpad-dbus:
    interface: dbus
    bus: system
    name: com.canonical.fpgad
```

but it must also be added to the application:

```yaml
apps:
  <provider-app>:
    command: bin/<provider-snap>
    plugs:
      - fpgad-dbus
```

The parts section describes how to form the snap package

```yaml

parts:
  version:
    plugin: nil
    source: .
    build-snaps:
      - jq
    override-pull: |
      craftctl default
      cargo_version=$(cargo metadata --no-deps --format-version 1 | jq -r .packages[0].version)
      craftctl set version="$cargo_version+git$(date +'%Y%m%d').$(git describe --always --exclude '*')"
  <provider-app>:
    plugin: <plugin> # see documentation.ubuntu.com/snapcraft/stable/how-to/integrations/ for information on building inside a snap
    source: <relative/path/to/source>
    # ... any other build related settings
  bitstream-data:
    plugin: dump
    ...
```

Here `version` just runs a simple script to generate a unique version string, `<provider-app>` part defines how to build
the rust package which creates the `bin/k26-default-bitstreams` used in the app section and `bitstream-data` is a "dump"
of local or remote files into the snap data. Unfortunately, there is no good documentation about the dump plugin at time
of writing. Hopefully the two provided examples provide enough of an example of how it can be used.

[//]: # (TODO: update the above when available.)

# Licenses

The source code here is distributed under the GPL-3.0-only licence provided in the repository root's `LICENSE` file.
