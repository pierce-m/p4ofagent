# P4 Openflow Agent
This is the controller-facing layer of a P4 agnostic Openflow 1.3 Agent.
## Building
To build, run

`git submodule update --init --recursive`

`sudo autoreconf -fi`

`./configure`

This generates a Makefile with target `p4ofagent` to be required by a [p4factory](https://github.com/p4lang/p4factory) target Makefile.

## Features
The P4 Openflow Agent is P4-independent, so features vary with the source P4 program. With everything enabled, the `openflow.p4` module supports:

1. Forwarding

2. Multicast (OFPGT_ALL) groups.

3. Vlan push/pop/set

4. MPLS push/pop/ttl-set

5. Network ttl set

6. Packet in, packet out

## Dependencies
#### PD
The P4 Openflow Agent depends on the PD (Protocol Dependent) API generated by [the P4 compiler](https://github.com/p4lang/p4c-behavioral) when building a p4factory target. It is intended to be used as a submodule to p4factory.

## Adding Openflow to a Behavioral Model target
#### The P4 Source
There is an `openflow.p4` file in the `p4src/` directory of this repo. A copy of this file lives in the  [switch](https://github.com/p4lang/p4factory/tree/master/targets/switch) target of p4factory, where Openflow integration is already done. But you will probably have to edit it slightly for other targets. In particular, packet-in and packet-out must be enabled by uncommenting the `OPENFLOW_PACKET_IN_OUT` macro, which protects fabric header definitions and related parsers/actions.

`openflow.p4` defines two "public" actions, `openflow_apply` and `openflow_miss`. These must be included in an Openflow table's action set. The `process_ofpat_ingress` control block must be called from within the target's ingress control flow, and the `process_ofpat_egress` control block must be called from within the target's egress control flow. Finally, the `packet_out` table should be applied appropriately in the ingress control flow with a definition of the `nop` action to take on a miss, but this is subject to programmer discretion.

Note that to enable features such as VLAN and MPLS, the header declarations must be added if not present, and modifications to the source program's parser must to be made to unpack these headers.

#### The Mapping File
When adding Openflow to a target, you must write a mapping file that tells the behavioral model which P4 tables to expose as Openflow tables. This keeps the P4 Openflow Agent P4-independent. The mapping file enumerates the tables to expose as Openflow tables and each tables' match fields, along with their corresponding Openflow types per the Openflow 1.3 specification.

#### Final steps
Edit the target Makefile to build and link this library, then add a call to `p4ofagent_init` (defined in `p4ofagent.c`) to the `main.c` that is starting the behavioral model. This will start the P4 Openflow Agent as a thread in the behavioral model. Then you're done! Whew!

#### Examples
Look to the [l2_switch](https://github.com/p4lang/p4factory/tree/master/targets/l2_switch) p4factory target as an example of the above.

