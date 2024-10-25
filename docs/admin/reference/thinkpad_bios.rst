Upgrading the BIOS on Lenovo ThinkPad laptops
=============================================

.. _thinkpad_bios:

The instructions below assume the use of a Linux-based computer for the creation of a BIOS upgrade USB. To upgrade the BIOS:

- Locate the ThinkPad's "machine type" in its BIOS setup program:

  #. Boot (or reboot) the ThinkPad and follow the prompts to enter setup, usually via the <Enter> and <F1> keys.
  #. On the **Main** tab, look for the **Machine Type Model**.  The first four characters, such as `20L5`, `20L6`, or `20S0`, are the machine type.

- Visit `<https://support.lenovo.com>`_ in the Linux-based computer. Type the machine type found above into the search bar, then press **Enter**.
- In the "Product Home" page, select **Drivers And Software** and choose **BIOS/UEFI**.
- Download the file called either **BIOS Update (Bootable CD)** or **BIOS Update (Utility & Bootable CD)**.

.. note::
  A Tails USB can be used for the verification and conversion process described below, but the Lenovo support site blocks requests over Tor, preventing the ISO download. To work around this, either:

  - download the BIOS ISO on a different computer and transfer it to Tails using a USB stick, or
  - download the ISO in Tails using the Unsafe Browser as follows:

    - Start Tails with an administration password set and the Unsafe Browser enabled under "Additional Settings" on the Welcome Screen.
    - Open the Unsafe Browser: **Applications > Internet > Unsafe Browser** and find and download the ISO
    - Note the filename, as you'll need it for subsequent steps.
    - Leave the Unsafe Browser running, and open a terminal via **Applications > System Tools > Terminal**.
    - Copy the ISO to the desktop with the command:

      .. code-block:: sh

        sudo cp /var/lib/unsafe-browser/chroot/home/clearnet/Downloads/<fileName.iso> ~amnesia/Desktop

    - Fix the ISO file's ownership with the command:

      .. code-block:: sh

        sudo chown amnesia:amnesia ~amnesia/Desktop/<fileName.iso>

- Verify the checksum of the downloaded ISO file using the following command, comparing it against the checksum in the file listing above:

  .. code-block:: sh

    sha256sum /path/to/downloaded.iso

- Create a USB-bootable version of the ISO using the command:

  .. code-block:: sh

    geteltorito <path/to/CDISO> > usb-bios.iso

  .. note:: To install the ``geleltorito`` utility on Debian-based systems, use the command

    .. code-block:: sh

      sudo apt install genisoimage

    To install it on Fedora-based systems, use the command:

    .. code-block:: sh

      sudo dnf install geteltorito genisoimage

- Plug in a USB and check its device name with the ``lsblk`` command - use the root device name below, not a partition (eg. ``/dev/sdc`` instead of ``/dev/sdc1``).

- Write the BIOS update ISO to the USB using the following command:

  .. code-block:: sh

    sudo dd if=usb-bios.iso of=/dev/sdX bs=1M && sync

  where ``sdX`` is the device name verified above.

  .. caution::

    The ``dd`` command will wipe data on the targeted device. Make sure that you use the correct device name.

  Once complete, remove the USB.

- Plug the USB into the ThinkPad.

- Boot the ThinkPad and follow the prompts to enter its startup and boot menus, likely via the <Enter> and <F12> keys, respectively.

- Follow the on-screen instructions to update the BIOS, including any mandatory reboots. Note that the instructions may refer to an update CD instead of your update USB.
