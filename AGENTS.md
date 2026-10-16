# Ohje koodiagenteille (Codex, Claude): testaus ennen pushia

Monolith on Plan2001:n sovellukset ja jakelu omana repona (erotettu
16.10.2026 Plan2001:n commitin `f399452` jälkeen, historia mukana):
- 9frontin julkaisukopio (`subset/9front`) ja USB-asennin;
- palvelut: rc-httpd:n sivusto, laskentapooli (crsrv, rcc), CPU-VM
  (`tools/vm-cpu`) ja Plan 9 -pilvipalvelimen työkalut;
- drawterm ja 9pterm;
- Plan2001:n entinen wasm32-kone (ydin, 3c/3l, kirjastot, testit) ja
  selainpalvelut (webterm, signupd, passkeyd, weblimitd, websession,
  sovellusten originit).

Lue ensin `README.md`, sitten `docs/plan9-wasm32.md` ja
`docs/architecture.md` (historiaa Plan2001:n ajalta).

## Suhde Plan2001:een (kernel-repo)

Plan2001 (`PLAN2001`, oletus `~/Plan2001`) on 9frontin UEFI-fork
kolmelle raudalle: kernel, loader ja niiden käännös. Tämä repo käyttää
siitä:
- VM-työkaluja ja niiden `build/`ia (`tools/vm`, `9run`, `target.sh`,
  `inst.dialog`, `hw`): CPU-VM, osajoukon johto, USB-tikku ja pilvi-image;
- kerneliä ja loaderia (`build/amd64`, Plan2001:n `tools/build.sh`);
- BootInfoa (`plan2001/sys/include/bootinfo.h`, `port/bootinfo.c`,
  `bootargs.c`) ja hjfs:n `auth.c`:tä, jotka ovat kernelin boot-imagen.

Kernelin muutokset tehdään Plan2001:ssä, eivät täällä. Plan9-wasm32
(pereuna/plan9-wasm32) on selaimen 9front-yhteensopiva Plan 9. Tämän repon
wasm32-ominaisuudet on tarkoitus siirtää sinne. Oraclen pilveen asennetaan
toistaiseksi vakio-OpenBSD tai -Linux (päätös 16.10.2026). Sen työkalut
tehdään tänne; Plan 9 -pilven `tools/cloud` ja `cloud-image` ovat historiaa.

## Puu

- **`plan2001/`** on tämän repon puu 9frontin juuren muodossa
  (`sys/`, `rc/`, `lib/`, `usr/`, `wasm32/`): uudet tiedostot ja ne
  9frontin tiedostot, joita Plan2001 muuttaa, kokonaisina samassa
  polussa. Diffejä ei pidetä: muutos 9frontin tiedostoon on sen kopio
  `plan2001/`:ssa, ja lähde kirjataan commitiin.
- **`subset/9front/`** on 9frontin julkaisu sellaisenaan (alla), lähde,
  josta kopioidaan.
- `tools/tree` kokoaa näistä puun (`build/tree`: julkaisu, Plan2001:n
  kernelpuu ja tämän repon `plan2001/` sen päällä) natiivia käännöstä
  varten.
- wasm32-kone pitää toistaiseksi 9frontin `kbdfs`:n ja `devcons`:n:
  Plan2001:n versiot ovat pc64:n konsolin (rivieditori ESC-
  sekvensseineen, joita sivun konsoli ei vielä tulkitse). Natiivi
  käännös (`tools/native-wasm32`) ja `build-bin3`/`build-9wasm32`
  käyttävät niissä julkaisun tiedostoja.

## Säännöt, joita ei rikota

- **`subset/9front/` on 9frontin julkaisu sellaisenaan** (`tools/9front.release`),
  generoitu. Älä kopioi sinne tiedostoja käsin äläkä muokkaa niitä:
  `tools/subset/make.py` poistaa listaamattomat ja `tools/subset/check`
  hylkää muutetut. Uusi 9front-ohjelma tai -tiedosto: polku ja perustelu
  `tools/subset/extra`an, sitten VM:ssä `tools/subset/derive`,
  `python3 tools/subset/make.py build/subset/amd64 .` ja `tools/subset/check`
  (ks. `docs/install-subset.md`; VM on Plan2001:n). Jos VM:ää ei ole, kerro se commitissa:
  käyttäjä ajaa derivoinnin.
