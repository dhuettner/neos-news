# Waterproof.News

News- und Blog-Paket für **Neos CMS 9**. Artikel sind Dokument-Knoten, Listen entstehen über [Flowpack.Listable](https://github.com/Flowpack/Flowpack.Listable), Kategorien über [Sitegeist.Taxonomy](https://github.com/sitegeist/Sitegeist.Taxonomy).

Das Paket liefert **Struktur, kein Design**: Markup mit Tailwind-Klassen, keine eigene CSS-Datei und kein eigener Build. Gestaltet wird im Build des einbindenden Site-Packages.

## Installation

```bash
composer require waterproof/neos-news
./flow neos.flow:package:rescan
./flow flow:cache:flush --force
./flow resource:publish
```

Die drei Befehle nach dem Require sind unter DDEV Pflicht: Das Composer-Plugin schreibt den Paket-Cache nach `Data/Temporary/`, Flow liest ihn aber aus `/tmp/Flow/`. Ohne `package:rescan` bleibt das Paket unsichtbar.

### Tailwind einrichten — sonst bleibt alles ungestylt

Die Fusion-Dateien dieses Pakets liegen außerhalb deines Site-Packages. Tailwind findet Klassen nur in Dateien, die im `content`-Glob stehen. Ergänze in deiner `tailwind.config.js`:

```js
content: [
    './Resources/Private/**/*.{fusion,html,js}',
    './NodeTypes/**/*.{yaml,fusion}',
    '../../Packages/Application/Waterproof.News/Resources/Private/**/*.fusion',
],
```

Ohne diesen Eintrag ist das Paket funktionsfähig, aber ohne Layout.

## Node-Typen

| Node-Typ | Backend-Label | Zweck |
|---|---|---|
| `Waterproof.News:Document.Article` | Artikel | Einzelbeitrag mit Datum, Teasertext, Teaserbild, Autor, Inhaltsbereich |
| `Waterproof.News:Document.ArticleIndex` | Artikelübersicht | Elternseite, listet ihre Artikel paginiert |
| `Waterproof.News:Content.ArticleTeaser` | Artikel-Teaser | Zeigt die jüngsten Artikel auf beliebigen Seiten |
| `Waterproof.News:Document.Feed` | Feed | Atom- oder RSS-Ausgabe der Artikel |

Die Artikelübersicht lässt unterhalb ihrer selbst nur Artikel zu. Umgekehrt verhindert das nicht, dass ein Artikel an anderer Stelle angelegt wird — soll das ausgeschlossen sein, grenze es in deinem Site-Package über die Constraints deiner Basis-Seite ein.

## Spaltenwahl

`Document.ArticleIndex` und `Content.ArticleTeaser` besitzen die Eigenschaft `columns`:

| Wert | Label | Ergebnis |
|---|---|---|
| `cols1` | Einspaltig | eine Spalte auf allen Breiten |
| `cols2` | Zweispaltig | ab `md` zwei Spalten |
| `cols3` | Dreispaltig | ab `md` zwei, ab `lg` drei Spalten (Standard) |
| `cols4` | Vierspaltig | ab `sm` zwei, ab `lg` vier Spalten |

Die Klassen stehen in `Component/GridClass.fusion` ausgeschrieben. Zusammengesetzte Klassennamen wären für den Tailwind-Scanner unsichtbar.

## Eigenes Design einsetzen

Überschreibe in deinem Site-Package die Prototypen, die du gestalten willst:

```fusion
prototype(Waterproof.News:Document.Article.Short) < prototype(Neos.Fusion:Component) {
    renderer = afx`…dein Kartenlayout…`
}
```

| Prototyp | Zuständig für |
|---|---|
| `Waterproof.News:Document.Article.Short` | Kartendarstellung in Listen |
| `Waterproof.News:Content.ArticleIndexBody` | Listenkörper der Übersicht |
| `Waterproof.News:Content.ArticleBody` | Detailseite |
| `Waterproof.News:Component.ArticleCollection` | Raster um die Karten |
| `Waterproof.News:Component.GridClass` | Zuordnung Spaltenwert zu Klassen |

## Filter und Archiv

Die Übersicht wertet zwei Parameter aus:

| Parameter | Wirkung |
|---|---|
| `kategorie` | Knotenname der Taxonomie, etwa `?kategorie=abwasserbeseitigung` |
| `jahr` | vierstellig, etwa `?jahr=2026` |

Beide sind kombinierbar und bleiben beim Blättern erhalten. Die Filterleiste zeigt nur Kategorien, denen tatsächlich ein Artikel zugeordnet ist, und die im Bestand vorkommenden Jahre.

## Feed

Der Feed ist ein eigener Dokumentknoten unterhalb der Übersicht, Format wählbar zwischen Atom und RSS 2.0. Die Übersichtsseite verweist im Kopf per `<link rel="alternate">` darauf.

**Warum kein `/aktuelles.rss`:** Neos routet Dokumente mit genau einem site-weiten URI-Suffix, standardmäßig `.html`. Eine abweichende Endung ließe sich nur über die Site-Konfiguration erreichen und würde dann für alle Seiten gelten. Der Feed liegt deshalb unter `/aktuelles/feed.html` — mit korrektem `Content-Type`.

## Strukturierte Daten

Artikelseiten enthalten ein JSON-LD-Objekt vom Typ `Article` mit Titel, Datum, Beschreibung, kanonischer URL und Herausgeber. Der Autor erscheint nur, wenn er gepflegt ist.

## Sprachen

Backend-Labels liegen als XLIFF in `Resources/Private/Translations/` für Deutsch und Englisch vor. Weitere Sprachen: Katalog kopieren und übersetzen, die Node-Typen selbst brauchen keine Änderung.

## Kategorien

Artikel referenzieren Taxonomien über die Eigenschaft `taxonomyReferences`. Mehrfachzuordnung ist möglich. Vokabular und Taxonomien legst du im Neos-Backend unter *Taxonomie* an.

## Lizenz

MIT. Feed-Aufbau und strukturierte Daten orientieren sich an [Sebobo/Shel.Blog](https://github.com/Sebobo/Shel.Blog) (MIT).
