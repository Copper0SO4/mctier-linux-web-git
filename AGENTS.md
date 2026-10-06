# Arch package maintenance

Read README.md and HANDOFF.md before changing the package. Keep this repository focused on Arch packaging; application changes belong in Copper0SO4/MCTier_Linux_Web.

Update HANDOFF.md for every maintenance change and README.md when installation, launch, dependencies or behavior changes. Regenerate .SRCINFO with makepkg --printsrcinfo whenever PKGBUILD changes; update checksums when local source files change.

The installed command must remain mctier. Preserve the checked bundled EasyTier core/CLI, ordinary-user execution, loopback-only local service, and existing interactive capability authorization. Never add automatic service startup, firewall modification or authorization to pacman hooks. Do not strip bundled EasyTier or bypass its hash checks.

Build/package checks do not authorize installing to the system, invoking pkexec or joining real rooms. Ask the user before such actions unless already explicitly authorized. Record build/package results separately from installation and real-environment validation.
