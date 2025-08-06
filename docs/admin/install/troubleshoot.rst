Troubleshooting Installation Errors
===================================

Error configuring SecureDrop Workstation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Failed to import Submission Private Key
---------------------------------------

If importing the submission key  using ``sdw-admin --configure`` fails, you can also copy the submission key manually.

- Open a ``dom0`` terminal by opening the **Q Menu**, selecting the gear icon on the left-hand side, then selecting **Other > Xfce Terminal**. Once the Terminal window opens, run the following command to list the SVS submission key details, including its fingerprint:

  .. code-block:: sh

    qvm-run --pass-io vault \
      "gpg --homedir /run/media/user/TailsData/gnupg -K --fingerprint"

- Next, run the comand:

  .. code-block:: sh

    qvm-run --pass-io vault \
      "gpg --homedir /run/media/user/TailsData/gnupg --export-secret-keys --armor <SVSFingerprint>" \
      > /tmp/sd-journalist.sec

  where ``<SVSFingerprint>`` is the submission key fingerprint, typed as a single unit without whitespace. This will copy the submission key in ASCII format to a temporary file in dom0, ``/tmp/sd-journalist.sec``.

- Verify the that the file starts with ``-----BEGIN PGP PRIVATE KEY BLOCK-----`` using the command:

  .. code-block:: sh

    head -n 1 /tmp/sd-journalist.sec

- Unmount the SVS USB 

- Run the following command in the ``dom0`` terminal:

  .. code-block:: sh

    sudo cp /tmp/sd-journalist.sec /usr/share/securedrop-workstation-dom0-config/

- Proceed with :ref:`configuring the workstation<manual_configure>`

.. _manual_copy_journalist: 

Failed to import *Journalist Interface* details
-----------------------------------------------

If importing the *Journalist Interface* details using ``sdw-admin --configure`` fails, you can copy the configuration to ``dom0`` manually.

- Copy the *Journalist Interface* configuration file to ``dom0``. If your SecureDrop instance uses v3 onion services, use the following command:

  .. code-block:: sh

    qvm-run --pass-io vault \
      "cat /run/media/user/TailsData/Persistent/securedrop/install_files/ansible-base/app-journalist.auth_private" \
      > /tmp/journalist.txt

- Verify that the ``/tmp/journalist.txt`` file on ``dom0`` contains valid configuration information using the command ``cat /tmp/journalist.txt`` in the ``dom0`` terminal.

- Proceed with :ref:`configuring the workstation<manual_configure>`


If you encounter a validation error due to a password-protected GPG key, see :doc:`/admin/reference/removing_gpg_passphrase`.

.. _manual_configure:

Configure SecureDrop Workstation
--------------------------------

Once the *Journalist Interface* details and submission key have been copied to ``dom0``, you can create the configuration for the SecureDrop Workstation.

- Your submission key has a unique fingerprint required for the configuration. Obtain the fingerprint by using this command:

  .. code-block:: sh

    gpg --with-colons --import-options import-show --dry-run --import /tmp/sd-journalist.sec

  The fingerprint will be on a line that starts with ``fpr``. For example, if the output included the line ``fpr:::::::::65A1B5FF195B56353CC63DFFCC40EF1228271441:``, the fingerprint would be the character sequence ``65A1B5FF195B56353CC63DFFCC40EF1228271441``.

- Next, create the SecureDrop Workstation configuration file:

  .. code-block:: sh

    cd /usr/share/securedrop-workstation-dom0-config
    sudo cp config.json.example config.json

- The ``config.json`` file must be updated with the correct values for your instance. Open it with root privileges in a text editor such as ``vi`` or ``nano`` and update the following fields' values:

  - **submission_key_fpr**: use the value of the submission key fingerprint as displayed above
  - **hidserv.hostname**: use the hostname of the *Journalist Interface*, including the ``.onion`` TLD
  - **hidserv.key**: use the private v3 onion service authorization key value
  - **environment**: use the value ``prod``

.. note::

   You can find the values for the **hidserv.*** fields in the ``/tmp/journalist.txt`` file that you created in ``dom0`` earlier.
   The file will be formatted as follows:

   .. code-block:: none

     ONIONADDRESS:descriptor:x25519:AUTHTOKEN

- Verify that the configuration is valid using the command below in the ``dom0`` terminal:

  .. code-block:: sh

    sdw-admin --validate


