# Governance

`cloud-itonami-isic-1629` is an OSS open-business blueprint for wood/cork/straw-products-shop plant operations coordination.

## Maintainers
Maintainers may merge changes that preserve these invariants:
- a cutting/molding/weaving-equipment action the governor refuses is never dispatched to hardware.
- the Wood, Cork & Straw Products Shop Plant Operations Governor remains independent of the advisor.
- hard policy violations (equipment-control bypass, cutting/molding/weaving-line-run finalization, record-suppression, unauthorized disclosure) cannot be overridden by human approval.
- every schedule, sign-off, record and disclose path is auditable.
- sensitive operating and personal data stays outside Git.

## Decision Records
Architecture decisions live in `docs/adr/`. Changes to the trust model, storage contract, public business model, operator certification or license should add or update an ADR.

## Operator Governance
Anyone may fork and operate independently. itonami.cloud certification is a separate trust mark and should require safety, audit and data-flow review.

Certified operators can lose certification for:
- bypassing cutting/molding/weaving-equipment-control or record policy checks
- mishandling sensitive data
- misrepresenting certification status
- failing to respond to security or safety incidents
