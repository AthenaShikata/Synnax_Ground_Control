# Synnax_Ground_Control
Disclaimer - This project began as a rewrite of software originally built for the SDSU Rocket Project's Synnax Ground Control system, aimed at making it more efficient, modular, and reliable. It was ultimately not adopted by the team, so I've forked it for personal use and as a portfolio piece. A small amount of code remains from the original (primarily peripheral configuration and thread initialization), though even those sections have been modified, and all surrounding logic has been rewritten.



Synnax is a data acquisition and control platform used for testing and operating hardware systems, such as rocket propulsion test stands. This program is a Python backend that lets the Synnax frontend GUI interface with National Instruments and LabJack devices, handling both data acquisition and relay/actuator control.



Startup Instructions

Open command line terminal or vs code terminal. Run the following command:
synnax start --listen=localhost:9090 --insecure --license-key=#########
Open Synnax desktop app and connect to the default cluster or the one specified in client
Load Synnax control panel (P&ID) and select channels for each switch, indicator, and graph
Set python Configuration Variables
Run python program
(Instead of using the command above, the synnax cluster can also be run as a docker container)
(Instead of connecting with the desktop app, you can also connect directly at the clusters web address (i.e. localhost:9090). Linux users must use this because there is no console for linux)



Stop Instructions

Press ctrl+c in the python terminal or the shutdown button in the synnax app
Close Synnax app
Type stop in the command line window and press enter before closing the terminal



Configuration Explanation 

Terminology: A sensor is any physical sensor conencted to the NI module for logging. Sensors have a type, sensor_type, that can be either 'PT' (pressure transducer), 'TC' (thermocouple), or 'LC' (load cell). Sensor scaling is the way be convert a raw voltage or current value for a sensor into units. The raw sensor valueis multiplied by the multiplier value and this is added to the offset value. Multiplier and offset values are optional, but if either is applied, the raw value will be separately logged in synnax and the log files. A switch is a switch in synnax that controls the state of a valve, lockout, or sequence. A switch can only be "flipped" in Synnax while the shift key is held to prevent accidental actuations. A condition is a switch whose value must be true (conditions_true) or false (conditions_false) for another switch to be in an active state. State refers to the value of the switch (synnax is 0/1, but can state can also be T/F, On/Off, etc) and in the initial arrays is used to specify the initial state on startup. A valve is a channel on an ssr relay controlling a solonoid actuated valve (or some other electrical cicuit in special cases). A lockout is a switch who is only a condition. A sequence is a logic based series of automated events run as either a thread or function with the goal to automatically actuate valves or switches or to perform some other action automatically. Channel refers to either the channel on the relevant NI module or the synnax channels for a switch (context dependent). Module/module number refers to the slot number (with adjustment for starting at 0 or 1 in "NI Counting" below) of an NI module in the cDAQ. Nominal is the plain text default state of a valve (Closed/Open or On/Off/something else in special cases). Plain name is the plain text name of a switch (should include the type of switch ie valve, lockout, or sequence). In sequence_array, function_name is the string identifier of the python function associated with that sequence. The function_name_interpreter dictionary is used to convert the function_name into the actual python function. In sequence_array, thread is a boolean that if True, will have the function run as a thread, and if False, will run the function as a normal function. 

NI Counting: We always start counting at 0. NI starts counting at both 0 and 1 depending on the device. The NI chassis start counting slot numbers at 1, so you will need to subtract 1 to determine the module number. NI-9485 SSR Modules start counting their channels at 0, so you won't need to subtract 1, but other modules do start at 1 so make sure to check. If NI starts at 0, start at 0. If NI starts at 1, you will have to subtract 1 from their value to get the value needed by this program. Example: NI chassis slot 4 becomes module 3 because it starts at module 1, but channel 2 stays 2 because it starts at channel 0.

Limitations
Only one NI cDAQ chassis supported. Nothing is configured to distinguish between different cDAQ chassis. To add multi-chassis support, you will need to add a chassis parameter to valve_array, modify how data is handled in NI, and modify how writing to NI is handled. Not difficult, but definetely, annoying
Sensor type of each NI module is hard coded (ie 9205 is PT only)
All NI device names must be in the format cDAQ{cDAQ number}Mod{Module Number} for modules and cDAQ{cDAQ number} for chassis
If running on Linux, follow the instructions in this thread if having issues creating simulated NI hardware (Maybe real hardware too idk) https://forums.ni.com/t5/Multifunction-DAQ/Simulated-devices-on-Linux-stuck-in-quot-Initializing-quot/m-p/4460845
If running on Linux, the shift detect functionality will not work because this program uses the Keyboard module. To use shift detect on linux, modify the program to work with the pynput module instead (a working example of this is in the ValkyrieSpaceSystems/MIDGARD_Ground_Control repo) or use the bypass_shift variable.
Only the LabJack T7 is supported (this is because we only have the T7, but this can be easily changed in the handle variable and there shouldn't be any programatic reasons to prevent other device from working)
The only supported LabJack channels are FIO0-7 for valves and AIN0-13 for sensors (inclusive)