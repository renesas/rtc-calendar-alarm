# Real-Time Clock (RTC) Software Building Block

## Overview

This Software Building Block (SBB) demonstrates typical use of the Renesas RA RTC HAL module APIs. It initializes the RTC and provides operations to set calendar date and time, configure a calendar alarm, enable a periodic interrupt, and read the current RTC calendar time. When the periodic interrupt is enabled, the on-board LED toggles every second. When the calendar alarm occurs, the reference application turns the LED on for two seconds.

This documentation covers only these boards:

- FPB-RA0L1
- FPB-RA2E3
- FPB-RA4E2
- FPB-RA6E2
- FPB-RA8E1

## 1. Use Case Demonstrated

The SBB demonstrates using the MCU real-time clock for calendar timekeeping, calendar alarms, and periodic RTC interrupts.

The reference application provides four terminal operations:

1. Set the RTC calendar date and time.
2. Set the RTC calendar alarm.
3. Set the RTC periodic interrupt rate and start the periodic interrupt.
4. Read the current RTC calendar date and time.

The application accepts calendar date and time in `DD:MM:YYYY HH:MM:SS` format. For the `RTC` peripheral, the alarm input is a seconds value in `SS` format. For `RTC_C`, the alarm input is hour and minute in `HH:MM` format.

The RTC supports a 100-year calendar from 2000 through 2099. The calendar date and time must be set before the other demonstrated operations.

## 2. SBB Core Code

The reusable RTC functionality is implemented in `rtc_ep.c` with its public interface in `rtc_ep.h`.

### RTC initialization

`rtc_init()` opens the FSP-generated RTC instance using `g_rtc_ctrl` and `g_rtc_cfg`. The source abstracts the particular FSP RTC implementation through macros so that the calls resolve to either the `r_rtc_c` or `r_rtc` driver according to the peripheral available on the MCU.

### Calendar date and time

`set_rtc_calendar_time()` converts the supplied `DD:MM:YYYY HH:MM:SS` text into an `rtc_time_t` structure, sets the RTC calendar time, reads the value back, and converts the month and year fields to user-readable values.

For devices using `RTC_C`, the implementation also calculates the day of the week before setting the calendar. The code validates the applicable month, day, and year values as part of that calculation.

`get_rtc_calendar_time()` reads the current RTC calendar time and adjusts the returned month and year values for display.

### Calendar alarm

`set_rtc_calendar_alarm()` configures the demonstrated recursive alarm. The alarm can only be configured after the calendar date and time has been successfully set.

- For `RTC_C`, the application accepts `HH:MM` and configures the hour and minute alarm values.
- For `RTC`, the application accepts `SS` and enables the seconds match field.

### Periodic interrupt

`set_rtc_periodic_rate()` configures the RTC periodic interrupt for `RTC_PERIODIC_IRQ_SELECT_1_SECOND`.

The `rtc_callback()` callback distinguishes between alarm and periodic RTC events and sets `g_alarm_irq_flag` or `g_periodic_irq_flag`. The reference application processes those flags outside the callback.

### LED support

`led_level_toggle()` reads and toggles the first BSP LED. `led_level_set()` explicitly sets its output level. These functions use the FSP I/O port interface and BSP LED information.

### Deinitialization

`rtc_deinit()` closes an opened RTC instance through the appropriate RTC driver abstraction.

## 3. Reusable SBB Code vs. Demo/Reference Application Code

### Reusable SBB code

The RTC implementation and interface are contained in:

- `rtc_ep.c`
- `rtc_ep.h`

These files contain RTC initialization, calendar-time handling, alarm configuration, periodic interrupt configuration, RTC callback processing, LED helper functions, and RTC deinitialization.

The implementation is coupled to FSP-generated configuration and BSP resources through `hal_data.h`, `g_rtc_ctrl`, `g_rtc_cfg`, `g_ioport_ctrl`, and `g_bsp_leds`.

### Demo/reference application code

The terminal-oriented reference application is contained in:

- `rtc_example_app.c`
- `rtc_example_app.h`

`rtc_example_app_run()` is the single reference-application entry point. It initializes the UART console, obtains and prints the FSP version, initializes the RTC, presents the menu, dispatches user selections, and processes alarm and periodic-interrupt flags.

The following are reference-application concerns rather than RTC core functionality:

- Terminal menu and prompts
- UART-console initialization and polling
- Parsing menu selections
- Reading text with `fgets()`
- Printing RTC values and status messages
- Turning the LED on for two seconds after an alarm
- Error reporting and breakpoint-based error handling

## 4. Public Functions to Review First

Start with the following functions when understanding or integrating the SBB:

