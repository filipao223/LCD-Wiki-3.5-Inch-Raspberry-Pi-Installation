# Installing the MHS-3.5" LCD Touchscreen on Raspberry Pi (Default Full Raspbian 32-bit Image)

This guide will walk you through installing and configuring the MHS-3.5" LCD (480x320) touchscreen on a Raspberry Pi with a default Raspbian OS (32-bit) image.

The guide available over at [LCD Wiki](http://www.lcdwiki.com/3.5inch_RPi_Display) didn't fully work out of the box, requiring some digging to get both the display to work and the touch screen with the correct axis calibration.

For this guide, I specifically had a problem requiring the screen be rotated 180 degrees, for which there are scripts available, but didn't leave the touchscreen correctly configured. You may adjust the final steps to fit your own rotation needs.

## 🧾 Requirements

- Raspberry Pi (any model with GPIO, although for this guide, a RPi 3B+ with 512MB of RAM was used)
- MHS-3.5” LCD (from http://www.lcdwiki.com/MHS-3.5inch_RPi_Display)
- Raspberry Pi OS 32-bit image (Full Desktop version, better support than 64 bit)

---

## Attach the LCD

Plug the MHS-3.5" LCD into the Raspberry Pi GPIO header, following the instructions provided with the screen.

---

## Configure the LCD Driver

These instructions are available at (LCD Wiki)[http://www.lcdwiki.com/3.5inch_RPi_Display#Driver_Installation]

1. **Update your system**:

    Run the following command to ensure your Raspberry Pi is up to date:

    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

2. **Download and install the LCD driver**:

    Download the driver package for the MHS-3.5" LCD:

    ```bash
    git clone https://github.com/goodtft/LCD-show.git
    ```

3. **Run the installation script**:

    Navigate to the LCD-show directory:

    ```bash
    cd LCD-show/
    ```

    Then, run the installation script for your specific LCD model (MHS-3.5" uses the `MHS35` script):

    ```bash
    sudo ./MHS35-show
    ```

    After the installation finishes, the Raspberry Pi will automatically reboot.

---

## Switch from Wayland to X (Default Desktop)

1. **Disable Wayland**:

    In the default Raspbian (Raspberry Pi OS) image, Wayland might be enabled by default. To switch to the Xorg display server, edit the `raspi-config` settings:

    ```bash
    sudo raspi-config
    ```

2. Navigate to `System Options` → `Boot / Auto Login` → Select `Desktop` (this ensures the GUI will load after boot).

3. Exit `raspi-config` and reboot your Raspberry Pi:

    ```bash
    sudo reboot
    ```

The system will now automatically use Xorg and boot into the graphical desktop environment.

---

## Enable and Configure the LCD Screen

The default installation script may not enable the display, in my case it required further changes to the boot config.txt file.

1. Edit the `/boot/config.txt` file to enable the LCD:

    ```bash
    sudo nano /boot/config.txt
    ```

2. Add the following line to enable the LCD screen (this will rotate the display, you can use a different value for a different rotation):

    ```bash
    dtoverlay=piscreen,drm,rotate=180
    ```

3. Save and exit.

4. Reboot the Raspberry Pi:

    ```bash
    sudo reboot
    ```

---

## Fix Touchscreen Axis

1. First, check if the touchscreen is detected by running the following command:

    ```bash
    dmesg | grep -i touchscreen
    ```

   - If the screen is properly detected, you should see output related to the touchscreen device (e.g., ADS7846, which is commonly used in many resistive touchscreens).

2. You can also check the touchscreen device with the `lsinput` command:

    ```bash
    sudo apt install -y input-utils
    sudo lsinput
    ```

   - This will list all input devices, including the touchscreen. Look for entries like "ADS7846 Touchscreen" or similar.

3. Create a new Xorg calibration configuration file:

    ```bash
    sudo nano /etc/X11/xorg.conf.d/99-calibration.conf
    ```

4. Add the following configuration to fix the touchscreen orientation and axis:

    ```ini
    Section "InputClass"
        Identifier      "calibration"
        MatchProduct    "ADS7846 Touchscreen"
        Option          "TransformationMatrix" "0 -1 1 1 0 0 0 0 1"
    EndSection
    ```

This configuration will swap the X and Y axes and invert the Y-axis to match the screen's physical layout.

**Note**: This is going to be specific to your rotation needs, if you need a separate rotation value, you can play around with the transformation matrix values.

5. To apply all the changes, restart the `lightdm` service:
   ```bash
   sudo systemctl restart lightdm
   ```
