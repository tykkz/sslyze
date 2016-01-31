SSLyze
======

Fast and full-featured SSL scanner.


Description
-----------

SSLyze is a Python tool that can analyze the SSL configuration of a server by connecting to it. It is designed to be 
fast and comprehensive, and should help organizations and testers identify mis-configurations affecting their SSL
servers.

Key features include:
* Multi-processed and multi-threaded scanning: it's very fast.
* Support for all SSL protocols, from SSL 2.0 to TLS 1.2.
* Performance testing: session resumption and TLS tickets support.
* Security testing: weak cipher suites, insecure renegotiation, CRIME, Heartbleed and more.
* Server certificate validation and revocation checking through OCSP stapling.
* Support for StartTLS handshakes on SMTP, XMPP, LDAP, POP, IMAP, RDP, PostGres and FTP.
* Support for client certificates when scanning servers that perform mutual authentication.
* **NEW:** SSLyze can also be used as a library, in order to run scans and process the results directly from Python.
* And much more !


Getting Started
---------------

SSLyze can be installed directly via pip:
    
    pip install sslyze

It is also easy to directly clone the repository and the fetch the requirements:

    git clone https://github.com/nabla-c0d3/sslyze.git
    cd sslyze
    pip install -r requirements.txt --target ./lib
    
Then, the command line tool can be used to scan servers:

    python sslyze.py --regular www.yahoo.com:443 www.google.com
    
SSLyze has been tested on the following platforms: Windows 7 (32 and 64 bits), Debian 7 (32 and 64 bits), OS X El 
Capitan.


Usage as a library
------------------

Starting with version 0.13.0, SSLyze can be used as a Python module in order to run scans and process the results 
directly in Python:

```python
# Retrieve the certificate CN from smtp.gmail.com:587; first ensure the server is reachable
hostname = 'smtp.gmail.com'
try:
    server_info = ServerConnectivityInfo(hostname=hostname, port=587,
                                         tls_wrapped_protocol=TlsWrappedProtocolEnum.STARTTLS_SMTP)
    server_info.test_connectivity_to_server()
except ServerConnectivityError as e:
    raise RuntimeError('Error when connecting to {}: {}'.format(hostname, e.error_msg))

# Get the list of available plugins
sslyze_plugins = PluginsFinder()

# Create a process pool to run scanning commands concurrently
plugins_process_pool = PluginsProcessPool(sslyze_plugins)

# Queue a scan command to get the server's certificate
plugins_process_pool.queue_plugin_task(server_info, 'certinfo_basic')

# Process the result and print the certificate CN
for server_info, plugin_command, plugin_result in plugins_process_pool.get_results():
    if plugin_result.plugin_command == 'certinfo_basic':
        print 'Server Certificate CN: {}'.format(plugin_result.certificate_chain[0].as_dict['subject']['commonName'])
```

The scan commands are same as the ones described in the SSLyze CLI `--help` text. 

They will all be run concurrently using Python's multiprocessing module. Each command will return a `PluginResult` 
object with attributes with the result of the scan command run on the server (such as list of supported cipher suites 
for the `--tlsv1` command). These attributes are specific to each plugin and command but are all documented (within each 
plugin's module).

See _api\_sample.py_ for more examples of SSLyze's Python API.


Windows executable
------------------

A pre-compiled Windows executable is available in the _Releases_ tab. The package can also be generated by running the 
following command:

    python.exe setup_py2exe.py py2exe
    

How does it work ?
------------------

SSLyze is all Python code but it uses an 
[OpenSSL wrapper written in C called nassl](https://github.com/nabla-c0d3/nassl), which was specifically developed for
allowing SSLyze to access the low-level OpenSSL APIs needed to perform deep SSL testing.


Where do the trust stores come from?
------------------------------------

The Mozilla, Microsoft, Apple and Java trust stores are downloaded using the following tool: 
https://github.com/nabla-c0d3/catt/blob/master/sslyze.md .


License
-------

GPLv2 - See LICENSE.txt.