- **Muutos 9frontin tiedostoon** on kopio `plan2001/`:ssa saman polun
  alla, ja commitissa kerrotaan lähde. Kernelin tiedostot muutetaan
  Plan2001:ssä. Jos muutos on bugikorjaus, joka koskee myös
  Plan9-wasm32:ta, tarjoa se sinne.
- **Vitsit ja sitaatit eivät tule mukaan**: `DENY` tiedostossa
  `tools/subset/derive.py`. Pelit tulevat.
- Tee työ omassa haarassasi; älä pushaa toisen haaraan.

## Ympäristö

- Linux, gcc/clang, Python 3 (`cryptography`-paketti), Node 22.
- Testit ajavat headless-Chromiumia komennolla `chromium`. Jos sitä ei ole
  PATHissa, tee symlinkki (esim. Playwrightin):
  `mkdir -p ~/bin && ln -sf /opt/pw-browsers/chromium-*/chrome-linux/chrome ~/bin/chromium && export PATH=~/bin:$PATH`
- VM-testit tarvitsevat Plan2001:n 9front-VM:n (`$PLAN2001/tools/vm`, QEMU, `~/.cache/plan2001`).
  Ilman sitä ne ohitetaan; sano silloin commitissa, mitä jäi ajamatta.

## Rakennus (monolith/)

Selaimessa on vain wasm32-kone (D7): drawtermin selainosat, host3 ja
Monolithin JS on poistettu. `monolith/tools/pages` kokoaa sivun, jonka
CPU-palvelimen rc-httpd tarjoaa (`tools/vm-cpu --web`, `tools/cpu-live.rc
pages`). `tools/build` rakentaa enää vain 9ptermin (pilven hallinta) ja
natiivin drawtermin.

```
cd monolith
tools/build-libc3      # wasm32-kirjastot (libc, libthread, libsec, libauthsrv ...)
tools/build-9wasm32    # ydin: build/wasm32/kernel/9wasm32.wasm
tools/build-bin3       # ohjelmat ja juuri: build/wasm32/root, root.fs
```

`build-bin3` kertoo lopuksi ohjelmien määrän; jokainen virhe tulostuu
`build-bin3: NIMI:` -rivinä ja skriptin paluuarvo on silloin nollasta
poikkeava. Uusi ohjelma lisätään `build-bin3`:een (yhden tiedoston
komennot listaan `for c in ...`), sen kirjastot `build-libc3`:een.

Natiivi käännös: `tools/tree` kokoaa puun `build/tree`:hen (Plan2001:n kernelpuu mukana).
`PLAN9=plan9port tools/native-wasm32` kääntää siinä mkfileillä wasm32:n
kirjastot, ytimen ja juuren ohjelmat. `tools/test-native [NIMI...]` ajaa
`test-9wasm32`:n natiivilla ytimellä ja ohjelmilla. Aja se, kun muutat
mkfilejä, 9frontin tiedostojen kopioita tai 3c:n/3l:n objektimuotoa.
Uusi juuren ohjelma lisätään myös `native-wasm32`:n listaan. `tools/rccheck` tarvitsee niin ikään
`PLAN9`:n (plan9portin rc).

## Testit ennen pushia

Aja aina, kun muutos koskee ydintä, platform.js:ää, kirjastoja tai
juuren ohjelmia:

```
cd monolith
tools/test-9wasm32                 # kaikki (noin 25 min)
tools/test-9wasm32 NIMI [NIMI...]  # vain nämä, esim. rcpu tls notekill
```

Viimeinen rivi `test-9wasm32: ok` = kaikki ajetut menivät läpi. Joka testi
tulostaa `PASS` tai `FAIL`; FAIL:n tuloste on `build/wasmapp-kernel.txt`,
sivun loki `build/wasmapp-kernel.log` ja kuvakaappaus
`build/wasmapp-kernel.png`.

