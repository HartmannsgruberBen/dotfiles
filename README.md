# dotfiles

Konfigurationsdateien verwaltet mit [dotter](https://github.com/SuperCuber/dotter).

## dotter
Dotter legt Symlinks von diesem Repo im Home-Verzeichnis an, damit die installierten Programme die Configs hier nutzen.

```text
~/dotfiles/                       ~/.config/
├── alacritty/  ────symlink───►   alacritty/
├── helix/      ────symlink───►   helix/
├── jj/         ────symlink───►   jj/
├── starship.toml ──symlink──►    starship.toml
└── .dotter/
    ├── global.toml      (alle Pakete und deren Ziele)
    └── local.toml       (welche Pakete auf DIESEM Rechner aktiv sind)
```

`global.toml` ist im Repo (für alle Rechner gleich), `local.toml` ist `.gitignore`d (pro Rechner unterschiedlich).

---

## Dotfiles übertragen und anwenden
1. Repo in das Home-Verzeichnis klonen
2. In das geklonte Verzeichnis wechseln
3. Nachfolgenden Befehl im Verzeichnis ausführen
   ```bash
   dotter deploy
   ```
4. Prüfen ob `~/.config/` symlinks in das dotfiles Verzeichnis aufweist 

---

## Dotfiles initiale Einrichtung
Frischer Rechner, dotter wurde hier noch nie verwendet und es existiert auch noch kein Repo.
Dieser Abschnitt beschreibt den kompletten Weg von 0: dotter installieren, leeres Repo mit `dotter init` aufsetzen, Configs einpflegen.

### 1. Dotfiles-Verzeichnis anlegen und initialisieren
```bash
mkdir -p ~/dotfiles
cd ~/dotfiles
jj init
dotter init
```

`dotter init` legt das `.dotter/`-Verzeichnis mit einer leeren `global.toml` und einer leeren `local.toml` an. Struktur danach:

```text
~/dotfiles/
└── .dotter/
    ├── global.toml      (noch leer — alle Pakete und Ziele kommen hier rein)
    └── local.toml       (noch leer — welche Pakete auf DIESEM Rechner aktiv sind)
```

`local.toml` direkt zu `.gitignore` hinzufügen, damit rechnerspezifische Auswahl nie ins Repo wandert:

```bash
echo ".dotter/local.toml" >> .gitignore
```

### 3. Erste Config einpflegen
Beispiel `starship`: bestehende Config ins Repo verschieben, dann via dotter zurück-symlinken.

```bash
# Datei aus ~/.config ins Repo verschieben
mv ~/.config/starship.toml ~/dotfiles/starship.toml
```

`.dotter/global.toml` öffnen und das Paket eintragen:

```toml
[starship]
depends = []

[starship.files]
"starship.toml" = "~/.config/starship.toml"
```

`.dotter/local.toml` öffnen und das Paket aktivieren:

```toml
packages = ["starship"]
```

Für weitere Configs siehe [Eine neue Config hinzufügen](#eine-neue-config-hinzufügen).

### 4. Deploy ausführen
```bash
dotter deploy
```

dotter legt jetzt die Symlinks an.
Bei Konflikten (z. B. `~/.config/helix` existiert bereits) bricht es ab.

```text
   ┌─────────────┐    dotter deploy    ┌──────────────────┐
   │  ~/dotfiles │  ─────────────────► │  ~/.config/*     │
   │  (Quelle)   │       Symlinks      │  (Ziele)         │
   └─────────────┘                     └──────────────────┘
```

### 5. Prüfen
```bash
ls -la ~/.config/starship.toml
# → sollte einen Pfeil "-> /home/<user>/dotfiles/starship.toml" zeigen
```

### 6. Repo veröffentlichen (optional)
Damit das Setup auf weiteren Rechnern via `git clone` ausgerollt werden kann:

```bash
jj describe -m "chore: initial dotter setup"
jj git push --remote origin
```

---

## Eine neue Config hinzufügen
Beispiel: `btop` soll dazukommen.

### 1. Config-Ordner anlegen
Den Ordner so anlegen, dass die Struktur darunter exakt der Struktur unter `~/.config/` entspricht.

```bash
mkdir -p ~/dotfiles/btop
cp -r ~/.config/btop/* ~/dotfiles/btop/
```

Danach existierende `~/.config/btop` entfernen (dotter wird gleich einen Symlink anlegen):

```bash
rm -rf ~/.config/btop
```

### 2. Eintrag in `global.toml`

`.dotter/global.toml` öffnen und einen neuen Block ergänzen. Schema:

```toml
[<paketname>]
depends = []

[<paketname>.files]
"<quelle-im-repo>" = "<ziel-im-home>"
```

Für `btop`:

```toml
# === System-Monitor ===
[btop]
depends = []

[btop.files]
"btop" = "~/.config/btop"
```

Für eine einzelne Datei (statt eines Ordners) sieht das so aus — siehe `starship.toml` im Repo:

```toml
[starship]
depends = []

[starship.files]
"starship.toml" = "~/.config/starship.toml"
```

Soll der Symlink explizit symbolisch sein (statt hardlink), wie bei `yazi`:

```toml
[yazi.files]
"yazi" = { target = "~/.config/yazi", type = "symbolic" }
```

### 3. Eintrag in `local.toml`

Das Paket nur auf den Rechnern aktivieren, die es nutzen sollen. In `.dotter/local.toml` den Namen zur `packages`-Liste hinzufügen:

```toml
packages = [
    "alacritty",
    "starship",
    # ...
    "btop",       # ← neu
]
```

Gleiches ggf. in `local.toml.desktop` und/oder `local.toml.server` ergänzen, damit zukünftige Setups das Paket bekommen.

### 4. Deploy

```bash
dotter deploy
```

Kontrolle:

```bash
ls -la ~/.config/btop
# → -> /home/<user>/dotfiles/btop
```

### 5. Ins Repo aufnehmen

```bash
jj describe -m "feat: add btop config"
jj git push
```
