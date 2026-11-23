# Optional: gradual chamber purge after ASA/ABS (Bambu Lab H2C end G-code)

After an ASA/ABS print the H2C chamber is hot and full of fumes, and the rear vent is closed
(heating air-duct mode). This block in the printer's **end G-code** waits a minute, opens the vent
and ramps the printer's own exhaust fan 20 % → 40 % → 60 % → 100 %, so the part on the bed
cools slowly instead of being hit by cold air.

It runs on the printer itself, so cloud, camera and Bambu Handy keep working
(controlling the printer from Home Assistant would need LAN-only + Developer Mode).

The [blueprint](../blueprints/exhaust_fan_bambu_lab.yaml) detects this purge (air duct switches
heating → cooling while the job is still running) and keeps the duct fans at run-on speed,
then runs on for the set time after the job ends.

## Where to paste

Bambu Studio → printer settings (H2C) → **Machine G-code → Machine end G-code**. Paste it
**between** the second `;=====printer finish air purification=========` line and the first
`;=====printer finish  sound=========` line, so the end-of-print sound plays after the purge.
Save it as your own printer preset.

```gcode
;=====custom: chamber purge after a heating-mode print (ASA/ABS)=====
{if (overall_chamber_temperature >= 40)}
M400 S60         ; 1 min with the vent still closed
M145 P0          ; air duct -> cooling (opens the rear vent)
M106 P3 S51      ; exhaust 20 %
M400 S180        ; 3 min
M106 P3 S102     ; 40 %
M400 S180        ; 3 min
M106 P3 S153     ; 60 %
M400 S180        ; 3 min
M106 P3 S255     ; 100 %
M400 S600        ; 10 min
M106 P3 S0       ; exhaust off
{endif}
;=====custom: end of purge=====
```

About 20 minutes in total. `{if overall_chamber_temperature >= 40}` is the same condition the
stock start G-code uses to pick heating mode, so the block runs only for ASA/ABS/PC/PA prints.
Commands (`M145`, `M106 P3`, `M400 S`) come from Bambu's own H2C G-code.

Notes:
- The printer reports the job as running until the block finishes (progress stays at 100 %).
- Use at your own risk; check it on your printer and firmware. Not needed for the exhaust fan to work.