| Function | Purpose |
|---|---|
| `rtc_init()` | Opens the configured RTC instance. |
| `set_rtc_calendar_time()` | Converts, sets, and reads back the calendar date and time. |
| `get_rtc_calendar_time()` | Reads the current RTC calendar date and time. |
| `set_rtc_calendar_alarm()` | Configures the demonstrated RTC calendar alarm. |
| `set_rtc_periodic_rate()` | Configures the one-second periodic RTC interrupt. |
| `rtc_callback()` | Records alarm and periodic RTC events for application processing. |
| `rtc_deinit()` | Closes the RTC instance. |
| `rtc_example_app_run()` | Shows how the reference application assembles and exercises the SBB. |

The LED helper functions `led_level_toggle()` and `led_level_set()` show how the reference application provides visual indication of RTC events.

## 5. Reference Board Configuration Assumptions

### Supported boards covered by this document

| Board | Board-specific setup documented by the supplied README |
|---|---|
| FPB-RA0L1 | Place jumper `J9` on pins `1-2` to use the on-board debug functionality. |
| FPB-RA2E3 | Close `E25`, `E28`, `E46`, and `E47` to use J-Link OB VCOM for the UART serial terminal. |
| FPB-RA4E2 | No additional board-specific connection is stated in the supplied README. |
| FPB-RA6E2 | No additional board-specific connection is stated in the supplied README. |
| FPB-RA8E1 | Open jumpers `J6` and `J9` to use the on-board debug functionality. |

A Renesas RA board and a USB cable appropriate for that board are required. The USB debug port is connected to the PC for programming and debugging.

### Serial terminal configuration

The reference application uses a serial terminal with the settings documented in the supplied README:

- Port: J-Link On-Board provided port
- Speed: `115200`
- Data: `8 bit`
- Parity: `none`
- Stop bits: `1 bit`
- Flow control: `none`
- Receive new-line: `CR`
- Transmit new-line: `CR`
- Local echo: enabled

### Input assumptions

The supplied README states that:

- User-entered data is expected not to exceed 15 bytes.
- Calendar input uses `DD:MM:YYYY HH:MM:SS`.
- The RTC calendar covers years 2000 through 2099.
- Calendar date and time must be set before the other demonstrated operations.
- `RTC` alarm input uses `SS`.
- `RTC_C` alarm input uses `HH:MM`.
- Operation is not guaranteed for unsupported input types or incorrect terminal input formats.
- For `RTC_C`, alarm and periodic interrupts are retained after pressing the reset pin.

## 6. What Must Be Changed When Porting

When moving this SBB to a different board or project, review the following integration points.

### RTC peripheral and FSP configuration

The source selects between `r_rtc_c` and `r_rtc` using `BSP_PERIPHERAL_RTC_C_PRESENT` and `BSP_PERIPHERAL_RTC_PRESENT`. The generated RTC instance is accessed through `g_rtc_ctrl` and `g_rtc_cfg`.

The target project must therefore provide the applicable RTC peripheral and FSP-generated RTC configuration expected by the SBB.

### Board and LED resources

The example uses the first BSP LED through `g_bsp_leds.p_leds[0]` and controls it through `g_ioport_ctrl`. If the target board uses a different indication mechanism, adapt the LED-related reference behavior accordingly.

### RTC timing and alarm behavior

The demonstrated periodic interrupt is explicitly configured for one second through `RTC_PERIODIC_IRQ_SELECT_1_SECOND`. Alarm behavior differs between the two RTC peripheral implementations: `RTC` uses a seconds match in this example, while `RTC_C` uses hour and minute values.

Review these assumptions if the product requires different periodic timing or alarm matching.

### Calendar representation

The SBB accepts date/time text in `DD:MM:YYYY HH:MM:SS` format. Internally, the month and year are adjusted to the representation expected by `rtc_time_t`. The `RTC_C` path calculates the weekday, and the implementation is designed for the RTC calendar range from 2000 through 2099.

### Clock configuration

The supplied SBB source uses the FSP RTC configuration but does not specify clock-tree configuration values. Verify the RTC clock configuration in the target FSP project when integrating the SBB.

### Memory and task priorities

The supplied application does not specify an RTOS task-priority model or explicit heap/stack requirements. The reference application executes an infinite polling loop while RTC events are reported to `rtc_callback()` and communicated through volatile flags.

### Console integration

The reference application includes `console/uart_console.h` and uses the UART console for its user interface. If the target product does not use this console interface, replace or remove the terminal-oriented reference-application layer while retaining the required RTC functionality.

## 7. Dependencies

### FSP and HAL

The SBB uses FSP-generated RTC and I/O resources through `hal_data.h`.

Depending on the MCU RTC peripheral, the abstraction in `rtc_ep.h` maps operations to either:

