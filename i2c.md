# Ansteuerung der I2C-Schnittstelle

---

## I2C-Schnittstelle aktivieren:

> Aktiviere das I2C-Modul auf dem RPi. Eventuell können Einstellungen in den folgenden Dateien notwendig sein:
> 
> - /etc/modules (notwendig)
> - /boot/config.txt (notwendig)
> - /etc/modprobe.d/custom.conf
> - /sys/module/i2c_bcm2708/parameters/baudrate

bash    `sudo raspi-config`

select  `Interfacing Options`

select  `I2C` and confirm

## Kontrolle 

> Verifiziere mit dem Befehl "lsmod", dass das I2C-Modul korrekt aktiviert worden ist.

`bash`
```bash
$ lsmod | grep i2c
```

`output`
```bash
i2c_dev         12345     0
i2c_bcm2708     12345     0
```

währe der Prozess **i2c_bcm2708** nicht geladen, so ware i2c nicht aktiviert.

## I2C-Tools

```bash
$ sudo apt-get install i2c-tools
```