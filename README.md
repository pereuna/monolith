# Monolith

Plan2001:n sovellukset ja jakelu. Plan2001 on kernel-repo, 9frontin
UEFI-fork kolmelle raudalle. Monolith erotettiin siitä 16.10.2026
historioineen (Plan2001:n `docs/repos.md`).

| Osa | Polku |
|---|---|
| 9frontin julkaisukopio (osajoukko) ja USB-asennin | `subset/`, `tools/subset/`, `plan2001/rc/bin/inst`, `docs/install-subset.md` |
| Palvelut: rc-httpd:n sivusto, laskentapooli (crsrv, rcc), glendan profiili | `plan2001/rc/bin/rc-httpd`, `plan2001/sys/src/cmd`, `plan2001/usr` |
| CPU-VM ja Plan 9 -pilvipalvelin (historiaa: pilvi siirtyy OpenBSD:hen) | `tools/vm-cpu`, `tools/cpu-live`, `tools/cloud`, `tools/cloud-image`, `docs/cloud.md` |
| drawterm, 9pterm, rakennus- ja testityökalut | `monolith/` |
| wasm32-kone: ydin, sivu ja testit, 3a/3c/3l, kirjastot | `plan2001/sys/src/9/wasm32`, `plan2001/sys/src/cmd/3[acl]`, `plan2001/sys/src/lib*/wasm32`, `plan2001/wasm32` |
| Selainpalvelut: webterm, aux/wsrcpu, signupd, passkeyd, weblimitd, auth/passkey, aux/seckeys, websession, sovellusten originit | `plan2001/sys/src/cmd`, `plan2001/rc/bin`, `plan2001/lib/app` |

Polut ovat samat kuin Plan2001:ssä, jotta historia (`git log --follow`)
ja skriptien viittaukset säilyvät. Selaimen 9front-yhteensopiva Plan 9 on
[Plan9-wasm32](https://github.com/pereuna/plan9-wasm32). Tämän repon
wasm32-ominaisuudet on tarkoitus siirtää sinne.

## Riippuvuus Plan2001:een

Plan2001:n checkout (`PLAN2001`, oletus `~/Plan2001`) antaa:
- 9front-VM:n työkaluineen ja `build/`ineen;
- kernelin ja loaderin (`build/amd64`);
- BootInfon (`bootinfo.h`, `port/bootinfo.c`, `bootargs.c`) ja hjfs:n
  `auth.c`:n.

```
cd monolith
tools/build-libc3      # wasm32-kirjastot
tools/build-9wasm32    # ydin: build/wasm32/kernel/9wasm32.wasm
tools/build-bin3       # ohjelmat ja juuri: build/wasm32/root, root.fs
tools/test-3c          # kääntäjä
tools/test-9wasm32     # selaintestit (headless Chromium), ks. AGENTS.md
tools/pages            # sivu: build/pages
tools/build 9pterm     # 9pterm: build/9pterm/9pterm (myös Plan2001:n tools/9pterm)
```

Testaus ja säännöt: `AGENTS.md`.
