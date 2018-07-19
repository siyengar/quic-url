---
title: An httpqs URL scheme for QUIC
abbrev:
docname: draft-subodh-quic-httpqs-latest
date: 2018-07-18
category: std

ipr: trust200902
area: Transport
workgroup: QUIC
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: S. Iyengar
    name: Subodh Iyengar
    organization: Facebook
    email: subodh@fb.com
 -
    ins: M. Bishop
    name: Mike Bishop
    org: Akamai
    email: mbishop@evequefou.be

--- abstract

This document specifies a URL scheme `httpqs` for a browser to discover that a
particular resource is available over both HTTP over QUIC and HTTPS at the same
origin and IP address and port.

--- middle

Introduction        {#problems}
============

A browser can discover that a particular HTTP resource is available via QUIC via
Alt-Svc.  However, in some cases it is difficult to use Alt-Svc to discover the
use of QUIC:

* A website will want to experiment with QUIC before enabling QUIC for CDN use
  cases.  One desired model of experimentation is at a per-user level where a
  website decides whether to assign a user to a test or control group.  A CDN
  typically does not have access to cookies which determine users.  Thus to run
  such an experiment, additional co-operation between the website and the CDN
  would be needed.  If the website decides that the user should use QUIC, it
  cannot normally supply an Alt-Svc for the CDN.  The website instead needs to
  provide a hint to the CDN that it wishes to use QUIC for the resource and and
  the CDN would then supply the Alt-Svc itself.  This process requires an
  additional round-trip to discover the use of QUIC and makes experiments more
  complex.

* A particular origin might not be requested frequently in one browser session.
  For example, it is common for a CDN to cache different content types in
  different server locations and refer to each location with a different URL.
  Once a client requests an HTTP resource over the network, it might not
  request it again over the network, because subsequent tries might receive a
  cached response.  If there are not many requests per origin, then discovering
  than an origin can use QUIC might be too late.

To help make experimentation easier, we propose a new URL scheme for QUIC
`httpqs`.  Unlike Alt-Svc, `httpqs` is limited in functionality and specifies
that the server makes the same resource available over both HTTPS over TCP, as
well as HTTP over QUIC at the same IP address and port.  Alt-Svc scopes the use
of an entire origin, but `httpqs` only defines what happens to that particular
resource.


Server and client behaviors
===========================

Server Behavior
---------------

A CDN that supports the `httpqs` scheme MUST accept connections over HTTP
over QUIC as well as HTTPS on the same IP and also the `https` port.  Accepting
QUIC and TCP connections on the same IP does not require CDNs to listen on QUIC
and TCP on the same host.

Websites that use such CDNs can send `httpqs` URLs wherever a `https` scheme
could have been used.  Discovery that a CDN supports this scheme is out of scope
of this document. We assume that websites and CDNs have a trust relationship
that allows them to discover that they can use this scheme.

Client Behavior
---------------

When a browser recieves a URL to a resource with the scheme `httpqs`, if it does
not have a cached connection for QUIC or TCP, it SHOULD race both a HTTP over
QUIC and a HTTPS connection as soon as it has an IP address for the hostname to
connect to.  The specific policy of racing connections and the tradeoffs between
using pooled connections versus opening new connections are out of scope of this
document.

It is possible that the browser already has an Alt-Svc setting available on the
origin of the `httpqs` resource.

[OPEN QUESTION?]
We have two options here:

* Browsers MUST give `httpqs` higher precedence, as it is a more specific
  marker.  This is a more natural choice.

* Browsers MUST give Alt-Svc a higher precedence. This would allow a service to
  stop accepting `httpqs` URLs at some point.  Should we even support stopping
  accepting `httpqs` URLs.

A browser MAY reuse a pooled connection which was created as a result of
connecting to an Alt-Svc, provided that the origin of the `httpqs` URL matches
the original service of the Alt-Svc.

QUIC transport negotiation
---------------------------

Alt-Svc allows specific QUIC versions as well as ALPNs to be provided as hints
to reduce round trips during connection establishment.  `httpqs` follows the
default transport parameter negotiation as if no version was specified as a
hint even if an Alt-Svc value is available for the origin of the resource.

Load balancing considerations
=============================

CDNs might want to load balance requests differently between QUIC and TCP.
`httpqs` may limit the ability to do so as it is a commitment to support both
HTTP over QUIC and HTTPS over the same IP and port.  CDNs MUST only support this
scheme if they are ready to deal with these tradeoffs.

If a CDN decides it wants to stop supporting QUIC it has several options:

* Stop listening on a QUIC port at the IP. Browsers should fallback to TCP at
  that point.

* Advertise a Version negotiation packet with an invalid version which is not
  supported.


Security Considerations
=======================
For the browser security model `httpqs` schemes should be treated the same as
`https` URLs for same origin checks.

CDNs SHOULD have mitigations in place for Denial of Service attacks where
attacking QUIC would take down the QUIC server as well.

--- back
