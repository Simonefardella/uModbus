.. image:: https://travis-ci.org/AdvancedClimateSystems/uModbus.svg
   :target: https://travis-ci.org/AdvancedClimateSystems/uModbus

.. image:: https://coveralls.io/repos/AdvancedClimateSystems/uModbus/badge.svg?service=github
    :target: https://coveralls.io/github/AdvancedClimateSystems/uModbus

.. image:: https://img.shields.io/pypi/v/uModbus.svg
    :target: https://pypi.python.org/pypi/uModbus

uModbus
=======

uModbus or (μModbus) is a pure Python implementation of the Modbus protcol as
described in the `MODBUS Application Protocol Specification V1.1b3`_. The "u"
or "μ" in the name comes from the the SI prefix "micro-". uModbus is very small
and lightweight. The source can be found on GitHub_. Documentation is avaible
at `Read the Docs`_.

Routing Modbus requests is easy:

.. 
    Because GitHub doesn't support the include directive the source of
    scripts/examples/simple_data_store.py has been copied to this file.

.. code:: python

    #!/usr/bin/env python
    # scripts/examples/simple_data_store.py
    import logging
    from collections import defaultdict

    from umodbus import get_server
    from umodbus.utils import log_to_stream

    # Add stream handler to logger 'uModbus'.
    log_to_stream(level=logging.DEBUG)

    # A very simple data store which maps addresss against their values.
    data_store = defaultdict(int)

    app = get_server('localhost', 502)


    @app.route(slave_ids=[1], function_codes=[3, 4], addresses=list(range(0, 10)))
    def read_data_store(slave_id, address):
        """" Return value of address. """
        return data_store[address]


    @app.route(slave_ids=[1], function_codes=[6, 16], addresses=list(range(0, 10)))
    def write_data_store(slave_id, address, value):
        """" Set value for address. """
        data_store[address] = value

    if __name__ == '__main__':
        try:
            app.serve_forever()
        finally:
            app.stop()

Features
--------

The following Modbus functions have been implemented.

* 01: Read Coils
* 02: Read Discrete Inputs
* 03: Read Holding Registers
* 04: Read Input Registers
* 05: Write Single Coil
* 06: Write Single Register
* 15: Write Multiple Coils
* 16: Write Multiple Register

Road map
--------

uModbus is far from complete. The next, unordered list shows what is going
to be implemented in the future:

* Support for all Modbus functions
* Modbus RTU
* Use asyncio for handling of requests
* Other Modbus 'flavours', so uModbus is able to handle 32 bits or signed
  values

License
-------

uModbus software is licensed under `Mozilla Public License`_. © 2015 `Advanced
Climate Systems`_.

.. External References:
.. _Advanced Climate Systems: http://www.advancedclimate.nl/
.. _GitHub: https://github.com/AdvancedClimateSystems/uModbus/
.. _MODBUS Application Protocol Specification V1.1b3: http://modbus.org/docs/Modbus_Application_Protocol_V1_1b3.pdf
.. _Mozilla Public License: https://github.com/AdvancedClimateSystems/uModbus/blob/develop/LICENSE
.. _Read the Docs: http://umodbus.readthedocs.org/en/latest/