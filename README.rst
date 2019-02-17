uModbus
=======

This is a fork with some additional features of the original uModbus library, more specifically:

- Add the features of reading/writing on devices/systems with values bigger than 16 Bit data
- Add the flush_on_write parameter to disable the auto flush after writing (some hardware drivers auto manage the flush on write)


uModbus or (μModbus) is a pure Python implementation of the Modbus protocol as
described in the `MODBUS Application Protocol Specification V1.1b3`_. uModbus
implements both a Modbus client (both TCP and RTU) and a Modbus server (both
TCP and RTU). The "u" or "μ" in the name comes from the the SI prefix "micro-".
uModbus is very small and lightweight. The source can be found on GitHub_.
Documentation is available at `Read the Docs`_.

Quickstart
----------

Creating a Modbus TCP server is easy:

..
    Because GitHub doesn't support the include directive the source of
    scripts/examples/simple_tcp_server.py has been copied to this file.

.. code:: python

    #!/usr/bin/env python
    # scripts/examples/simple_tcp_server.py
    import logging
    from socketserver import TCPServer
    from collections import defaultdict

    from umodbus import conf
    from umodbus.server.tcp import RequestHandler, get_server
    from umodbus.utils import log_to_stream

    # Add stream handler to logger 'uModbus'.
    log_to_stream(level=logging.DEBUG)

    # A very simple data store which maps addresss against their values.
    data_store = defaultdict(int)

    # Enable values to be signed (default is False).
    conf.SIGNED_VALUES = True

    TCPServer.allow_reuse_address = True
    app = get_server(TCPServer, ('localhost', 502), RequestHandler)


    @app.route(slave_ids=[1], function_codes=[3, 4], addresses=list(range(0, 10)))
    def read_data_store(slave_id, function_code, address):
        """" Return value of address. """
        return data_store[address]


    @app.route(slave_ids=[1], function_codes=[6, 16], addresses=list(range(0, 10)))
    def write_data_store(slave_id, function_code, address, value):
        """" Set value for address. """
        data_store[address] = value

    if __name__ == '__main__':
        try:
            app.serve_forever()
        finally:
            app.shutdown()
            app.server_close()

Doing a Modbus request requires even less code:

..
    Because GitHub doesn't support the include directive the source of
    scripts/examples/simple_data_store.py has been copied to this file.

.. code:: python

    #!/usr/bin/env python
    # scripts/examples/simple_tcp_client.py
    import socket

    from umodbus import conf
    from umodbus.client import tcp

    # Enable values to be signed (default is False).
    conf.SIGNED_VALUES = True

    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(('localhost', 502))

    # Returns a message or Application Data Unit (ADU) specific for doing
    # Modbus TCP/IP.
    message = tcp.write_multiple_coils(slave_id=1, starting_address=1, values=[1, 0, 1, 1])

    # Response depends on Modbus function code. This particular returns the
    # amount of coils written, in this case it is.
    response = tcp.send_message(message, sock)

    sock.close()

Features
--------

The following functions have been implemented for Modbus TCP and Modbus RTU:

* 01: Read Coils
* 02: Read Discrete Inputs
* 03: Read Holding Registers
* 04: Read Input Registers
* 05: Write Single Coil
* 06: Write Single Register
* 15: Write Multiple Coils
* 16: Write Multiple Registers

Other featues:

* Support for signed and unsigned register values.
* Easy data packing and unpacking (Write and Read) with the methods data_packer & data_unpacker

License
-------

uModbus software is licensed under `Mozilla Public License`_. © 2018.

.. External References:
.. _GitHub: https://github.com/AdvancedClimateSystems/uModbus/
.. _MODBUS Application Protocol Specification V1.1b3: http://modbus.org/docs/Modbus_Application_Protocol_V1_1b3.pdf
.. _Mozilla Public License: https://github.com/AdvancedClimateSystems/uModbus/blob/develop/LICENSE
.. _Read the Docs: http://umodbus.readthedocs.org/en/latest/
