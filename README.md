# Waterproof.News

News and blog package for **Neos CMS 9**. Articles are document nodes, lists are
built with [Flowpack.Listable](https://github.com/Flowpack/Flowpack.Listable),
categories with [Sitegeist.Taxonomy](https://github.com/sitegeist/Sitegeist.Taxonomy).

The package ships **structure, not design**. Markup carries Tailwind class names,
there is no CSS file and no build step of its own. Styling happens in the build
of the site package that includes it.

## Installation

```bash
composer require waterproof/neos-news
./flow neos.flow:package:rescan
./flow flow:cache:flush --force
./flow resource:publish
```

The three commands after the require are mandatory under DDEV. The Composer
plugin writes the package cache to `Data/Temporary/`, while Flow reads it from
`/tmp/Flow/`. Without `package:rescan` the package stays invisible.

### Set up Tailwind, or everything stays unstyled

The Fusion files of this package live outside your site package. Tailwind only
finds class names in files listed in its `content` glob. Add this to your
`tailwind.config.js`:

```js
content: [
    './Resources/Private/**/*.{fusion,html,js}',
    './NodeTypes/**/*.{yaml,fusion}',
    '../../Packages/Application/Waterproof.News/Resources/Private/**/*.fusion',
],
```

Without that entry the package works, but arrives without any layout.

## Node types

| Node type | Backend label | Purpose |
|---|---|---|
| `Waterproof.News:Document.Article` | Artikel | Single post with date, teaser text, teaser image, author, content area |
| `Waterproof.News:Document.ArticleIndex` | Artikelübersicht | Parent page, lists its articles with pagination |
| `Waterproof.News:Content.ArticleTeaser` | Artikel-Teaser | Shows the latest articles on any page |
| `Waterproof.News:Document.Feed` | Feed | Atom or RSS output of the articles |

The article index only accepts articles below itself. That does not stop an
article from being created elsewhere. If you want to rule that out, narrow the
constraints of your own base page in your site package.

## Column choice

`Document.ArticleIndex` and `Content.ArticleTeaser` carry a `columns` property:

| Value | Label | Result |
|---|---|---|
| `cols1` | Einspaltig | one column at every width |
| `cols2` | Zweispaltig | two columns from `md` |
| `cols3` | Dreispaltig | two from `md`, three from `lg` (default) |
| `cols4` | Vierspaltig | two from `sm`, four from `lg` |

The class names are written out in `Component/GridClass.fusion`. Composed class
names would be invisible to the Tailwind scanner.

## Apply your own design

Override the prototypes you want to style in your site package:

```fusion
prototype(Waterproof.News:Document.Article.Short) < prototype(Neos.Fusion:Component) {
    renderer = afx`…your card layout…`
}
```

| Prototype | Responsible for |
|---|---|
| `Waterproof.News:Document.Article.Short` | Card in listings |
| `Waterproof.News:Content.ArticleIndexBody` | List body of the index |
| `Waterproof.News:Content.ArticleBody` | Detail page |
| `Waterproof.News:Component.ArticleCollection` | Grid around the cards |
| `Waterproof.News:Component.GridClass` | Mapping of column value to classes |

## Filters and archive

The index reads two parameters:

| Parameter | Effect |
|---|---|
| `kategorie` | Node name of the taxonomy, for example `?kategorie=abwasserbeseitigung` |
| `jahr` | Four digits, for example `?jahr=2026` |

They combine and survive pagination. The filter bar only offers categories that
actually carry an article, and the years present in the archive.

## Feed

The feed is a document node below the index, with a choice between Atom and
RSS 2.0. The index page points to it with `<link rel="alternate">` in the head.

**Why not `/aktuelles.rss`:** Neos routes documents with exactly one site wide
URI suffix, `.html` by default. A different ending would have to be set in the
site configuration and would then apply to every page. The feed therefore lives
at `/aktuelles/feed.html`, served with the correct content type.

## Structured data

Article pages carry a JSON-LD object of type `Article` with title, date,
description, canonical URL and publisher. The author only appears when it is
filled in.

## Languages

Backend labels ship as XLIFF in `Resources/Private/Translations/` for German and
English. For another language, copy the catalogue and translate it. The node
types themselves need no change.

## Categories

Articles reference taxonomies through the `taxonomyReferences` property.
Multiple assignments are possible. Vocabulary and taxonomies are created in the
Neos backend under *Taxonomie*.

## License

MIT. Feed structure and structured data follow the approach of
[Sebobo/Shel.Blog](https://github.com/Sebobo/Shel.Blog) (MIT).

Built and maintained by [waterproof.agency](https://waterproof.agency/).