Testien ryhmät (`tools/test-9wasm32`in alussa tarkemmin):
- ydin ja ohjelmat: `c2a bootinfo badblob oldheader disk diskdead echo long fork forkloop rc rcif rci c3a failfork rfmem
  threads preempt rfmemloop failhelper failrfmem init fault procargs`
- ruutu, näppäimistö, hiiri, rio: `clock keys mouse hello dclock bytes rc3
  rci3 rioclock riorc boot paste ctrlv` (`boot` tarvitsee vga-alifontit, ks. alla)
- D3 tunnistus: `crypto dp9ik dp9iknoas secstore seckeys pagesecstore`
  (+ `dp9ikvm` ja `secstorevm` VM:llä, `secstorevm` vaatii `tools/vm-cpu
  --update`:n jälkeisen secstore-tilin; `secstore`: factotumin avaimet `test-authsrv`in secstoresta,
  `seckeys`: niiden salattu kopio levyllä ja haku siitä ilman verkkoa)
- D4 rcpu: `tls notekill netre nettimeout rcpu rcpuask rcpukeys` (+ `rcpuvm`
  VM:llä; `rcpukeys`: cpu-istunnon factotum on päätteen)
- WebAuthn: `webauthn signup signupnopk passkeyoffline signupd passkeyc passkeyadd passkeyphone passkeynoprf pagelogin pagerp weblimit weblimitd` (+ `signupvm`
  VM:llä) (`#W`, virtuaalinen autentikaattori, `test-authsrv`in signupd ja
  passkeyd; `signupd` ja `passkeyc` ajavat oikeat C-daemonit koneessa,
  `weblimit` CPU-palvelimen `websession`-rajan, `weblimitd` sen
  prosessi- ja muistirajat, docs/webauthn.md)
- D7 sovelluksen origin: `app appresume sendq` (webterm:n `/rcpu`-istunto, katkos ja jatko, iso jatko-jono, sivun takaisinkytkentä)
- verkko VM:ää vastaan: `net netloop` (VM:llä)

Muut: `tools/rccheck` (rc-skriptien jäsennys plan9portin rc:llä, aja kun
muutat rc-skriptiä: `/boot`, rc-httpd, `tools/*.rc`; heittomerkki
viestissä, kuten `Plan2001's`, avaa lainauksen),
`tools/test-3c` (kääntäjä, nopea, aja kun 3c/3l muuttuu),
`tools/subset/check` ja `tools/subset/test` (VM: osajoukko ja asennus).

### Uusi testi

- Testiohjelma `plan2001/sys/src/9/wasm32/test/NIMI.c` (lisää `build-bin3`:n
  testilistaan), odotettu tuloste `plan2001/sys/src/9/wasm32/test/NIMI.out`,
  ajo `tools/test-9wasm32`iin (`want NIMI $names && run NIMI '[argv]'`).
- `.out` alkaa rivistä `ticks in it: ok` ja **sen rivinvaihto on `\r\n`**
  (ytimen konsoli), muut rivit `\n`. Python tekstitilassa kadottaa `\r`:n:
  kirjoita binäärinä tai `printf 'ticks in it: ok\r\n...'`.
- Tulosteen on oltava deterministinen: ei pid-numeroita (MASKPIDS hoitaa
  muodon `prog N:`), ei kilpailevia rivejä kahdesta prosessista.
- Kehotteeseen vastaava syöte: SEND-alkio `["kehote", "rivi\n"]`.
- Verkkoa tarvitseva testi ilman VM:ää: `tools/test-authsrv` (dp9ik-auth
  `/567`, rcpu-rele `/17019`↔`/17999`), `MONOLITH_WEBTERM=18099`.
- Tarkista, että testi oikeasti kaatuu ilman korjaustasi.

## Sudenkuopat

