# karat-files

Tento repozitář obsahuje jednoduchý statický web pro distribuci dvou instalačních souborů:

- `KaratAssitenceSupport.exe`
- `KRA.msi`

Nejde o aplikační zdrojové kódy produktu KARAT. Je to pouze malý download rozcestník a sada statických souborů, které se nasazují do Azure Static Web Apps.

## Proč to existuje

Účel repozitáře je:

- nabídnout uživatelům přehledný webový rozcestník pro stažení podpory a instalace,
- obsloužit několik veřejných domén nad stejným obsahem,
- umět z root URL podle hostname automaticky přesměrovat uživatele přímo na správný soubor.

## Co je v repozitáři

Hlavní soubory:

- `index.html` - hlavní rozcestník se čtyřmi kartami a hostname-based redirect logikou
- `404.html` - fallback stránka s okamžitým přesměrováním na odpovídající soubor
- `error.html` - stejný fallback jako `404.html`
- `staticwebapp.config.json` - konfigurace Azure Static Web Apps
- `KaratAssitenceSupport.exe` - binárka pro TV podporu
- `KRA.msi` - instalační balíček KRA
- `karat.svg`, `karat.ico`, `karat.bmp` - grafické assety

CI/CD workflow:

- `.github/workflows/azure-static-web-apps-agreeable-bay-0c8c11603.yml`
- `.github/workflows/azure-static-web-apps-nice-plant-03cba3603.yml`

## Jak to funguje

Web je čisté HTML/CSS/JS bez buildu.

Na root URL (`/` nebo `/index.html`) probíhá jednoduché přesměrování podle hostname:

- `tv.karatsoftware.cz` -> `/KaratAssitenceSupport.exe`
- `tv.karat.cz` -> `/KaratAssitenceSupport.exe`
- `kra.karatsoftware.cz` -> `/KRA.msi`
- `kra.karat.cz` -> `/KRA.msi`

Pokud uživatel přijde na jinou cestu, Azure Static Web Apps použije fallback stránku přes `404.html`.

## Kam je to napojené

K 12. 4. 2026 jsou nasazené dvě Azure Static Web Apps:

### 1. `karat-files`

- účel: TV domény
- Azure default hostname: `agreeable-bay-0c8c11603.1.azurestaticapps.net`
- navázané domény:
  - `tv.karatsoftware.cz`
  - `tv.karat.cz`

### 2. `karat-files2`

- účel: KRA domény
- Azure default hostname: `nice-plant-03cba3603.2.azurestaticapps.net`
- navázané domény:
  - `kra.karatsoftware.cz`
  - `kra.karat.cz`

## DNS

DNS není spravované v Azure.

Domény `karat.cz` a `karatsoftware.cz` jsou vedené mimo Azure a na Azure Static Web Apps míří přes `CNAME`.

Logické mapování je:

- TV hostnames -> `karat-files`
- KRA hostnames -> `karat-files2`

## Nasazení

Nasazení probíhá přes GitHub Actions po pushi do `main`.

Používají se dva samostatné workflow soubory:

- první workflow nasazuje stejný obsah do `karat-files`,
- druhý workflow nasazuje stejný obsah do `karat-files2`.

Oba deploye jsou záměrně nad stejným repozitářem a stejným obsahem. Rozdíl mezi chováním jednotlivých domén se řeší až v `index.html` a fallback stránkách podle aktuálního hostname.

## Co v repozitáři není

V repozitáři nejsou uložené:

- deployment tokeny,
- GitHub Actions secrets,
- žádné přístupové údaje k Azure,
- žádné DNS přihlašovací údaje.

Citlivé hodnoty patří pouze do GitHub Actions secrets nebo do odpovídající správy mimo repozitář.

## Provozní poznámka

Pokud se udělá rerun staršího GitHub Actions workflow, může přepsat novější deploy starší verzí obsahu. Pokud je potřeba oba weby znovu srovnat na stejnou verzi, nejjistější je udělat nový push do `main`, aby obě SWA nasadily stejný poslední commit.
