# Secure Private Cloud Storage on Azure — OwnCloud Deployment

A 2-tier secure architecture on Microsoft Azure, built manually through the Azure Console to replace an insecure Dropbox-based file-sharing workflow with a self-hosted, access-controlled alternative — deployed as part of Great Learning's PGP in Cloud Computing program.

> This was a hands-on console-driven lab (no Terraform/IaC) — the goal was to build fluency directly with Azure networking and VM primitives, as a complement to my Infrastructure-as-Code projects.

---

## The Problem

The scenario: a company's employees were using Dropbox to share files internally and externally. According to the brief, 40–75% of employees at similar businesses use unsanctioned file-sharing tools like this, and roughly half of Dropbox users do so despite knowing it violates policy. The average cost of a resulting data breach: **$5.5 million**. The task was to design and deploy a private, access-controlled alternative that keeps sensitive company and customer data inside infrastructure the company actually controls.

---

## Architecture

**2-tier design: Application tier (public) + Database tier (private)**

| Layer | Component |
|---|---|
| Virtual Network | P1VNET — 10.0.0.0/16 |
| Public Subnet | 10.0.1.0/24 — hosts the Application Server |
| Private Subnet | 10.0.2.0/24 — hosts the Database Server, no public IP |
| Application Server | Ubuntu 22.04 VM, running Apache + OwnCloud |
| Database Server | Ubuntu 22.04 VM, running MySQL, **no public IP** |
| NAT Gateway | Lets the private subnet reach the internet (e.g. for package installs) without being reachable from it |
| Network Security Groups | `AppNSG` (ports 22, 80) and `DbNSG` (ports 22, 3306 — internal-only) |

**Why this design:**
- The database server has **no public IP at all** — it can only be reached from within the virtual network, closing off the most common attack path (direct internet access to a database).
- The NAT Gateway gives the private subnet outbound-only internet access, so the database server can still pull software updates without being directly exposed.
- Two separate Network Security Groups scope exactly which ports are reachable and from where, rather than relying on a single shared security boundary.
- OwnCloud itself replaces Dropbox with a private, self-hosted file-sharing platform — the company's data never leaves infrastructure it controls.

---

## What Was Built

1. **Virtual network and subnetting** — created P1VNET with segmented public and private address spaces.
2. **NAT Gateway** — provisioned and attached to the private subnet for controlled outbound access.
3. **Network Security Groups** — `AppNSG` and `DbNSG`, each scoped to only the ports each tier actually needs.
4. **Application Server** — Ubuntu 22.04 VM in the public subnet; installed OwnCloud and its PHP/Apache dependencies via SSH.
5. **Database Server** — Ubuntu 22.04 VM in the private subnet; installed and configured MySQL via a provided install script, reached only through the Application Server as a jump point.
6. **Verification** — confirmed the deployment by accessing OwnCloud's setup page over the Application Server's public IP in a browser.

---

## Verified Proof

**Virtual Network provisioned (P1VNET, East US)**
![Virtual Network](./screenshots/virtual-network.png)

**Public and private subnets segmented**
![Subnets](./screenshots/subnets.png)

**NAT Gateway attached to the private subnet**
![NAT Gateway](./screenshots/nat-gateway.png)

**AppNSG — SSH and HTTP only**
![AppNSG rules](./screenshots/appnsg-rules.png)

**DbNSG — SSH and MySQL only, no public exposure**
![DbNSG rules](./screenshots/dbnsg-rules.png)

**Application Server — public IP, in the public subnet**
![Application Server](./screenshots/application-server.png)

**Database Server — no public IP, in the private subnet**
![Database Server](./screenshots/database-server.png)

**MySQL installed on the database server via SSH**
![MySQL installation](./screenshots/mysql-installation.png)

**OwnCloud downloaded, extracted, and permissions configured**
![OwnCloud installation](./screenshots/owncloud-installation.png)

**OwnCloud setup page live in the browser, via the Application Server's public IP**
![OwnCloud live](./screenshots/owncloud-live.png)

---

