# Setup
Setup for the tp-link plug is pretty straight forward. You need to drop some configuration into your moonraker.cfg file, your printer.cfg file, and your macros.cfg file if you have one.

# moonraker.cfg
Add the below to your moonraker.cfg file. Make sure to change the IP address to the IP address of your tp-link.
```
##################################################
#     Power
##################################################
[power printer_plug]
type: tplink_smartplug
address: 192.168.10.132
#   The type of device.  Can be either gpio, klipper_device, rf,
#   tplink_smartplug, tasmota, shelly, homeseer, homeassistant, loxonev1,
#   smartthings, mqtt or hue.
#   This parameter must be provided.
initial_state: on
#    The state the power device should be initialized to.  May be on or
#    off.  When this option is not specifed no initial state will be set.
off_when_shutdown: true
#   If set to True the device will be powered off when Klipper enters
#   the "shutdown" state.  This option applies to all device types.
#   The default is False.
off_when_shutdown_delay: 0
#   If "off_when_shutdown" is set, this option specifies the amount of time
#   (in seconds) to wait before turning the device off. Default is 0 seconds.
on_when_job_queued: true
#   If set to True the device will power on if a job is queued while the
#   device is off.  This allows for an automated "upload, power on, and
#   print" approach directly from the slicer, see the configuration example
#   below for details. The default is False.
locked_while_printing: False
#   If True, locks the device so that the power cannot be changed while the
#   printer is printing. This is useful to avert an accidental shutdown to
#   the printer's power.  The default is False.
restart_klipper_when_powered: True
#   If set to True, Moonraker will schedule a "FIRMWARE_RESTART" to command
#   after the device has been powered on. If it isn't possible to immediately
#   schedule a firmware restart (ie: Klippy is disconnected), the restart
#   will be postponed until Klippy reconnects and reports that startup is
#   complete.  Prior to scheduling the restart command the power device will
#   always check Klippy's state.  If Klippy reports that it is "ready", the
#   FIRMWARE_RESTART will be aborted as unnecessary.
#   The default is False.
restart_delay: 1.
#   If "restart_klipper_when_powered" is set, this option specifies the amount
#   of time (in seconds) to delay the restart.  Default is 1 second.
bound_services:
#  klipper
#  klipper-mcu
#   A newline separated list of services that are "bound" to the state of this
#   device.  When the device is powered on all bound services will be started.
#   When the device is powered off all bound services are stopped.
#
#   The items in this list are limited to those specified in the allow list,
#   see the [machine] configuration documentation for details.  Additionally,
#   the Moonraker service can not be bound to a power device.  Note that
#   service names are case sensitive.
#
#   When the "initial_state" option is explcitly configured bound services
#   will be synced with the current state.  For example, if the initial_state
#   is "off", all bound services will be stopped after device initialization.
#
#   The default is no services are bound to the device.
```

# printer.cfg
Add the below to your printer.cfg file. If you have a macros.cfg file, you can move the macros in there. Make sure that the idle_timeout stays in the printer.cfg file.
```
##################################################
#     Idle Timeout
##################################################
[idle_timeout]
# Description: This macro is called when the printer goes into an idle state and calls the previous delayed_gcode. It updates the delay to the duration defined below.
# After 5 minutes (300seconds) of being idle, the gcode below will be called. It tells the relay to shut off after 10 minutes (600seconds).
gcode:
  M118 Printer has been idle for 5 minutes. Printer will shutdown in 10 minutes!
  M84
  TURN_OFF_HEATERS
  UPDATE_DELAYED_GCODE ID=delayed_printer_off DURATION=600 ; This is how long the relay will switch after the printer has gone idle
timeout: 300 ; This is how long it will take for the printer to go idle
# i.e. total time to shutoff is DURATION+timeout

# Comments:
# "idle_timeout" is a module that tracks the "state" of the printer. The "state" can be "Idle", "Printing", or "Ready"
# "M84" turns off the stepper motors
# "TURN_OFF_HEATERS" turns of the heaters
# "UPDATE_DELAYED_GCODE"
#   UPDATE_DELAYED_GCODE [ID=<name>] [DURATION=<seconds>]:
#   Updates the delay duration for the identified [delayed_gcode] and starts the timer for gcode execution.
#   A value of 0 will cancel a pending delayed gcode from executing.
```

# macros.cfg
Add the below to your macros.cfg file. If you don't have one, just add them to the printer.cfg but make sure that they are before the Idle Timeout section.
```
##################################################
#     Power Macros
##################################################
# Description: This section has a macro that calls moonraker to toggle our relay, a macro for delayed gcode, and a macro to call the delayed gcode when our printer enters the state "idle"

[gcode_macro POWER_OFF_PRINTER]
# Description: This macro asks moonraker to toggle our "relay" state to "off"
gcode:
  {action_call_remote_method(
    "set_device_power", device="printer_plug", state="off"
  )}
# Comments:
# action_call_remote_method is used to call a method registered by a remote client (moonraker)
# "set_device_power" is the method we want to call from the moonraker.cfg
# "device" is the name of the deivce defined in moonraker.cfg
# "state" is the state of the relay, on, off, or toggle

  
[delayed_gcode delayed_printer_off]
# Description: This macro calls the "POWER_OFF_PRINTER" macro once the "printer" object is in state "idle"
initial_duration: 0.
gcode:
  {% if printer.idle_timeout.state == "Idle" %}
    POWER_OFF_PRINTER
  {% endif %}
# Comments:
# "delayed_gcode" is a way to execute gcode after a set delay
# "initial_duration"
#   The duration of the initial delay (in seconds). If set to a
#   non-zero value the delayed_gcode will execute the specified number
#   of seconds after the printer enters the "ready" state. This can be
#   useful for initialization procedures or a repeating delayed_gcode.
#   If set to 0 the delayed_gcode will not execute on startup.
#   Default is 0.
# "POWER_OFF_PRINTER" calls the "POWER_OFF_PRINTER" macro defined above
```
