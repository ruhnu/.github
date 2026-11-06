# Ruhnu

Experimental work on **revocable, audited access to secrets stored in Git** — including
secrets in commits already written — using the SSH keys engineers already have.

Built on [CDOC2](https://open-eid.github.io/CDOC2/2.0), the Estonian encrypted document
format. CDOC2's key-shares scheme splits key material across several servers, so that no
single server can decrypt anything on its own. An ssh-agent can sign but not decrypt — the
same constraint the scheme was designed around for Smart-ID — so the scheme carries over
with the signature coming from an SSH key instead.

## Why Ruhnu

Ruhnu is Estonia's most remote inhabited island. Key shares are held on separate servers,
each fragment useless by itself. One island is not the archipelago.

## What lives here

- Design documents and probe experiments for the SSH work
- Forks of the upstream CDOC2 repositories, where changes are prepared for review

## Status

Early. Design and experiments, not a working system — treat anything here as a draft unless
it says otherwise.

**Not affiliated with RIA or the open-eid project**, and not an official CDOC2 component.
Upstream lives at [open-eid](https://github.com/open-eid).
