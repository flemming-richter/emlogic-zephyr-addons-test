.. _emlogic_thread_demo:

EmLogic thread demo
###################

Overview
********

A simple sample that can be used with any :ref:`supported board <boards>` and
prints "EmLogic thread demo" to the console.

Building and Running
********************

This application can be built and executed on QEMU as follows:

.. zephyr-app-commands::
   :zephyr-app: samples/emlogic_thread_demo
   :host-os: unix
   :board: qemu_x86
   :goals: run
   :compact:

To build for another board, change "qemu_x86" above to that board's name.

Sample Output
=============

.. code-block:: console

    EmLogic thread demo x86

Exit QEMU by pressing :kbd:`CTRL+A` :kbd:`x`.
