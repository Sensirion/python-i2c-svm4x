Install the SVM4x Driver
--------------------------

.. include:: driver-installation.rst


Use the SensorBridge on Windows
-------------------------------

1. Install the driver for the `Sensirion SEK-SensorBridge`_:

   .. code-block:: console

      python -m pip install sensirion-shdlc-sensorbridge

2. Connect the SEK-SensorBridge to your PC over USB.

   If the SEK-SensorBridge is not detected by your system, follow the
   `SensorBridge FTDI Driver Installation`_.

3. Connect the SVM4x sensor to the SEK-SensorBridge.

4. Run an example script in the activated virtual environment.

   By default, the script assumes that the SensorBridge is connected to the
   ``COM1`` serial port. If a different port is used, specify it with the
   ``--serial-port`` parameter:

   .. code-block:: console

        python examples/example_usage_sensorbridge_svm4x.py --serial-port <your COM port>


.. _Sensirion SEK-SensorBridge: https://developer.sensirion.com/product-support/sek-sensorbridge/
.. _SensorBridge FTDI Driver Installation: https://sensirion.github.io/python-shdlc-sensorbridge/sensor-bridge-installation.html

Example script
~~~~~~~~~~~~~~

.. sourcecode:: python

    import time
    from sensirion_shdlc_driver import ShdlcSerialPort, ShdlcConnection
    from sensirion_shdlc_sensorbridge import SensorBridgePort, \
        SensorBridgeShdlcDevice, SensorBridgeI2cProxy
    from sensirion_i2c_driver import I2cConnection
    from sensirion_i2c_svm41 import Svm41I2cDevice

    # Connect to the SensorBridge with default settings:
    #  - baudrate:      460800
    #  - slave address: 0
    with ShdlcSerialPort(port='COM1', baudrate=460800) as port:
        bridge = SensorBridgeShdlcDevice(ShdlcConnection(port), slave_address=0)
        print("SensorBridge SN: {}".format(bridge.get_serial_number()))

        # Configure SensorBridge port 1 for SVM41
        bridge.set_i2c_frequency(SensorBridgePort.ONE, frequency=100e3)
        bridge.set_supply_voltage(SensorBridgePort.ONE, voltage=3.3)
        bridge.switch_supply_on(SensorBridgePort.ONE)

        # Create SVM41 device
        i2c_transceiver = SensorBridgeI2cProxy(bridge, port=SensorBridgePort.ONE)
        device = Svm41I2cDevice(I2cConnection(i2c_transceiver))

        # Print some device information
        print("Version: {}".format(device.get_version()))
        print("Serial Number: {}".format(device.get_serial_number()))

        # Start measurement
        device.start_measurement()
        print("Measurement started... ")
        while True:
            time.sleep(10.)
            humidity, temperature, air_quality_voc, air_quality_nox = \
                device.read_measured_values()

            # use default formatting for printing output:
            print("{}, {}, {}, {}".format(humidity,
                                          temperature,
                                          air_quality_voc,
                                          air_quality_nox))


Use the Linux I²C Driver
------------------------

On Linux systems, the sensor can alternatively be accessed directly through
the Linux user-space I²C driver.

1. Connect the SVM4x sensor to an I²C port of your system, for example I²C
   port 1 of a Raspberry Pi.

2. Run an example script in the activated virtual environment.

   By default, the script assumes that the sensor is connected to
   ``/dev/i2c-1``. If a different port is used, specify it with the
   ``--i2c-port`` parameter:


Example script
~~~~~~~~~~~~~~

.. sourcecode:: python

    import time
    from sensirion_i2c_driver import I2cConnection, LinuxI2cTransceiver
    from sensirion_i2c_svm41 import Svm41I2cDevice


    with LinuxI2cTransceiver('/dev/i2c-1') as i2c_transceiver:
        device = Svm41I2cDevice(I2cConnection(i2c_transceiver))
        device.device_reset()

        # Print some device information
        print("Version: {}".format(device.get_version()))
        print("Serial Number: {}".format(device.get_serial_number()))

        # Start measurement
        device.start_measurement()
        print("Measurement started... ")
        while True:
            time.sleep(10.)
            humidity, temperature, air_quality_voc, air_quality_nox = \
                device.read_measured_values()

            # use default formatting for printing output:
            print("{}, {}, {}, {}".format(humidity,
                                          temperature,
                                          air_quality_voc,
                                          air_quality_nox))




