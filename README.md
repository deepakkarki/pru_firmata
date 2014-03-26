PRU Firmata
===========

A remoteproc based VM firmware for the PRU. The VM acts as a server that interprets BotSpeak code from the client process (which will be running on linux). The client will communicate with the PRU via RPC's done through the virtio interface.

Here is how the components are interconnected :
![alt tag](https://raw.github.com/wiki/deepakkarki/pru_firmata/Botspeak.jpg)

More information on different components of the project, working of each component and high level design specification will be updated shortly. (below text is taken from pruduino mockup)

# design

This is rough pseudo-code to explain the design concept.

```C
int script_is_running = 0;
int missed_realtime = 0;

main() {
	while(;;) {
		test_for_flags();
		if(script_is_running) {
			preprocess_next_command();
			test_for_realtime();
		}
	}
}

test_for_flags() {
	int flags = read_flags();

	if(script_is_running && (flags & HAVE_SCRIPT_TIMER)) {
		execute_next_command();
	}
	if(flags & HAVE_TIMER_EVENT) {
		process_timer_event();
	}
	if(flags & HAVE_MESSAGE) {
		process_message();
	}
}

test_for_realtime() {
	int flags = read_flags();
	
	// We should have time to get back into the loop
	// before the timer has done off
	if(flags & HAVE_SCRIPT_TIMER) {
		missed_realtime = 1;
	}
}
```

This means that commands will only be executed upon timer events to keep the timing
consistent.  Between executing commands, messages can come in for immediate execution,
so less than 50% of the inner loop can be spent on executing a command to allow for
consistent timing. However, I can imagine that non-IO commands could be executed in a
more-than-one-per-cycle manner, but that'll probably come in a later implementation.

# script commands

* SETMODE pin, dir: Set pin direction
* SERVO ATTACH pin, min, max: Enable servo
* SERVO WRITE pin, value: Write to servo
* SERVO DETACH pin: Disable servo
* SET dst, value (=): dst = value
* GET src (g): returns src
* ADD dst, src (+): dst = dst + src
* SUB dst, src (-): dst = dst - src
* MUL dst, src (*): dst = dst * src
* DIV dst, src (/): dst = dst / src
* MOD dst, src (%): dst = dst % src
* AND dst, src (&): dst = dst & src
* OR dst, src (|): dst = dst | src
* BSL dst, src (<): dst = dst << src
* BSR dst, src (>): dst = dst >> src
* NOT dst (~): dst = ~dst
* GOTO addr (G): Go to addr
* IF x condition y, addr (I): On test true, go to addr
* UNLESS x condition y, addr (i): On test false, go to addr
* INT event, addr (V): On event, go to addr 
* DETACH event (v): Remove interrupt event
* RETURN (.): Return to previous execution point when interrupted
* WAIT ms (W): Delay for ms
* WAIT us (w): Delay for ms
* SYSTEM (!): TBD
* SCRIPT (s): Start script saving off code to run in an array
* ENDSCRIPT (E): End script going back to immediate interpretation
* LBL reg (L): Save address of point in script into register to pass to GOTO/IF/UNLESS/INT
* RUN (r): Begin executing script
* RUN&WAIT (R): Execute script and wait for completion before accepting new commands
* DEBUG (d): Return debug information during execution
* VER (v): Return interpreter version
* SPEED clocks (S): Set number of clocks between instruction execution

Arguments

* DIO[#]
* AI[#]
* PWM[#]
* TMR[#]

Example: noduino 'dw(13, 1)' becomes 'SET DIO[13],1'
