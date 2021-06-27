# Client

The main client utility seems to be `net`. Per its manual, there are multiple supported protocols, and `ads` seems to be the modern one.

Debugging can be done by specifying `-d NUMBER` option, up to `10`.

# Misc

* DC: Domain Controller.
* `kerberos` is a protocol. Most known are two implementations: some proprietary solution by MS, and the original MIT licensed solution, named presumably just `kerberos`.
  While Kerberos can be used alone, but usually is used together with Active Directory, where AD domain is mapped to a realm, and AD UPN → Kerberos principal name.
* `KDC`: Key Distribution Center, typically a service provided by DC. A centralized repository for users' password information.
* `realm`: a thing similar to `domain`, but inside Kerberos and case-sensitive *(typically upcased)*.