- `R_RTC_C_Open()` / `R_RTC_Open()`
- `R_RTC_C_CalendarTimeSet()` / `R_RTC_CalendarTimeSet()`
- `R_RTC_C_CalendarTimeGet()` / `R_RTC_CalendarTimeGet()`
- `R_RTC_C_CalendarAlarmSet()` / `R_RTC_CalendarAlarmSet()`
- `R_RTC_C_CalendarAlarmGet()` / `R_RTC_CalendarAlarmGet()`
- `R_RTC_C_PeriodicIrqRateSet()` / `R_RTC_PeriodicIrqRateSet()`
- `R_RTC_C_Close()` / `R_RTC_Close()`

The LED helpers additionally use:

- `R_IOPORT_PinRead()`
- `R_IOPORT_PinWrite()`
- `g_ioport_ctrl`
- `g_bsp_leds`

### Reference-application dependencies

The reference application additionally uses:

- `console/uart_console.h`
- `uart_console_init()`
- `uart_console_has_char()`
- `uart_console_flush()`
- `printf()`
- `fgets()`
- `atoi()`
- `R_FSP_VersionGet()`
- `R_BSP_SoftwareDelay()`

### Hardware resources

The demonstrated application uses:

- MCU RTC or RTC_C peripheral
- RTC alarm interrupt
- RTC periodic interrupt
- First BSP LED and its I/O port resource
- UART/serial console for the reference user interface
- Board debug and J-Link On-Board resources as applicable to the selected reference board

## 8. Demonstrated Behavior and Performance

The supplied project demonstrates the following observable behavior:

- The user can set the RTC calendar date and time.
- The application can read and display the current RTC calendar date and time.
- A calendar alarm can be configured after calendar date and time have been set.
- For `RTC`, the demonstrated recursive alarm matches a user-selected seconds value.
- For `RTC_C`, the demonstrated recursive alarm uses a user-selected hour and minute.
- When an alarm interrupt occurs, the application reports the current date/time and turns the on-board LED on for two seconds before turning it off.
- The RTC periodic interrupt is configured for a one-second rate.
- Each processed periodic interrupt causes the reference application to toggle the on-board LED.
- For `RTC_C`, the supplied README states that alarm and periodic interrupts are retained after pressing the reset pin.

No other measured performance figures are supplied in these files.

## 9. File Modification Guidance

### SBB implementation files

- `rtc_ep.c`
- `rtc_ep.h`

These contain the RTC functionality and its public interface. Product-specific modifications to these files create a local variant of the SBB that must be considered when adopting a later SBB version.

### Demo/reference application files

- `rtc_example_app.c`
- `rtc_example_app.h`

These implement the terminal user interface and demonstration workflow. They show one way to exercise the SBB and can be adapted or replaced when integrating the RTC functionality into a product application.

### FSP-generated configuration

The SBB depends on generated declarations and instances supplied through `hal_data.h`, including `g_rtc_ctrl`, `g_rtc_cfg`, and `g_ioport_ctrl`. The supplied files do not classify the complete set of generated project files. Configuration belonging to the FSP project should be maintained through the project's FSP configuration flow rather than treated as SBB application source.

## 10. Adopting a Newer SBB Release After Local Modifications

The supplied files do not define an automatic source-upgrade or merge mechanism for locally modified SBB code.

To keep future SBB updates manageable, keep product-specific application behavior outside `rtc_ep.c` and `rtc_ep.h` where practical. The provided separation between the RTC implementation and `rtc_example_app.c` gives the product application a natural integration boundary.

If the supplied SBB source itself is modified, record those changes. When adopting a newer SBB release, compare the updated RTC implementation and its FSP configuration assumptions with the locally used version, carry forward only the product-specific changes that remain necessary, and revalidate the required RTC calendar, alarm, periodic-interrupt, and board-indication behavior.

## Reference Application Operation

1. Prepare the selected FPB board according to the applicable board-specific requirements above.
2. Connect the board USB debug port to the PC for programming and debugging.
3. Open the J-Link On-Board serial port and configure the terminal for `115200`, 8 data bits, no parity, 1 stop bit, no flow control, CR receive/transmit newline, and local echo.
4. Run the application.
5. Select option `1` and enter the RTC date/time in `DD:MM:YYYY HH:MM:SS` format.
6. Use option `2` to configure a calendar alarm. Enter `SS` for `RTC` or `HH:MM` for `RTC_C`.
7. Use option `3` to configure the one-second periodic RTC interrupt and observe the LED toggling.
8. Use option `4` to read and display the current RTC date and time.

## Source File Map

| File | Role |
|---|---|
| `rtc_ep.c` | RTC initialization, calendar processing, alarm configuration, periodic IRQ configuration, RTC callback, LED helpers, and RTC deinitialization. |
| `rtc_ep.h` | SBB constants, RTC/RTC_C abstraction, shared flags, and public RTC function declarations. |
| `rtc_example_app.c` | Demo/reference application orchestration, UART menu handling, RTC-event processing, display behavior, and error handling. |
| `rtc_example_app.h` | Reference-application interface, terminal banner/menu strings, and `rtc_example_app_run()` declaration. |