- `tools/test-9wasm32` ajaa `build-bin3`:n vain, jos juurta ei ole
  (`build/wasm32/root/bin/d2`). Kun poistat tai siirrät lähteitä, rakenna
  juuri puhtaalta ennen pushia: `rm -rf build/wasm32/root build/wasm32/o/cmd
  && tools/build-bin3`, ja sitten `tools/pages`. D7:n poisto jäi kerran
  kiinni vasta toisen buildissa (clock.c).
- **Älä muokkaa `tools/test-9wasm32`:ää sen ajon aikana** (bash lukee
  skriptiä ajon aikana: syntaksivirhe keskellä).
- Katkennut ajo voi jättää `tools/serve`n (portit 18092/18093) tai
  `test-authsrv`in (18099) henkiin, ja seuraava ajo puhuu vanhalle:
  `ps -eo pid,args | grep -E 'tools/serve|python3 - 127|test-authsrv'` ja
  lopeta ne. Älä käytä `pkill -f`:ää komentorivillä, joka sisältää saman
  merkkijonon (tappaa oman kuoresi).
- `boot`-testi tarvitsee rion fontin vga-alifontit
  (Plan2001:n `build/subset/amd64/src/lib/font/bit/vga/vga.*`, derive tuo ne VM:stä);
  ilman niitä rion ikkunat jäävät tyhjiksi ja testi kaatuu.
- Kaksi saman koneen prosessia, jotka matkivat asiakasta ja palvelinta,
  tarvitsevat erilliset noteryhmät ja nimiavaruudet (`rfork(RFNOTEG|RFNAMEG)`),
  kuten aux/listen antaa oikealle palvelulle.
- Selaimen koneen ytimellä on 64 Mt muistia (image ja keko). Juuren
  arkisto (`root.fs`), BootInfo ja kehyspuskuri ovat sen yläpuolella
  (`docs/boot-abi-wasm32.md`). Jos tulee `no memory for allocb`, etsi
  vuoto tai silmukka ennen kuin kasvatat muistia.
- Koneen levy (D6) on OPFS-tiedosto `sdW0`. Testiselaimella on aina
  tuore profiili, joten levy on tyhjä ja `/boot/disk` reamaa sen. Jos
  testi kirjoittaa glendan kotiin, se menee levylle. Saman originin
  toinen välilehti jää ilman levyä, koska OPFS:n sync handle on
  yksinomainen.
- Initin argv on BootInfon configissa `init=`-rivinä (tokenizen
  lainaus). Argumentissa ei siis voi olla rivinvaihtoa: platform.js
  hylkää sellaisen.
- Taustapalvelin (lib9p:n `postsrv`: mntgen, factotum, ramfs) pitää auki
  ne fd:t, joilla se käynnistettiin. Jos se käynnistetään putken
  kirjoittavan pään sisällä (`{... factotum ...} | sed`), sed ei saa
  EOF:ia, eikä testi pääty. Käynnistä palvelimet ennen putkea.
- Jos `~/.cache/plan2001/cpu.pass` (CPU-VM:n glendan salasana; ei
  pilven, sen salasanat ovat `cloud/`issa) on kadonnut, `tools/vm-cpu
  --repass` antaa uuden. Älä vaihda nvramin avainta (`auth/wrkey`)
  ilman `auth/convkeys /adm/keys`:ia: keyfs ei silloin enää avaa
  käyttäjiä.
- Pysyvä VM-levy (`cpu.qcow2`) suljetaan `fshalt`illa. Plan2001:n `tools/vm stop`
  tekee sen itse (`NOHALT=1` ohittaa). Pelkkä QEMU:n quit jätti cwfs:n
  kirjoittamatta, eikä levy enää mountannut.

## Commit

- Viesti kertoo mitä ja miksi, repon tyyliin (`git log`).
- Kerro, mitkä testit ajoit ja mitkä jäivät ajamatta (VM, rauta).
- `docs/architecture.md`:n vaiheen tila ajan tasalle, kun vaihe etenee.
- Kun otat koodia Plan2001:stä, Plan9-wasm32:sta tai 9frontista, kirjaa
  lähde (repo ja commit).
