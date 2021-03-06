---
layout: page
title: fi_shm(7)
tagline: Libfabric Programmer's Manual
---
{% include JB/setup %}

# NAME

fi_shm \- The SHM Fabric Provider

# OVERVIEW

The SHM provider is a complete provider that can be used on Linux
systems supporting shared memory and process_vm_readv/process_vm_writev
calls.  The provider is intended to provide high-performance communication
between processes on the same system.

# SUPPORTED FEATURES

This release contains an initial implementation of the SHM provider that
offers the following support:

*Endpoint types*
: The provider supports only endpoint type *FI_EP_RDM*.

*Endpoint capabilities*
: Endpoints cna support any combinations of the following data transfer
capabilities: *FI_MSG*, *FI_TAGGED*, *FI_RMA*, amd *FI_ATOMICS*.  These
capabilities can be further defined by *FI_SEND*, *FI_RECV*, *FI_READ*,
*FI_WRITE*, *FI_REMOTE_READ*, and *FI_REMOTE_WRITE* to limit the direction
of operations.

*Modes*
: The provider does not require the use of any mode bits.

*Progress*
: The SHM provider supports *FI_PROGRESS_MANUAL*.  Receive side data buffers are
  not modified outside of completion processing routines.  The provider processes
  messages using three different methods, based on the size of the message.
  For messages smaller than 4096 bytes, tx completions are generated immediately
  after the send.  For larger messages, tx completions are not generated until
  the receiving side has processed the message.

*Address Format*
: The SHM provider uses the address format FI_ADDR_STR, which follows the general
  format pattern "[prefix]://[addr]".  The application can provide addresses
  through the node or hints parameter.  As long as the address is in a valid
  FI_ADDR_STR format (contains "://"), the address will be used as is.  If the
  application input is incorrectly formatted or no input was provided, the SHM
  provider will resolve it according to the following SHM provider standards:

  (flags & FI_SOURCE) ? src_addr : dest_addr =
   - if (node && service) : "fi_ns://node:service"
   - if (service) : "fi_ns://service"
   - if (node && !service) : "fi_shm://node"
   - if (!node && !service) : "fi_shm://PID"

   !(flags & FI_SOURCE)
   - src_addr = "fi_shm://PID"

  In other words, if the application provides a source and/or destination
  address in an acceptable FI_ADDR_STR format (contains "://"), the call
  to util_getinfo will successfully fill in src_addr and dest_addr with
  the provided input.  If the input is not in an ADDR_STR format, the
  shared memory provider will then create a proper FI_ADDR_STR address
  with either the "fi_ns://" (node/service format) or "fi_shm://" (shm format)
  prefixes signaling whether the addr is a "unique" address and does or does
  not need an extra endpoint name identifier appended in order to make it
  unique.  For the shared memory provider, we assume that the service
  (with or without a node) is enough to make it unique, but a node alone is
  not sufficient.  If only a node is provided, the "fi_shm://" prefix  is used
  to signify that it is not a unique address.  If no node or service are
  provided (and in the case of setting the src address without FI_SOURCE and
  no hints), the process ID will be used as a default address.
  On endpoint creation, if the src_addr has the "fi_shm://" prefix, the provider
  will append ":[uid]:[dom_idx]:[ep_idx]" as a unique endpoint name (essentially,
  in place of a service).  In the case of the "fi_ns://" prefix (or any other
  prefix if one was provided by the application), no supplemental information
  is required to make it unique and it will remain with only the
  application-defined address.  Note that the actual endpoint name will not
  include the FI_ADDR_STR "*://" prefix since it cannot be included in any
  shared memory region names. The provider will strip off the prefix before
  setting the endpoint name. As a result, the addresses
  "fi_prefix1://my_node:my_service" and "fi_prefix2://my_node:my_service"
  would result in endpoints and regions of the same name.
  The application can also override the endpoint name after creating an
  endpoint using setname() without any address format restrictions.

*Msg flags*
  The provider currently only supports the FI_REMOTE_CQ_DATA msg flag.

*MR registration mode*
  The provider implements FI_MR_VIRT_ADDR memory mode.

*Atomic operations*
  The provider supports all combinations of datatype and operations as long
  as the message is less than 4096 bytes (or 2048 for compare operations).

# LIMITATIONS

The SHM provider has hard-coded maximums for supported queue sizes and data
transfers.  These values are reflected in the related fabric attribute
structures

EPs must be bound to both RX and TX CQs.

No support for counters.

# RUNTIME PARAMETERS
*FI_SHM_DISABLE_CMA*
: Force disable use of CMA (Cross Memory Attach) in shm environment. CMA is a
  Linux feature for copying data directly between two processes without the use
  of intermediate buffering. This requires the processes to have full access to
  the peer's address space (the same permissions required to perform a ptrace).
  CMA is enabled by default but checked for availability during run-time.
  For more information see the CMA [`man pages`]
  (https://linux.die.net/man/2/process_vm_writev)

# SEE ALSO

[`fabric`(7)](fabric.7.html),
[`fi_provider`(7)](fi_provider.7.html),
[`fi_getinfo`(3)](fi_getinfo.3.html)
