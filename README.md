# ATTEROTECH MCHstreamer Talker A4

Firmware version: `6.1.1-HYB4.118-A4-MSRP-REARM`

Device name: `MCHstreamer 4WAY DSP Talker`

## Purpose

The August 10 logs show a valid ACMP listener record, correct CRF lock and AAF connection parameters, but `Stream Start = 0` and `Frames TX = 0`. The controller connection exists while the XMOS source remains dormant in `POTENTIAL`, so MSRP/FIFO/packet generation is never re-armed.

## A4 correction

A4 distinguishes a genuinely new listener from a duplicate ACMP request. When the first real listener is added and the source is still stale in `POTENTIAL`, the source is moved through `DISABLED` and then through the normal XMOS initialization path back to `POTENTIAL`. This re-registers the source runtime instead of merely recording the listener in ACMP state.

The re-arm is deliberately guarded:

- it runs only for the first genuinely new listener;
- duplicate ACMP retries do not retrigger it;
- an already running source is not reset;
- the live ACMP connection count continues to be reported.

A4 does not change audio samples, gain, channel order, I2S pin mapping, AAF format, CRF processing, or gPTP selection.

## Validation status

The source and image are build-verified with xTIMEcomposer Community 14.4.1. Hardware/network validation is still required.

## Recommended test

1. Flash `ATTEROTECH_MCHSTREAMER_TALKER_A4.xe` and power-cycle the MCH talker.
2. Connect the MT32 CRF output to the MCH `Input Stream CRF` and confirm MCR lock.
3. Connect MCH `Output Stream AAF` to exactly one listener.
4. Wait 10-15 seconds and take a fresh FullDump and Milan Manager state.

Expected evidence:

- `Stream Start` changes from 0 to 1;
- `Frames TX` increases continuously;
- Milan Manager keeps the connection blue;
- the listener retains Stream ID `0x0006240019130000`;
- audio reaches the listener.

Then disconnect and reconnect the AAF stream once to confirm that the talker remains online and resumes transmission.
