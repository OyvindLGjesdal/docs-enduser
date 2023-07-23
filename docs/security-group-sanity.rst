Security Group Sanity Check
===========================

.. _CIDR: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
.. _CIDR (Wikipedia): https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
.. _CIDR Calculator IPv6: https://www.vultr.com/resources/subnet-calculator-ipv6/
.. _CIDR Calculator IPv4: https://www.vultr.com/resources/subnet-calculator/

.. contents::

Security groups are firewalls in OpenStack. They consist of one or
more firewall rules, called **security group rules**. Usually, a
security group rule is used to open one or more ports to one or more
IP addresses, giving access to servives on the instances on which the
security group is applied.

Creating security group rules can be easy or difficult, depending on
one's understanding of networking and firewalls. Regardless, it's easy
to make mistakes and open up more than intended. For this reason,
security group rules are checked.

Each security group rule is checked every day. This check is automatic
and looks for simple mistakes by the user:

* Wrong netmask for the given IP address
* The rule opens too many ports to too many IP addresses

If the check finds a discrepancy, an email is sent to the project
admin, simply to make the admin aware of the issue. No further action
is taken. If the issue persists, a new email is sent after 30 days,
then every 30 days until the issue is fixed.


General Considerations
----------------------

The following general considerations governs the check:

* Only single rules are checked. We do not consider complete security
  groups, or the combination of security groups that comprise the
  firewall in front of an instance

* The IP address ranges of UiO and UiB are whitelisted. As long as the
  IP address and netmask decribes a subset of these ranges, the rule
  is ignored

* Ports 22 (ssh), 80 (http) and 443 (https) are whitelisted. Any rule
  that opens to one of these three ports is ignored

* Only ingress rules (incoming traffic) are checked. Egress rules
  (outgoing traffic) are ignored

* Only rules that use a CIDR_ address as target are checked. Rules
  that use a security group as target are ignored


Wrong Netmask
-------------

The system accepts what we consider to be a «wrong» netmask. For
example, the system accepts ``129.240.114.36/0`` as a valid CIDR_
address. However, this is identical to ``0.0.0.0/0`` as a netmask of
``0`` negates the IP address in its entirety. This is an extreme
example, in which the user probably wanted to give access to a single
host but ended up giving access to the entire internet. The correct
CIDR_ address that describes a single host in this case would be
``129.240.114.36/32``.

A less extreme example would be ``129.240.114.0/16``. The netmask of
``16`` is too low number, and this CIDR_ address is effectively the
same as ``129.240.0.0/16``. Here the user probably meant
``129.240.114.0/24`` or similar.

These are IPv4 examples. The same applies to IPv6. For example, the
CIDR_ address ``2001:700:100:8070::36/0`` is valid, but the netmask
``0`` negates the entire IP address and this CIDR is then exactly the
same as ``::/0``, the entire internet.

If you get an alert about wrong netmask, the email will also describe
the minimal netmask that makes sense.


Port Limits
-----------

The NREC team has created a set of port limits, which describes the
maximum number of ports one should open for a specific netmask:

* For netmasks ``0`` through ``16`` (IPv4) or ``0`` through ``64``
  (IPv6), only a single port is considered safe

* For netmask ``32`` (IPv4) or ``128`` (IPv6), you may open all 65536
  ports

* For netmask between ``16`` and ``32`` (IPv4) or ``64`` and ``128``,
  the number of ports that is considered safe is calculated by the
  formula :math:`2^{mask - 16}` (IPv4) or :math:`2^{\frac{mask - 64}{4}}` (IPv6) 