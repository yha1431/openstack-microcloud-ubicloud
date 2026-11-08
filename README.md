# Open source private cloud: OpenStack, MicroCloud, or a bare-metal control plane? How to pick a route, what each costs, and the checks that save you a rebuild

Three routes cover nearly every open source private cloud running in production today: a full OpenStack and Ceph stack you either operate yourself or rent as a single-tenant hosted service, Canonical's three-component MicroCloud cluster, and control planes such as Ubicloud that turn bare metal into a cloud API. They differ less in feature lists than in who carries the control plane and how the bill arrives each month. What follows is the operational cost of each route, the three pricing structures that decide your year-two spend, and the hardware questions to settle before you deploy a single node.

> **Quick pick.** If you are pricing the compute layer first, Spin Servers publishes KVM plans from 2 vCPU / 2 GB up to 32 vCPU / 32 GB, with the 12 vCPU / 12 GB configuration listed from **$62.00/month**. That buys you a machine to run the cluster on; the cloud software stays your job.
> 👉 [See Spin Servers' current KVM VPS configurations and monthly rates](https://bit.ly/Spinservers)

## Spin Servers KVM plans at a glance

| Plan | vCPU / RAM | SSD | Bandwidth | Price | Order |
| --- | --- | --- | --- | --- | --- |
| Simple Application Server | 2 vCPU / 2 GB DDR4 | 40 GB RAID-10 | 4 TB/mo | Rate at checkout | [View plan](https://bit.ly/Spinservers) |
| Simple Application Server | 4 vCPU / 4 GB DDR4 | 80 GB RAID-10 | 8 TB/mo | Rate at checkout | [View plan](https://bit.ly/Spinservers) |
| Simple Application Server | 6 vCPU / 8 GB DDR4 | 120 GB RAID-10 | 8 TB/mo | Rate at checkout | [View plan](https://bit.ly/Spinservers) |
| Simple Application Server | 8 vCPU / 8 GB DDR4 | 160 GB RAID-10 | 10 TB/mo | Rate at checkout | [View plan](https://bit.ly/Spinservers) |
| KVM 12 vCPU 12 GB | 12 vCPU / 12 GB | 160 GB | 10 TB/mo | **From $62.00/mo** | [View plan](https://bit.ly/Spinservers) |
| Advanced Application Server (Xeon) | 16 vCPU / 16 GB DDR4 | 320 GB RAID-10 | 15 TB/mo | Rate at checkout | [View plan](https://bit.ly/Spinservers) |
| Advanced Application Server (Xeon) | 32 vCPU / 32 GB | 500 GB | 20 TB/mo | Rate at checkout | [View plan](https://bit.ly/Spinservers) |

Every plan above ships with a 1 Gbps network port, one IPv4 and one IPv6 address, and KVM virtualisation. Only the 12 vCPU / 12 GB rate was displayed when this was written; the other configurations each carry their own monthly figure on the same plan page, so check the number for the exact size you want before committing.

One detail that catches people running mail-adjacent services: **port 25 is closed by default on all VPS plans**. Opening it requires a ticket with a justification, and the provider reserves the right to refuse.

## The three routes, and the team each one fits

### Full stack: OpenStack plus Ceph

OpenStack is a modular collection of sub-projects built on open standards, assembled into a scalable private or hybrid cloud. Because workloads can move between a private cloud and a public one, it keeps vendor lock-in comparatively low. In practice a production build pairs it with Ceph for distributed block storage.

Canonical's Charmed OpenStack is the reference implementation for the self-run version. MAAS provisions bare metal, Juju models applications, and charms carry the deployment and configuration instructions for each service. Canonical says this compresses adoption to "just a few weeks" and automates day-two work such as cluster scale-out and database backups. On releases, the company says it supports every upstream OpenStack version within two weeks of release, ships an LTS every two years with five years of support, and offers up to ten years of security updates to some customers under an Ubuntu Advantage for Infrastructure subscription. Those numbers come from Canonical, not from independent benchmarking.

OpenMetal shows what the same stack looks like as a rented service: a Cloud Core cluster of three hyperconverged servers runs OpenStack for compute and networking alongside Ceph for storage, with three nodes providing quorum. You get the Horizon interface, the full API, root and IPMI access, NVMe storage, and software-defined networking with isolated tenant networks, floating IPs, VLAN and overlay networking, load balancing and routing, all on dedicated hardware at a fixed monthly price.

This route fits organisations that need a broad API surface and have someone who owns the platform. The upgrade path deserves the most scrutiny. Canonical's own material notes that OpenStack upgrades "are known to be painful due to the complexity of the process," which is why the release cadence and automation coverage matter more than the feature list at install time.

### Lightweight: MicroCloud

MicroCloud targets small private clouds specifically. It combines LXD for virtual machines and system containers, MicroCeph for distributed storage replicated across cluster members, and MicroOVN for software-defined networking, packaged so that cluster creation is largely automated. It is written in Go and released under AGPLv3 at no cost.

The capability list is longer than "small cloud" suggests: new machines join an existing cluster automatically, compatible VM workloads can live-migrate between members, LXD projects and access policies provide multi-tenancy, and you get a graphical interface, CLI and REST API. Preseed configuration files make deployments repeatable, dedicated networks can be carved out for Ceph management, cluster communication and OVN underlay traffic, and multi-site or edge layouts are supported.

Canonical positions it for edge computing, development and test environments, distributed offices and small private data centres, and states plainly that it is not aimed at organisations needing large-scale cloud complexity. The operational burden is three components instead of a dozen OpenStack services, and each component keeps its own management interface.

### Bare metal as a cloud API: Ubicloud

Ubicloud describes itself as an open source cloud that can run anywhere, with the Linux-to-proprietary-OS comparison as its framing. It runs on bare metal from providers such as Hetzner, Leaseweb and AWS Bare Metal, or as a managed service, and exposes Linux VMs on Cloud Hypervisor with public IPv4 and IPv6, private networking and encrypted disks; managed PostgreSQL with one or two standbys, read replicas, point-in-time restore, PgBouncer and log export; managed Kubernetes with node pools; x64 and arm64 GitHub Actions runners; AI inference endpoints for open-weight models; and private subnets, firewalls and load balancers. You drive it through a console, a REST API with an OpenAPI spec, a CLI, or Terraform.

The architecture is a Ruby control plane on Postgres that keeps every resource's target state in the database and advances it through state machines it calls progs, pushing changes to data-plane hosts over SSH. The VMM sits inside Linux namespaces, networking runs over IPsec tunnels with a separate network namespace per customer VM, firewalls and load balancing use nftables, and permissions use attribute-based access control.

The limit you need to plan around sits in the block storage layer: **Ubicloud's ubiblk block storage is explicitly non-replicated**. Redundancy has to come from a layer above it, whether that is database replication, application-level copies, or your own storage design. Its founders came from Azure, Amazon and Heroku, and include Citus Data's co-founder and founding members; the stated goal is to build "10% of the cloud services that make up 80% of that consumption," which is a vendor target, not a shipped roadmap.

On OpenStack itself the two vendors disagree, and neither position has third-party arbitration. Canonical describes OpenStack as one of the most featureful and versatile open source cloud platforms. Ubicloud says OpenStack "is still primarily used for 3 cloud services" and calls its habit of supporting many hypervisors, object stores and block backends "consultant-ware." Read both as positioning. CloudStack-based stacks and Kubernetes-native virtualisation projects cover the same territory from different angles, and are worth a look if neither of the above fits.

## Where the money goes after year one

**Support subscriptions are priced per CPU or per node.** VMware prices support and subscription by CPU; Charmed OpenStack prices it per node. On a node with a high core count, that difference compounds across every renewal. Canonical presents this as the reason its model lowers operating cost and adds predictability, which is a vendor claim you should test against your own core density.

**Fixed monthly versus metered billing.** OpenMetal sells single-tenant infrastructure at a fixed monthly rate with no metering traps and no egress fees, and frames the pitch directly against per-VM, per-IOPS and per-request pricing. Fixed pricing wins when your workloads are chatty on egress or steady on IOPS. Metered wins when they are mostly idle. Do the arithmetic on your own traffic before assuming either.

**Whether storage replicates.** Ceph replicates data across nodes, which is why OpenMetal can claim workloads survive a single hardware failure. Ubicloud's block storage does not, so copies are a separate design decision with its own cost. Ask any vendor whether replication is included or an add-on.

**Upgrade cadence as a budget line.** Each OpenStack release is a project. Canonical's two-week support window and two-year LTS rhythm exist to make long-lived deployments plannable; MicroCloud's preseed files and Ubicloud's state machines serve the same purpose in different ways. A platform that upgrades cleanly costs less in year three than one that does not, even at identical subscription prices.

## Five checks before you move off VMware

Canonical lists five considerations for anyone migrating to an open source private cloud, and they work as a general checklist regardless of which distribution you pick.

1. **Economics.** Model capital and operating cost together, including the per-CPU versus per-node question above.
2. **Technology support.** Confirm every component in your stack has a support path: the hypervisor, the storage layer, the networking plugins, the OS.
3. **Velocity of development and innovation.** How quickly do upstream features reach the version you run?
4. **Day-N operations.** What does the platform look like on day 200? Automation coverage is the honest measure: MAAS and Juju for OpenStack, preseed files for MicroCloud, progs for Ubicloud.
5. **Upgrades.** Is the release cadence predictable, and is there a tested path from your version to the next one?

OpenStack's ecosystem helps on the second point. Canonical notes support from mainstream IT vendors, hypervisors, public cloud and hosted private cloud providers, plus networking and storage vendor plugins, and argues this gives it broader technical options than VMware or other OpenStack distributions. Applications portable between private and public cloud are the practical payoff of the open-standards design.

## What the host has to provide

Every route above assumes a layer underneath: machines with predictable CPU, RAID-protected local disk, enough bandwidth to pull images and replicate storage, and out-of-band access for provisioning. Get that layer wrong and the cloud software cannot compensate.

Spin Servers publishes its KVM range with DDR4 memory, RAID-10 SSD storage, a 1 Gbps port, one IPv4 and one IPv6 address, and monthly bandwidth allowances between 4 TB and 20 TB depending on configuration. The published sizes run from 2 vCPU / 2 GB to 32 vCPU / 32 GB, with the 12 vCPU / 12 GB tier listed from $62.00/month.

Two capabilities reported by users matter for clustered deployments. A reviewer renting dedicated servers describes a VPC feature that puts separate servers on the same layer-2 network with 10G connectivity, which is what you want for Ceph replication or a cluster heartbeat that expects L2 adjacency, and notes IPMI access added to the client area. The same review reports the "instant servers" line auto-provisioning within the 30 minutes advertised.

👉 [Compare Spin Servers' KVM plans and current monthly pricing](https://bit.ly/Spinservers)

## How to tell an open source private cloud project is already dead

The AeroFS private cloud appliance repository is the cleanest cautionary example available. It is an ongoing restoration project: the last maintenance release of the code dates to early 2018, and the build process depended on internal infrastructure (VPN, AWS SES, an internal Maven repo, an internal Docker registry, an internal apt repo) that no longer exists.

The known-issues list reads like a checklist of everything that rots in eight years. The build needs JDK 8, because newer versions "have not been tested and will probably break," and protobuf 2.6.1, where newer versions will not work at all. Large amounts of Python 2.7 code await porting. The base Docker images are described as old and probably full of CVEs. Appliance releases depend on CoreOS, long discontinued. Certificates under the project's own domain have expired. There is no CI setup, no code-signing credentials for Windows or macOS, no Apple Silicon support, and Windows builds are stuck at 32-bit. The Java build depends on Gradle 2.14.

Turn that into six questions for any project you are considering:

- When was the last maintained release, and how many years ago is that?
- Does it build on a clean machine with current toolchains, or does it pin a decade-old compiler?
- Are the base images and dependencies current, and who tracks CVEs in them?
- Is there CI?
- Does it run on the hardware you actually own?
- Does it depend on the vendor's internal infrastructure or a discontinued base OS?

If two or three answers are bad, the licence being free will not save the project.

## FAQ

**How many people does it take to run an open source private cloud?**
Nobody publishes a headcount, and the honest answer varies by route. MicroCloud reduces the surface to three components with a GUI and API, which a single administrator can realistically own for a small site. OpenStack needs a named owner for the control plane even with MAAS, Juju and charms automating routine work, because upgrades and networking changes still land on a person. A hosted option such as OpenMetal shifts that work to the provider at a fixed monthly cost.

**Can you migrate from VMware?**
The workloads move; the operational model has to move with them. Canonical frames the migration around financial institutions and argues the economic case is strong, with the qualification that you must pick a distribution that is easily deployable, maintainable, upgradable and cost-effective. Test the upgrade path on a staging cluster before you commit production.

**Is a three-node cluster enough?**
Three is the practical floor for high availability. OpenMetal's reference architecture uses exactly three hyperconverged servers to provide quorum, and MicroCloud replicates data across cluster members. Below three nodes you lose quorum or failure tolerance, and Ceph generally prefers odd numbers for the same reason.

**Is an open source private cloud actually free?**
The software can be. MicroCloud is AGPLv3, and Ubicloud's code is open source. The hardware, the bandwidth, the people and the support subscription are not. Canonical sells support per node, OpenMetal charges a fixed monthly rate. Budget for the second and third items before you celebrate the first.

## Choosing by situation

- **One administrator, a small site, edge locations, or a test environment:** MicroCloud. Three components, AGPLv3, live migration and preseed automation, and Canonical is explicit that it is not built for hyperscale complexity.
- **A broad API surface with software-defined networking and distributed storage, plus a platform owner:** the OpenStack and Ceph route, self-run with an automated distribution or rented single-tenant as OpenMetal does it.
- **An AWS-shaped API on hardware you already control:** Ubicloud, with a redundancy plan for block storage written down before you migrate anything onto it.
- **You only need the machines:** price the KVM range, then confirm two things before ordering. The exact monthly rate for your configuration, and whether inter-node replication traffic counts against the monthly bandwidth allowance. Three-way Ceph replication means every write crosses the network more than once, and a 4 TB allowance behaves very differently from a 20 TB one under that load.

👉 [Check current Spin Servers KVM VPS pricing and configuration details](https://bit.ly/Spinservers)
