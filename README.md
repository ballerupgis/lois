# Mit Hood
Simpel webapp til at lave socioøkonomiske rapporter for et område som tegnes på et kort. Oplysninger i rapporten er blandt andet boligtype og -størrelse, aldersfordeling, familietyper, virksomheder, statsborgerskab osv. Løsningen er afhængig af LIFA's API til [LOIS Statistik](https://www.lifa.dk/statistik-analyse/lois-statistik/).

![image](https://user-images.githubusercontent.com/7534153/33887867-88cf99d6-df4b-11e7-9773-f07d86968d1b.png)

For at rapporterne kan dannes korrekt ar det afgørende at kolonnenavne i MSSQL-tabellen har det rigtige format. For hver variabel er der fire tilhørende værdier:
1. Summeret antal for kommune
2. Summeret antal for niveau (det tegnede område)
3. Procentvis andel for kommune
4. Procentvis andel fot niveau 

Titlerne for de fire variabelværdier kan angives i tabellen ```OversigtKolonner```, hvor kolonnerne skal gives et passende alias, som skal have følgende format for at rapporterne dannes korrekt:

1. Antal Etagebolig
2. Antal Etagebolig Niveau
3. Pct Etagebolig
4. Pct Etagebolig Niveau

Det er afgørende for formateringen, at der står ```Pct``` eller ```Antal``` som det første ord, og at der står ```Niveau``` som sidste ord ud fra dem som omhandler det tegnede område. Det skal bruges mellemrum til at adskille ord og ikke andre tegn såsom underscore, da logikken in app'en bruger mellemrum til at splitte de forskkelige ord op og opbygge rapporten. Hvad der ellers står i Alias bliver printet ud i rapporten og den vil også medtage titler med mellemrum. Herunder ses eksempel på tabellen ```OversigtKolonner``` hvor der under ```Kolonneforklaring``` skal stå det korrekte alias.

![image](https://user-images.githubusercontent.com/7534153/37901443-8625c0c8-30f1-11e8-9e3c-a7f0b058e3ab.png)

Der kan ligeledes angives flot titel og forklarende tekst til de forskellige oversigter i tabellen ```OversigtProcedureTabel```. Titel angives i kolonnen ```LangtNavn``` mens beskrivelsen angives i ```Notat```. 

![image](https://user-images.githubusercontent.com/7534153/37902099-a66f8fb0-30f3-11e8-8f2f-14cde657de17.png)

Som det er lige nu kræver løsningen ikke login så længe man er indenfor firewall og kan tilgå API'et. App'en kunne udbygges til at understøtte NTLM authorization så der køres på organistionens windowsbrugerstyring.

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