"Failed to return clean data"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An error similar to the following may be displayed during an installation or update:

.. code-block:: none

  sd-log:
        ----------
        _error:
            Failed to return clean data
        retcode:
            None
        stderr:
        stdout:
            deploy

This is a transient error that may affect any of the SecureDrop Workstation VMs. To clear it, run the installation command or update again.

"Temporary failure resolving"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Transient network issues may cause an installation to fail. To work around this, verify that you have a working Internet connection, and re-run the ``sdw-admin --apply`` command.

.. _reset_pci:

"Unable to reset PCI device"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On some hardware, network devices (Ethernet and Wi-Fi) will not immediately work out of the box and require a one-time manual configuration on install. After Qubes starts for the first time, ``sys-net`` will fail to start:

|screenshot_sys_net_pci_reset|

Open a ``dom0`` terminal via **Q > Gear Icon (left-hand side) > Other Tools > Xfce Terminal**, and run the following command to list the devices connected to the ``sys-net`` VM.

.. code-block:: sh

  qvm-pci ls sys-net


This will return the two devices (Ethernet and WiFi) that are connected to ``sys-net``:

.. code-block:: sh

  BACKEND:DEVID  DESCRIPTION                                                            USED BY
  dom0:00_14.3   Network controller: Intel Corporation                                  sys-net
  dom0:00_1f.6   Ethernet controller: Intel Corporation Ethernet Connection (5) I219-V  sys-net


For both device IDs (e.g. ``dom0:00_1f.6`` and ``dom0:00_14.3``), you will need to detach and re-attach the device to ``sys-net``, then restart ``sys-net`` as follows:

.. code-block:: sh

  qvm-pci detach sys-net dom0:00_14.3
  qvm-pci detach sys-net dom0:00_1f.6
  qvm-pci attach sys-net --persistent --option no-strict-reset=True dom0:00_14.3
  qvm-pci attach sys-net --persistent --option no-strict-reset=True dom0:00_1f.6
  qvm-start sys-net


``sys-net`` should now start, and network devices will be functional. This change is only required once on first install.  See the `Qubes documentation of this issue <https://www.qubes-os.org/doc/pci-troubleshooting/#unable-to-reset-pci-device-errors>`_ for more information.

.. |screenshot_sys_net_pci_reset| image:: ../../images/screenshot_sys_net_pci_reset.png

Full system freezes
~~~~~~~~~~~~~~~~~~~

A `known issue <https://github.com/QubesOS/qubes-issues/issues/8825>`_ with some hardware results in Qubes fully freezing.
If you encounter this issue, you will need to forcibly restart your computer, usually by holding down the power button.

When you boot up, you will see a black-and-white menu with the following options:

.. code-block:: text

  Qubes, with Xen hypervisor
  Advanced options for Qubes (with Xen hypervisor)
  UEFI Firmware Settings

While ``Qubes, with Xen hypervisor`` is selected, press :kbd:`e` to edit the option. You should now see a rudimentary
edit interface.

Find the line that starts with ``multiboot2   /xen-`` and ends with ``${xen_rm_opts}``. Use the arrow keys to move your
cursor to before ``${xen_rm_opts}`` and type :kbd:`cpufreq=xen:hwp=off` (leave a space between ``off`` and the ``$``.

Press :kbd:`Ctrl-x` to continue with booting. This will fix the current boot, we now need to make the fix permanent.

Once Qubes has started and you have logged in, open a ``dom0`` terminal via **Q > Gear Icon (left-hand side) > Other Tools > Xfce Terminal** and type
:kbd:`sudo nano /etc/default/grub` to start an editor.

Move your cursor to the bottom of the file and add: :kbd:`GRUB_CMDLINE_XEN_DEFAULT="$GRUB_CMDLINE_XEN_DEFAULT cpufreq=xen:hwp=off"`

Press :kbd:`Ctrl-x`, then :kbd:`y`, and then :kbd:`Enter` to save the file.

Finally, in the terminal run :kbd:`sudo grub2-mkconfig -o /boot/grub2/grub.cfg`. The workaround will now automatically be applied
going forwards.

.. |Attach TailsData| image:: images/attach_usb.png
  :width: 100%
.. |Unlock Tailsdata| image:: images/unlock_tails_usb.png
  :width: 100%

Getting Support
~~~~~~~~~~~~~~~
.. include:: ../../includes/getting_support.rst
