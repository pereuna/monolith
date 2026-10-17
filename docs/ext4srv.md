# ext4srv: ext4 käyttäjätilassa (17.10.2026)

Plan2001:n kernelissä oli ext4 (`#E`, lwext4) 16.–17.10.2026. Se
poistettiin, ja kernelin ainoa juuri on nyt gefs (Plan2001:n
`docs/gefs.md`). ext4 palaa käyttäjätilan palvelimeksi kuten 9frontissa:
9frontin `ext4srv` (`sys/src/cmd/ext4srv`), jolla Plan2001 lukee ja
kirjoittaa Linuxin ext4-osioita (esim. sillin `/home`).

## Lähteet

- 9front-11952:n `sys/src/cmd/ext4srv` alijoukossa (`tools/subset/extra`,
  kaikki sen tiedostot).
- `plan2001/sys/src/cmd/ext4srv/{ext4.c,ext4_fs.c,ext4_dir.c}`: samat
  tiedostot kokonaisina, mukana Plan2001:n kernelin lwext4:n korjaukset
  (Plan2001:n `sys/src/9/ext4` ennen commitia `df016ef`, ks. sen
  `docs/ext4.md`). Kernelisovituksia ei ole mukana (virhemerkkijonot,
  `ext4_config.h`).

## Korjaukset

1. `ext4_fread`: reiät ja kirjoittamattomat extentit luetaan nollina.
   lwext4 luki niiden kohdalle laitteen lohkon 0, joten esimerkiksi
   `mkfs.ext4 -d`:n tekemät binäärit (nollalohkot reikinä) luettiin väärin.
2. `metadata_csum_seed` (Debianin mkfs.ext4:n oletus): tarkistussummien
   siemen on superlohkon `s_checksum_seed`, ei UUID:stä laskettu.
3. Kirjoittava mount: tila merkitään kuten Linuxissa, eli `VALID_FS`
   pois mountin ajaksi ja takaisin umountissa. lwext4 kirjoitti
   `ERROR_FS`:n, jonka Linux ja e2fsck tulkitsevat löydetyiksi virheiksi.
   Löydetyt virheet säilyvät umountissa.
4. Hakemistonimen pituus rajataan `de.name`en: revision 0:n 16-bittinen
   pituus olisi kopioinut puskurin ohi.
5. Inodejen, ryhmäkuvaajien ja hakemistolehtien tarkistussummavirheet
   ovat virheitä, eivät näkymättömiä varoituksia, kun lohkoon ei ole
   muita viittauksia (lwext4 laskee summan vasta put:ssa).
6. `ext4_fwrite` merkitsee inoden likaiseksi, kun vain mtime muuttuu.
   Ilman sitä uusi mtime jäi välimuistiin vanhan tarkistussumman alle.

Kernelin ajurin omat korjaukset (yli 64 KiB:n write paloina, levyn
välimuistin flush, poistetun inoden Xfilet) koskivat devext4:ää, eivät
lwext4:ää. ext4srv:n 9P-palvelimessa ne ovat eri tavalla.

## Kuhalla (ei sillillä)

1. Alijoukko: `tools/subset/derive` (Plan2001:n VM), sitten
   `python3 tools/subset/make.py build/subset/amd64 .` ja
   `tools/subset/check` (`docs/install-subset.md`).
2. Käännös: `tools/tree`, ja `sys/src/cmd/ext4srv`:ssä `mk`.
3. Testi Linuxin ext4-kuvalla:
   - `mkfs.ext4 -d` puulla, jossa on harva tiedosto ja binäärejä:
     tiedostojen md5 Plan 9:ssä sama kuin Linuxissa (korjaus 1);
   - Debianin oletus-`mkfs.ext4` (`metadata_csum_seed`): mount onnistuu
     ilman tarkistussummavirheitä (2);
   - kirjoitus (luonti, `touch`, poisto), umount ja Linuxin
     `e2fsck -fn` puhdas (3, 6).
