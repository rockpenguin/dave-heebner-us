---
title: "Configuring SNMPv3 on Juniper SRX / Junos"
date: 2017-01-14
categories: 
  - "recipes"
---

I love my Juniper gear, but some things are definitely more "verbose" to setup than need be, and SNMPv3 is one of them. And the KB docs don't [really help](http://www.juniper.net/techpubs/en_US/junos11.4/topics/task/configuration/snmpv3-minimum-config-junos-nm.html) [either](https://kb.juniper.net/InfoCenter/index?page=content&id=KB22048&actp=search). I found setting up OSPF easier than SNMPv3! But given the flexibility that Junos provides, I guess I'll let them slide ;-)

<!--more-->

With many of the features in the Junos CLI, I am usually able to get it done using the autocomplete, but the SNMPv3 config is a tad confusing, at least in terms of how it needs to be configured. With other devices, it's usually just a matter of choosing SNMPv3, setting up the username and AuthPriv settings. But there is a method to Juniper's madness. [Ethan Banks](https://twitter.com/ecbanks) explains it well in this [post](http://ethancbanks.com/2014/02/14/junos-snmpv3-config-baseline-for-all-oid-access-using-usm-with-authorization-privacy/).  Basically the flow is this:

```
Setup USM user --> Setup VACM view for access --> Tell SNMP what OIDs the view is comprised of
```

The USM (user security model) is what allows you to define the user and the corresponding authentication/privacy settings (or _none_, but then why are you using SNMPv3???).  The VACM (view-based access control model) is what allows you to map the USM security "user" to a particular view, or "group" with read/write/no permissions.  Well, that's an over simplification, but that's how I make sense of it in my pea-brain.  It becomes more clear when you view the actual SNMP config:

```
root@rtr-srx> show configuration snmp
description "WAN Router";
location "Some rack, somewhere on Earth";
contact "Mr. NOC Admin";
v3 {
    usm {
        local-engine {
            user SNMPVIEWER {
                authentication-sha {
                    authentication-key "encrypted-data-here";
                }
                privacy-aes128 {
                    privacy-key "encrypted-data-here";
                }
            }
        }
    }
    vacm {
        security-to-group {
            security-model usm {
                security-name SNMPVIEWER {
                    group SNMPVIEWERS;
                }
            }
        }
        access {
            group SNMPVIEWERS {
                default-context-prefix {
                    security-model usm {
                        security-level privacy {
                            read-view SNMP-RO; ## READ-ONLY ACCESS
                        }
                    }
                }
            }
        }
    }
}
view SNMP-RO {
    oid .1 include; ## ALLOWS ACCESS TO ALL SNMP MIBS
}
```

This is the "minimal" config I've used to setup SNMPv3 on my Junos gear, YMMV. Obviously, Juniper's intent is that you can setup access to SNMP data in a very granular way.  So for instance, you could create a USM user "SNMPWRITER" that has write access to only a subset of the OIDs so that helpdesk folks can reset ports on the campus access switches.

Anyhoo, for an in-depth read on setting up SNMPv3 on Junos, the best source is the Junos OS [Network Management Administration Guide](http://www.juniper.net/techpubs/en_US/junos15.1/topics/concept/snmp-v3-overview-junos-nm.html).
