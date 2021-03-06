# High-Throughput on Virtual Machines

STTP Data Publisher implementations include a setting called `MaxPublishInterval` which defines the maximum publication interval, in milliseconds, for data publications where a zero value represents no defined maximum, i.e., publish without delay. Note that zero is the default value.

For TCP-only STTP connections with high data transmission volumes running on a Virtual Machine (VM), whose life exists as time-slices running on the parent machine, the STTP strategy of intentionally fragmenting publication packets at the application layer <sup>[[1](#ref1)]</sup> can actually hinder the VM's ability to send all data, i.e., the VM needs to send all the data it can send while its time-slice is active.

The `MaxPublishInterval` setting induces an intentional delay to _hold_ publication data to allow for a bigger cumulative block of data to be sent in the time available. The parameter should be set in the target STTP Data Publisher API, whose actual implementation details will vary.

In the case of the [sttp/gsfapi](https://github.com/sttp/gsfapi), the parameter can be set in the STTP Data Publisher [Action Adapter](https://gridprotectionalliance.org/docs/products/gsf/tsl-components-2015.pdf) connection string parameters using the host manager application. As a connection string parameter, there will be no need for updating the host config file and/or restarting the host service to apply the changes.  Navigate to the `Actions / Custom Actions` screen in the host manager application, find the desired STTP Data Publisher Action Adapter, set the `MaxPublishInterval` connection string parameter to an appropriate value, e.g., `250`, click `Save` on the Action Adapter record, then click `Initialize` to restart the STTP Data Publisher adapter.

<!--
  Links to this bookmark anchor from GitHub markdown require links to be referenced as "user-content-ref1", where as
  GitHub Pages references link by the exact name, i.e., "ref1", we defer to GitHub pages as the operational option.
-->
> <a name="ref1"></a><sup>1</sup> STTP intentionally fragments large data packets before presenting them to the transport layer for better network efficiency. This strategy benefits UDP data transmissions more than it does TCP and works by reducing the number of network fragments that need to be reassembled for a successful UDP transmission. With fewer fragments to reassemble, there are fewer overall data losses since fewer network fragments can be dropped.
