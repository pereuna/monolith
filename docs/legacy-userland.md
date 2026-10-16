# Legacy pois: userland (vaihe 4, 7.10.2026)

Plan2001:n docs/legacy.md:n kohdat 4–7 (versio 3432a31), jotka siirtyivät
Monolithiin jaossa 16.10.2026 (Plan2001:n docs/repos.md). Ytimen kohdat
1–3 ovat Plan2001:n docs/legacy.md:ssä.

## 4. Tunnistus (factotum, authsrv, keyfs)

| Mikä | Suositus |
|---|---|
| `p9sk1` (DES-avaimet ja -liput) | **pois:** dp9ik korvaa sen. keyfs:n DES-avain (`key`) pois, `aeskey` jää. Vaatii authsrv:n ja keyfs:n kopion `plan2001/`:een ja `p9any`:n neuvottelun vain dp9ik:lle |
| `p9cr`, `vnc`, `apop`, `cram`, `chap`, `mschap`, `mschapv2`, `ntlm`, `ntlmv2`, `httpdigest` | **pois** (vanhat haaste-vastaus-protokollat; ei Plan2001:n palveluja) |
| `dp9ik`, `p9any`, `pass`, `rsa`, `ecdsa` (TLS, ACME), `totp` | **jää** (`totp` päätä: ehkä toinen tekijä) |
| secstore (PAK), WebAuthn (passkey, signupd, passkeyd) | **jää** |

## 5. Etäkäyttö ja verkkopalvelut

| Mikä | Suositus |
|---|---|
| vanha `cpu` ja `import` (`-a`, p9sk1-aikainen salaus) | **pois:** `rcpu`, `rimport` ja `rexport` (TLS-PSK) jäävät |
| `telnet`, `telnetd`, `rlogin`, `rx`/`rexexec`, `ftpfs`, `ftpd` | **pois** |
| `srv` ja `9fs` ilman TLS:ää | **päätä:** `rimport` ja `srv` TLS:n kanssa riittävät |
| `cifs`, `nfs`, `smbfs`, `vncs`/`vncv` | **pois** |
| natiivi drawterm (`monolith/tools/build`) | **päätä:** pääte on wasm32 ja webterm. 9pterm (pilven hallinta) jää |
| `upas` (posti) | **päätä:** CPU-palvelimen ei tarvitse lähettää postia (`websession` estää sen web-tunnuksilta jo) |
| `ip/ppp`, `ip/pppoe` | **pois** |
| DNS (`ndb/dns`), DHCP-asiakas, `ip/ipconfig`, ACME (`certrenew`), `rc-httpd`, `tlssrv` | **jää** |

## 6. Tiedostojärjestelmät

| Mikä | Suositus |
|---|---|
| `cwfs` | **jää** (palvelimen fs) |
| `gefs` | **päätä:** 9frontin uusi fs. Valitaan yksi palvelimelle (cwfs on nyt käytössä ja testattu) |
| `hjfs` | **jää** (wasm32-koneen levy) |
| `dossrv` (FAT) | **jää** (UEFI:n esp) |
| `paqfs` | **päätä** (kohta 2) |
| `9660srv`, `cdfs`, ISO-työkalut | **pois** palvelimelta (asennus ei käytä CD:tä) |
| `cfs` (välimuisti-fs) | **pois** |
| `disk/mbr`, `disk/fdisk` (MBR) | **pois** (GPT: `disk/edisk`, `prep` jäävät) |
| fossil, venti | eivät ole osajoukossa: **pois** |

## 7. Ohjelmat ja ympäristö

| Mikä | Suositus |
|---|---|
| APE (ANSI/POSIX-ympäristö) ja sen ohjelmat | **päätä:** porttaukseen hyödyllinen, mutta iso. Plan2001:n omat ohjelmat eivät tarvitse sitä |
| `mothra` (vanha selain), `page`/`gs` | **pois** palvelimelta. Terminaalilla on selain |
| `aux/mouse` (PS/2- ja sarjahiiren asetus) | **pois** |
| pelit | **jää** (päätös: pelit tulevat) |
| vitsit ja sitaatit | jo pois (`DENY`) |
| `sam`, `acme`, `rio`, `rc`, perustyökalut | **jää** (terminaalin ohjelmat; palvelimella sovellusoriginien `lib/app`) |

