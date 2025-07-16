necessary-extension
===

A collection of [kalico](https://github.com/KalicoCrew/kalico) extensions and macros.

## Installation

First open a shell on the computer which has klipper installed, and then execute the following commands.

Navigate into your home directory
```console
user@pi:~$ cd ~/
```

Clone the repository
```console
user@pi:~$ git clone https://github.com/Luro02/necessary-extension.git
```

Then open the `~/printer_data/config/moonraker.conf` with an editor like `nano` and add the following code to the end of it:
```console
user@pi:~$ nano ~/printer_data/config/moonraker.conf
```

```ini
[update_manager necessary-extension]
type: git_repo
channel: dev
path: ~/necessary-extension
origin: https://github.com/Luro02/necessary-extension.git
managed_services: klipper
primary_branch: master
install_script: install.sh
is_system_service: False
```

Please ensure that the following sections are present (or included) in your `printer.cfg`.
```ini
# Overwrite idle_timeout, so when the printer is paused for filament change or filament runout,
# only the extruder is powered down and not the heated bed (which would result in the print
# coming loose from the bed)
[idle_timeout]
gcode:
    {% if printer.pause_resume.is_paused %}
        { action_respond_info("Idle Timeout: Extruder powered down") }
        M104 S0   ; Set Hot-end to 0C (off)
    {% else %}
        { action_respond_info("Idle Timeout: Stepper and Heater powered down") }
        TURN_OFF_HEATERS
        M84
    {% endif %}
```

Depending on your setup, some of those sections might be set in other included configs like `mainsail.cfg`.

Then execute the `install.sh` script:
```console
user@pi:~$ cd ~/necessary-extension
user@pi:~/necessary-extension$ ./install.sh
```

## Available Extensions

### `parse_marlin_params`

This extension adds the command `_PARSE_MARLIN_PARAMS` that translates arguments like `P500` that are
common in marlin macros to `P=500` that is common in klipper. With this, it is not necessary to manually
parse the `rawparams`.

Some included macros use this extension, therefore it is strongly recommended to enable it in your configs,
by adding the following section:
```ini
[parse_marlin_params]
```

## Available Macros

### `M18`/`M84` override

The `M18`/`M84` macro disables the stepper motors. By passing one or more axis as an argument
it is possible to only disable the stepper motor for that axis.
Klipper does not respect the arguments and instead disables all motors.

This override fixes that.

**Warning**: This macro makes some assumptions regarding which stepper motors are available and
what they are named. Depending on your setup, this might not be desireable.
