# PicoDMX
Probable one the simplest implementations of DMX on a Raspberry Pi Pico.


```
from machine import Pin, SPI
import array
import rp2
import time
import os

# We'll use the onboard LED indicate each DMX frame.
led = Pin(25, Pin.OUT)

@rp2.asm_pio(out_init=rp2.PIO.OUT_HIGH, set_init=rp2.PIO.OUT_HIGH, out_shiftdir=rp2.PIO.SHIFT_RIGHT, autopull=True, pull_thresh=8)

def w():
    label("stall")
    jmp(not_osre, "proceed") # STALL UNTIL THERE ARE SOME INTENSITIES IN THE FIFO
    jmp("stall")
    label("proceed")
    set(pins, 0)	[24] # BREAK - LOW FOR 100uS
    set(pins, 1)	[2] # MARK AFTER BREAK - HIGH FOR 12us    
    
    set(pins, 0)	# START BIT FOR SLOT 0
    set(pins, 0)	[7] # START CODE - USUALLY 0x00, CHECK DOCS
    set(pins, 1)	[1]	# STOP BITS    
    
    label("slot")
    set(pins, 0)	# START BIT FOR SLOTS 1-512
    out(pins, 1)
    out(pins, 1)
    out(pins, 1)
    out(pins, 1)
    out(pins, 1)
    out(pins, 1)
    out(pins, 1)
    out(pins, 1)
    set(pins, 1)	#[1]	# STOP BITS
    jmp(not_osre, "slot") # IF THERE IS ANOTHER BYTE WAITING, LOOP SLOT, ELSE CONTINUE TO WRAP
    wrap()

# Initialize State Machine, DMX freq. is 250kHz, all OUT & SET commands to go via Pin 0.
sm = rp2.StateMachine(0, w, freq=250000, out_base=Pin(0), set_base=Pin(0))

# Activate State Machine
sm.active(1)

# Iterate through 0-255 for channels 1-5 at a rate of ~10Hz
while 1:    
    for b in range(256):
        led.toggle()
        
        # Pass values for DMX channels 1-5
        sm.put(array.array("I", [b, b, b, b, b]), 0)
        time.sleep(1/10)
```
